---
layout: post
title: Booting Oracle WebLogic
categories: WebLogic
tags: [weblogic, wlst, nodemanager, middleware]
author: catoaune
---

This post is related to our presentations at Oracle OpenWorld 2014 and UKOUG Tech 14. All code/configuration used during the presentations are available here.

* The OOW presentation as a PDF file - [CON3633\_Sysco\_Booting\_Weblogic](/files/presentations/CON3633_Sysco_Booting_Weblogic.pdf)
* The Tech 14 presentation as a PDF file - [Sysco\_Booting\_Weblogic](/files/presentations/Sysco_Booting_Weblogic.pdf)
* See also my [article](http://www.otechmag.com/2014/otech-magazine-winter-2014-2/) in OTech Magazine

To start and stop AdminServer and the managed servers, we use a Jython (Python) script. It is on purpose made simple, but on production systems there should be more error handling, and also more efficient connection handling (i.e., check if there is a connection to NodeManager and/or AdminServer and reuse the connection if there is one active). Instead of calling the script below once for each server that we want to start, this script and the startall.sh script could be changed to give a list of servers to start, not just one and one as it is now.

*wls.py*

```bash
import sys

def startAdmin():
    print 'Starting AdminServer' 
    nmConnect(userConfigFile=nmUserFile, userKeyFile=nmKeyFile, host=nmHost, port=nmPort, domainName=domain, domainDir=domainPath, nmType=nmType)
    nmStart('AdminServer') 
    nmDisconnect()
    return

def stopAdmin():
    print 'Stopping AdminServer' 
    connect(userConfigFile=wlsUserFile, userKeyFile=wlsKeyFile, url=adminUrl) 
    shutdown('AdminServer', force='true')
    disconnect()
    return

def startManaged(managed):
    print 'Starting ', managed 
    connect(userConfigFile=wlsUserFile, userKeyFile=wlsKeyFile, url=adminUrl) 
    start(managed)
    disconnect()
return

def stopManaged(managed):
    print 'Stopping ', managed 
    connect(userConfigFile=wlsUserFile, userKeyFile=wlsKeyFile, url=adminUrl) 
    shutdown(managed, force='true') 
    disconnect()
return

if ((len(sys.argv) < 2) | (len(sys.argv) > 3)): 
    print ' Wrong number of arguments'
elif (sys.argv[1] == 'startadmin'):
    startAdmin()
elif (sys.argv[1] == 'stopadmin'): 
    stopAdmin()
elif (sys.argv[1] == 'start'): 
    startManaged(sys.argv[2]) 
elif (sys.argv[1] == 'stop'):
    stopManaged(sys.argv[2])
```

Full path for wlst.sh (under wlserver/common/bin) and wls.py must be given, unless they are in $PATH

*startall.sh*

```bash
wlst.sh -loadProperties config.properties -skipWLSModuleScanning wls.py startadmin
wlst.sh -loadProperties config.properties -skipWLSModuleScanning wls.py start ms1
```

*stopall.sh*

```bash
wlst.sh -loadProperties config.properties -skipWLSModuleScanning wls.py stop ms1
wlst.sh -loadProperties config.properties -skipWLSModuleScanning wls.py stopadmin
```

adminUrl must point to the AdminServer (often a VIP address) while nmHost points to the local NodeManager.

*config.properties*

```bash
adminUrl=t3://wls12c.dev.sysco.no:7001 
nmHost=wls12c.dev.sysco.no
nmPort=5556 
nmUserFile=/u01/app/oracle/config/nmUserFile 
nmKeyFile=/u01/app/oracle/config/nmKeyFile 
nmType=plain 
wlsUserFile=/u01/app/oracle/config/wlsUserFile 
wlsKeyFile=/u01/app/oracle/config/wlsKeyFile 
domain=mydomain 
domainPath=/u01/app/oracle/user_projects/domains/mydomain
```

Before running this command (deprecated in 12c, but still works in 12.1.3), you must source setDomainEnv.sh for the domain you are using.

Encrypt username and password (11g)

```bash
java weblogic.Admin -username nodemanager -userconfigfile /u01/app/oracle/config/nmUserFile -userkeyfile /u01/app/oracle/config/nmKeyFile STOREUSERCONFIG
```

To run this command, you must start WLST in interactive mode. First you have to source setDomainEnv.sh, then start WLST

Encrypt username and password (12c)

```bash
java weblogic.WLST
wls:/offline> nmConnect( ‘nodemanager','welcome1','localhost',5556,'mydomain', '/u01/app/oracle/user_projects/domains/mydomain', 'plain')
```

Encrypt username and password for Node Manager

```bash
wls:/mydomain/serverConfig> storeUserConfig(
'/u01/app/oracle/config/nmUserFile','/u01/app/oracle/config/nmKeyFile', 'true')
```

Encrypt username and password for WebLogic admin user

```bash
wls:/mydomain/serverConfig> connect('weblogic', 'welcome1', 't3://wls12c.dev.sysco.no:7001')
wls:/mydomain/serverConfig> storeUserConfig(
'/u01/app/oracle/config/wlsUserFile','/u01/app/oracle/config/wlsKeyFile','false')
```

This is not based on the script from the 12c documentation [http://docs.oracle.com/middleware/1213/wls/NODEM/java_nodemgr.htm#BABJIDFD](http://docs.oracle.com/middleware/1213/wls/NODEM/java_nodemgr.htm#BABJIDFD). One important difference is that this script starts NodeManager with user oracle, while the script in the documentation starts NodeManager as root, which is not recommended.

*/etc/init.d/nodemanager*

```bash
#!/bin/sh
#
# nodemanager Oracle Weblogic NodeManager service
#
# chkconfig:   345 85 15
# description: Oracle Weblogic NodeManager service

### BEGIN INIT INFO
# Provides: nodemanager
# Required-Start: $network $local_fs
# Required-Stop:
# Should-Start:
# Should-Stop:
# Default-Start: 3 4 5
# Default-Stop: 0 1 2 6
# Short-Description: Oracle Weblogic NodeManager service.
# Description: Starts and stops Oracle Weblogic NodeManager.
### END INIT INFO

. /etc/rc.d/init.d/functions

# Your WLS home directory (where wlserver is)
export MW_HOME="/u01/app/oracle/middleware/Oracle_Home"
export JAVA_HOME="/usr/java/latest"
DAEMON_USER="oracle"
PROCESS_STRING="^.*weblogic.NodeManager.*"

source $MW_HOME/wlserver/server/bin/setWLSEnv.sh &gt; /dev/null
export NodeManagerHome="$WL_HOME/common/nodemanager"
NodeManagerLockFile="$NodeManagerHome/nodemanager.log.lck"

PROGRAM="/u01/app/oracle/user_projects/domains/mydomain/bin/startNodeManager.sh"
SERVICE_NAME=`/bin/basename $0`
LOCKFILE="/var/lock/subsys/$SERVICE_NAME"

RETVAL=0

start() {
        OLDPID=`/usr/bin/pgrep -f $PROCESS_STRING`
        if [ ! -z "$OLDPID" ]; then
            echo "$SERVICE_NAME is already running (pid $OLDPID) !"
            exit
        fi

        echo -n $"Starting $SERVICE_NAME: "
        /bin/su $DAEMON_USER -c "$PROGRAM &amp;"

        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] &amp;&amp; touch $LOCKFILE
}

stop() {
        echo -n $"Stopping $SERVICE_NAME: "
        OLDPID=`/usr/bin/pgrep -f $PROCESS_STRING`
        if [ "$OLDPID" != "" ]; then
            /bin/kill -TERM $OLDPID
        else
            /bin/echo "$SERVICE_NAME is stopped"
        fi
        echo
        /bin/rm -f $NodeManagerLockFile
        [ $RETVAL -eq 0 ] &amp;&amp; rm -f $LOCKFILE

}

restart() {
        stop
        sleep 10
        start
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart|force-reload|reload)
        restart
        ;;
  *)
        echo $"Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $RETVAL
```

The weblogic scripts call the startall.sh when the server starts, and stopall.sh when the server stops.

*/etc/init.d/weblogic*

```bash
#!/bin/sh
#
# weblogic Oracle Weblogic start
#
# chkconfig: 345 85 15
# description: Oracle Weblogic service

### BEGIN INIT INFO
# Provides: 
# Required-Start: $nodemanager
# Required-Stop:
# Should-Start:
# Should-Stop:
# Default-Start: 3 4 5
# Default-Stop: 0 1 2 6
# Short-Description: Oracle Weblogic service.
# Description: Starts and stops Oracle Weblogic.
### END INIT INFO

 . /etc/rc.d/init.d/functions

# Your WLS home directory (where wlserver is)
export MW_HOME="/u01/app/oracle/middleware/Oracle_Home"
export BOOT_HOME="/u01/app/oracle/bootscripts"
export JAVA_HOME="/usr/java/latest"
DAEMON_USER="oracle"

source $MW_HOME/wlserver/server/bin/setWLSEnv.sh &gt; /dev/null
PROGRAM_START="$BOOT_HOME/startall.sh"
PROGRAM_STOP="$BOOT_HOME/stopall.sh"
SERVICE_NAME=`/bin/basename $0`
LOCKFILE="/var/lock/subsys/$SERVICE_NAME"
RETVAL=0

start() {
 echo -n $"Starting $SERVICE_NAME: "
 /bin/su $DAEMON_USER -c "$PROGRAM_START &amp;" RETVAL=$?
 echo [ $RETVAL -eq 0 ] &amp;&amp; touch $LOCKFILE
}

 stop() {
 echo -n $"Stopping $SERVICE_NAME: "
 /bin/su $DAEMON_USER -c "$PROGRAM_STOP &amp;" RETVAL=$?
 [ $RETVAL -eq 0 ] &amp;&amp; rm -f $LOCKFILE
 }

 restart() {
 stop
 sleep 10
 start
}

case "$1" in
 start)
        start
        ;;
 stop)
        stop
        ;;
 restart|force-reload|reload)
        restart
        ;;
 *)
        echo $"Usage: $0 {start|stop|restart}"
esac

exit 1
```

For both the nodemanager and the weblogic scripts, they must be made runable (chmod 0755) and activated (chkconfig --add) before they will be used next time the server starts or stops.

