---
title:  "Use PowerShell to enable or disable tracking for selected BizTalk Applications"
header:
categories: 
  - PowerShell
tags:
  - powershell
---

For a current client project we are using the excellent [BizTalk Deployment Framework](https://biztalkdeployment.codeplex.com/) for deploying our BizTalk applications. This works great, except it can be difficult to get the tracking flags set correctly for testing and production environments. 

Ideally you would want to have tracking disabled by default in Production environments and enabled for Test/Local Dev environments to aid with debugging message content.

I was sent a link to [an article on Sandro's blog](http://sandroaspbiztalkblog.wordpress.com/2014/11/03/biztalk-devops-how-to-take-control-of-your-environment-disable-tracking-settings-in-biztalk-server-environment/) that shows a PowerShell script for disabling tracking for all BizTalk applications. 

This was very useful, but I also needed the ability to turn tracking on and potentially for a subset of BizTalk Applications.

I have tweaked the PowerShell script and ended up with what you see below.
Feel free to use this in your environment and let me know if there are improvements I can make.

<script src="https://gist.github.com/mattcorr/7d5ec29cf8ab420cb95b.js"></script>

DISCLAIMER: Use this script at your own risk. While it is highly unlikely this will break anything, if it does I can not be held responsible.
{: .notice--danger}


Enjoy!