---
layout: post
title: Differences between ADF Mobile and Oracle MAF in development perspective
categories: compression
tags: [MAF,ADF Mobile]
author: cliops
---
This time, we will see some important advantages and disadvantages beetween these two versions. 
There we go:

#Overview#
ADF mobile is a mobile hybrid application development framework that can build mobile applications for two different Operating Systems: ![](/images/2015-09-21-differences-maf-and-adf-mobile/ios.JPG) ![](/images/2015-09-21-differences-maf-and-adf-mobile/android.JPG)    

	- IOS 7.0+ 
	- Android 4.x+ 

Also, this framework makes use of another technologies like Java, JavaScript, HTML5 and CSS3.    
Now, there is a new version of ADF Mobile that is called Oracle MAF. These have different licensing model because ADF mobile requires ADF and weblogic license, but Oracle MAF is licensed as an independent product. So we don't need any specific backend server for Oracle MAF.    

#Differences#

**I. Oracle MAF can develop using more IDE tools like Eclipse besides JDeveloper.**    

Developers with little experience in JDeveloper can use Eclipse IDE for development

	- Oracle MAF 
		-- Jdeveloper 12.1.3      
		-- Eclipse OEPE       
	- ADF Mobile 
		-- Jdeveloper 11.2.4    
		

Creating application in Oracle MAF using Eclipse   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/createMAFApplicationEclipse.JPG) 
		
Creating application in Oracle MAF using JDeveloper

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/createMAFApplication.JPG) 

Creating application in ADF Mobile using Jdeveloper

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/createADFMobileApplication.JPG) 


**II. Oracle MAF has more AMX components to bring a rich look and feel.**  

In both cases we can create views without AMX components, because we can include several frameworks as AngularJs and Ionic with pure HTML code.
If we are using amx components, there are 80 in total for MAF.

MAF (General Controls)  

There is a new component called OutputHTML to include HTML content in AMX pages.  

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxGeneralControlsMAF.JPG) 

ADF Mobile (General Controls)   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxGeneralControlsADFMobile.JPG) 

MAF (Text and selection)  

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxTextAndSelectionMAF.JPG) 

ADF Mobile (Text and Selection)   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxTextAndSelectionADFMobile.JPG) 

MAF (Data Views)  

There is a film strip to display multiple items arranged into a horizontal or vertical strip.   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxDataViewsMAF.JPG) 

ADF Mobile (Data Views)   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxDataViewsADFMobile.JPG) 

MAF (Layout)  

There are Panel stretch layout and Deck as new components in this group.   
Also there is a fragment to keep an entire page inside of it and accepts attributes and methods.

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxLayoutMAF.JPG) 

ADF Mobile (Layout)   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxLayoutADFMobile.JPG) 

MAF (Operations)  

There is a new component called Client listener that gives you a declarative way to register a Javascript listener that will be executed when a specific event type is fired.

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxOperationsMAF.JPG) 

ADF Mobile (Operations)   

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxOperationsADFMobile.JPG) 


**III. Oracle MAF has new Data visualization objects.**  

Now we can include a Sunburst to display hierarchical data across two dimensions and a Timeline to display a series of events in chronological order. 

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/amxDVTMAF.JPG) 

**IV. MAF has more CSS features that makes the UI development easier.**  

We can use several skins:

	- amx  
		mobileAlta-1.0  
			mobileAlta-1.1   
				mobileAlta-1.2  
					mobileAlta-1.3  
		mobileFusionFx-1.0  
			mobileFusionFx-1.1  
			
By default a new MAF project uses the latest version of the mobileAlta skin Family. If you want to change the sking just you have to go to maf-config.xml.   


**V. MAF has additional plugins like barcode integration.**  

We can use cordova plugins to handle more funcitonalities in the devices.

- ![](/images/2015-09-21-differences-maf-and-adf-mobile/barCode.JPG) 

#Conclusion#

A developer who currently works in ADF Mobile can adapt easily to Oracle MAF.   
The migration from ADF mobile project to MAF project is easy, just need to open in Jdeveloper 12.1.3!   
In future, any mobile development using Oracle Technology will be made in MAF and the actual ADF Mobile development will need to be switched.  


  
