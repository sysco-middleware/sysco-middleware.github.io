---
layout: post
title: Updating AMX TableLayouts dynamically with Kadene's 2D Algorithm sample
categories: Oracle MAF
tags: [MAF,AMX,kadene 2D]
author: cliops
---
Now we will learn how to change dynamically some MAF AMX components.
There we go:

## Create classes ##

To start with the table structure we are going to create one class for the entire table called TableSum and other for the cells called CellNumber.

```java 
public class TableSum {
    
    
    CellNumber[] TableRow1 = new CellNumber[5];
    CellNumber[] TableRow2 = new CellNumber[5];
    CellNumber[] TableRow3 = new CellNumber[5];
    CellNumber[] TableRow4 = new CellNumber[5];
    CellNumber[] TableRow5 = new CellNumber[5];
    CellNumber[] TableRow6 = new CellNumber[5];
    CellNumber[] TableRow7 = new CellNumber[5];
    CellNumber[] TableRow8 = new CellNumber[5];
    CellNumber[] TableRow9 = new CellNumber[5];
    CellNumber[] TableRow10 = new CellNumber[5];
	String resultSum="";
	
``` 

```java 
public class CellNumber {
    String number;
    String color;
``` 

Basically the table contains 10 rows and 5 columns. Every cell contains 2 fields a number and a color.  

### Create Data Controls ###

To map the table class over the interface we will create one data control called TableSum.  

- ![](/images/2015-10-19-update-dynamically-amx-tablelayout/dataControlTableSum.jpg)

### Create TableLayout ###

Now, we create a TableLayout and we should map every TableRow of Data Controls to each rowLayout.
 

```html 
    <amx:tableLayout id="tl1" borderWidth="1" width="100%">
      <amx:rowLayout id="rl1">
        <amx:iterator var="row" value="#{bindings.tableRow11.collectionModel}" id="i1">
          <amx:cellFormat id="cf1" halign="center">
            <amx:outputText value="#{row.number}" id="ot2" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl2">
        <amx:iterator var="row" value="#{bindings.tableRow21.collectionModel}" id="i2">
          <amx:cellFormat id="cf2" halign="center">
            <amx:outputText value="#{row.number}" id="ot3" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl3">
        <amx:iterator var="row" value="#{bindings.tableRow31.collectionModel}" id="i3">
          <amx:cellFormat id="cf3" halign="center">
            <amx:outputText value="#{row.number}" id="ot4" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl4">
        <amx:iterator var="row" value="#{bindings.tableRow41.collectionModel}" id="i4">
          <amx:cellFormat id="cf4" halign="center">
            <amx:outputText value="#{row.number}" id="ot5" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl5">
        <amx:iterator var="row" value="#{bindings.tableRow5.collectionModel}" id="i5">
          <amx:cellFormat id="cf5" halign="center">
            <amx:outputText value="#{row.number}" id="ot6" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl6">
        <amx:iterator var="row" value="#{bindings.tableRow6.collectionModel}" id="i6">
          <amx:cellFormat id="cf6" halign="center">
            <amx:outputText value="#{row.number}" id="ot7" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl7">
        <amx:iterator var="row" value="#{bindings.tableRow7.collectionModel}" id="i7">
          <amx:cellFormat id="cf7" halign="center">
            <amx:outputText value="#{row.number}" id="ot8" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl8">
        <amx:iterator var="row" value="#{bindings.tableRow8.collectionModel}" id="i8">
          <amx:cellFormat id="cf8" halign="center">
            <amx:outputText value="#{row.number}" id="ot9" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl9">
        <amx:iterator var="row" value="#{bindings.tableRow9.collectionModel}" id="i9">
          <amx:cellFormat id="cf9" halign="center">
            <amx:outputText value="#{row.number}" id="ot10" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
      <amx:rowLayout id="rl10">
        <amx:iterator var="row" value="#{bindings.tableRow10.collectionModel}" id="i10">
          <amx:cellFormat id="cf10" halign="center">
            <amx:outputText value="#{row.number}" id="ot11" inlineStyle="color:#{row.color};"/>
          </amx:cellFormat>
        </amx:iterator>
      </amx:rowLayout>
    </amx:tableLayout>
	
``` 

### Add action button ###

To update programatically, we will add a button from a method that belongs to TableSum class called runAlgorithm.
  

```java 
        public void runAlgorithm(){
        // Modify the array's elements to now hold the sum  
                // of all the numbers that are above that element in its column 
        int N=10,M=5;
        int max_ending_here, max_so_far, maximum = Integer.MIN_VALUE;
        int xBottom=0,yBottom=0,xTop=0,yTop=0;
        int xBottomAux=0,yBottomAux=0,xTopAux=0,yTopAux=0,yTopAuxOld=0,yTopAuxNew=0;

            for (int startx = 0; startx < N; startx++) {
                int [] sum= new int[N];
                for (int x = startx; x < N; x++) {
                    max_ending_here = 0;
                    max_so_far = Integer.MIN_VALUE;
                    yTopAuxOld=0;yTopAuxNew=0;
                    
                    for (int y = 0; y < M; y++) {
                        sum[y] += new Integer(FullTable[x][y].getNumber());
                        
                        max_ending_here = Math.max(0, max_ending_here + sum[y]);
                        if (0==max_ending_here){
                        
                            yTopAuxNew=y+1<M?y+1:yTopAuxOld;
                        }
                        if (max_so_far<max_ending_here){
                            xTopAux=startx;
                            xBottomAux=x;
                            yBottomAux=y;
                            yTopAux=yTopAuxOld;
                            yTopAuxOld=yTopAuxNew;
                        }
                        
                        max_so_far = Math.max(max_so_far, max_ending_here);
                    }
                    if (max_so_far>maximum){
                        xTop=xTopAux;
                        xBottom=xBottomAux;
                        yTop=yTopAux;
                        yBottom=yBottomAux;
                        
                    }    
                    maximum = Math.max(maximum, max_so_far);
                }
            }
        propertyChangeSupport.firePropertyChange("resultSum", "zz", maximum +" ("+xTop+","+yTop+") ("+xBottom+","+yBottom+")");     
        paintResult(xTop,yTop,xBottom,yBottom);
    }
              
``` 

Also, we are adding the result in the resultSum variable.

```html 
    <amx:tableLayout id="tl2" width="100%">
      <amx:rowLayout id="rl11">
        <amx:cellFormat id="cf11" halign="center">
          <amx:commandButton actionListener="#{bindings.runAlgorithm.execute}" text="run Algorithm"
                             disabled="#{!bindings.runAlgorithm.enabled}" id="cb1"/>
        </amx:cellFormat>
      </amx:rowLayout>
    </amx:tableLayout>
    <amx:outputText value="#{bindings.resultSum.inputValue}" id="ot12"/>
              
``` 

### Refresh components ###

To refresh amx components we can use ProviderChangeSupport and PropertyChangeSupport.     

	- PropertyChangeSupport for attributes like String, Boolean, Integer, etc.
	- ProviderChangeSupport for collections like arrays, collections, etc.

First we should add the following lines to the TableSum Class:

```java 
    protected transient ProviderChangeSupport providerChangeSupport = new ProviderChangeSupport(this);
    private transient PropertyChangeSupport propertyChangeSupport = new PropertyChangeSupport(this);
	
    public void addProviderChangeListener(ProviderChangeListener l) {
        providerChangeSupport.addProviderChangeListener(l);
    }

    public void removeProviderChangeListener(ProviderChangeListener l) {
        providerChangeSupport.removeProviderChangeListener(l);
    }
    public void addPropertyChangeListener(PropertyChangeListener l) {
        propertyChangeSupport.addPropertyChangeListener(l);
    }

    public void removePropertyChangeListener(PropertyChangeListener l) {
        propertyChangeSupport.removePropertyChangeListener(l);
    }
``` 

Then for every refreshing we can use the following methods.   

For every table rows:   

```java 
    providerChangeSupport.fireProviderRefresh("tableRow1");
```

For result message:   

```java 
    propertyChangeSupport.firePropertyChange("resultSum", "oldValue", maximum +" ("+xTop+","+yTop+") ("+xBottom+","+yBottom+")");     
```

### Testing ###

Finally, we test the sample application   

Before run algorithm:   

- ![](/images/2015-10-19-update-dynamically-amx-tablelayout/TestBefore.jpg)

After run algorithm:   
Basically, the algorithm let us to get the max sum of a 2D array in a complexity O(n^3).   
This time the result was 108 and the rectangle goes from Top-Left (1,1) to Bottom-right (9,3) colored with blue. 

- ![](/images/2015-10-19-update-dynamically-amx-tablelayout/TestAfter.jpg)

You can download the project [***Here***](/files/applications/RefreshingMAF.zip)












