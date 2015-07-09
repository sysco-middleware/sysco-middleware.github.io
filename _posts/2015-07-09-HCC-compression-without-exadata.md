---
layout: post
title: HCC compression with DIRECT INSERT and without it
categories: compression
tags: [oracle, exadata, hcc, compression]
author: oleg
---

I was invited to test a compression on ZFS storage when datafiles were placed on dnfs.

And now I can prove, that the percent of compression really depends on the type of inserts.

In case of direct inserts we have 10-times compression:


``` sql
CREATE TABLESPACE tbsp1 DATAFILE '/zfs/ss7120/orashare/tbsp1.dbf' 
	SIZE 1M AUTOEXTEND ON MAXSIZE 10G;
CREATE TABLESPACE tbsp2 DATAFILE '/zfs/ss7120/orashare/tbsp2.dbf' 
	SIZE 1M AUTOEXTEND ON MAXSIZE 10G;
CREATE TABLESPACE tbsp3 DATAFILE '/zfs/ss7120/orashare/tbsp3.dbf' 
	SIZE 1M AUTOEXTEND ON MAXSIZE 10G;
drop table big_size_table;
drop table medium_size_tab;
drop table small_size_tab;
```

``` sql
CREATE TABLE big_size_table 
nocompress 
tablespace tbsp1
as  select * from scott.emp;
```
``` sql
DECLARE
BEGIN
  FOR i IN 1 .. 100000 LOOP
    insert into big_size_table select * from scott.emp;
    COMMIT;
  END LOOP;
END;
/
```

``` sql
CREATE TABLE medium_size_tab 
compress for query high
tablespace tbsp2
as select * from big_size_table;
```

``` sql
CREATE TABLE small_size_tab 
compress for archive high
tablespace tbsp3
as select * from big_size_table;
```

check the sizes of tables:

``` sql
COLUMN segment_name FORMAT A30
SELECT segment_name, bytes, 
bytes/
(SELECT bytes
FROM   user_segments
WHERE  segment_name ='BIG_SIZE_TABLE'
) as ratio
FROM   user_segments  
WHERE  segment_name IN ('BIG_SIZE_TABLE', 'MEDIUM_SIZE_TAB', 'SMALL_SIZE_TAB');
```

check the level of compression:

``` sql
select table_name, compression, compress_for 
from dba_tables
where table_name in ('BIG_SIZE_TABLE', 'MEDIUM_SIZE_TAB', 'SMALL_SIZE_TAB');
```

```
 SEGMENT_NAME                        BYTES      RATIO
------------------------------   ----------      ----------
BIG_SIZE_TABLE                   75497472          1
MEDIUM_SIZE_TAB                   786432          .01042
SMALL_SIZE_TAB                     393216          .00521
```

```
TABLE_NAME                      COMPRESS        COMPRESS_FOR
---------------------------     --------        ------------
BIG_SIZE_TABLE                  DISABLED
MEDIUM_SIZE_TAB                 ENABLED         QUERY HIGH
SMALL_SIZE_TAB                  ENABLED         ARCHIVE HIGH
```

And in case of usual insert we have only 10 percent of compression, so it's not usual HCC:

``` sql
drop table big_size_tab;
drop table medium_size_tab;
drop table small_size_tab;
```

``` sql
CREATE TABLE big_size_tab 
nocompress 
tablespace tbsp1
as  select EMPNO, ENAME, JOB from scott.emp;
```

``` sql
CREATE TABLE medium_size_tab 
nocompress 
tablespace tbsp2
as select EMPNO, ENAME, JOB from scott.emp;
```

``` sql
CREATE TABLE small_size_tab 
nocompress 
tablespace tbsp3
as select EMPNO, ENAME, JOB from scott.emp;
```

``` sql
DECLARE
BEGIN
  FOR i IN 1 .. 100000 LOOP
    insert into big_size_tab select EMPNO, ENAME, JOB from scott.emp;
    COMMIT;
  END LOOP;
  FOR i IN 1 .. 100000 LOOP
    insert into medium_size_tab select EMPNO, ENAME, JOB from scott.emp;
    COMMIT;
  END LOOP;
  FOR i IN 1 .. 100000 LOOP
    insert into small_size_tab select EMPNO, ENAME, JOB from scott.emp;
    COMMIT;
  END LOOP;
END;
/
```

Check the sizes of tables and a level of compression:

``` sql
COLUMN segment_name FORMAT A30
SELECT segment_name, bytes
FROM   user_segments
WHERE  segment_name IN ('BIG_SIZE_TAB', 'MEDIUM_SIZE_TAB', 'SMALL_SIZE_TAB');
```

``` sql
select table_name, compression, compress_for 
from dba_tables
where table_name in ('BIG_SIZE_TAB', 'MEDIUM_SIZE_TAB', 'SMALL_SIZE_TAB');
```

```
 SEGMENT_NAME                   BYTES          RATIO
------------------------------ ----------    ----------
BIG_SIZE_TAB                   75497472         1
MEDIUM_SIZE_TAB                65664327        .8697
SMALL_SIZE_TAB                 65664327        .8697
```

```
TABLE_NAME                     COMPRESS        COMPRESS_FOR
----------------------------   --------        ------------
BIG_SIZE_TAB                   DISABLED
MEDIUM_SIZE_TAB                ENABLED         QUERY HIGH
SMALL_SIZE_TAB                 ENABLED         ARCHIVE HIGH
```

Of course, the level of HCC compression depends on the quantity of repeatable rows in each HCC block.

As you can see, I have populated the tables with strings from scott.emp.

On real data you might get less percentage. Not 10 times.