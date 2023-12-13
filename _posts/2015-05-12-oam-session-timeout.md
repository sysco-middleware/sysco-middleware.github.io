---
layout: post
title: OAM Session Timeout Configuration Woes
categories: Oracle Access Management OAM Access Manager
tags: [access, identity, oam, middleware]
author: tisaksen
keep: no
---
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
After a full day of work some people working late started losing their session and were unable to log back on.

This applies to OAM 11.1.2.2.0 and later and the basic configuration was as follows:

![](/images/2015-05-12-oam-session-timeout/oam_session_timeout_settings.jpg)

Nothing out of the ordinary as you can see.
The expected session lifetime is set to 12 hours (720 minutes) and idle timeout is set to 1 hour. 

After a lot of digging, checking timeout values on the webgate, http server, load balancer and whatnot, it turned out that there is an undocumented setting in *oam-config.xml* called **CredentialValidityInterval** which defaults to 480 minutes (8 hours). After 8 hours users will hit this setting and lose session and the ability to log back in for another 4 hours.

The solution is simple, just edit the value of **CredentialValidityInterval** and increase it from 480 M to 720 M . Please remember to update the Version element of *oam-config.xml* or your changes will be ignored.

*oam-config.xml*

	<Setting Name="SessionConfigurations" Type="htf:map">
		... 
		<!-- Change this from 480 M to 720 M --> 
		<Setting Name="CredentialValidityInterval" Type="htf:timeInterval">720 M</Setting>
		...
	</Setting>

This finding has been documented at Oracle Support:<br>

[Sessions Timing Out In OAM (Doc ID 1577300.1)](https://support.oracle.com/epmos/faces/DocumentDisplay?id=1577300.1)
 

<div class="fa fa-magic">&nbsp;</span> Did you know that you can also set session idle timeout at domain level?</div>

![](/images/2015-05-12-oam-session-timeout/domain_session_timeout_settings.jpg)
