---
layout: single
title:  "Confluence 5.8.4 crashing unexpectedly?"
header:
categories: 
  - Confluence
tags:
  - java
---

We utilise [Atlassian Confluence](https://www.atlassian.com/software/confluence) for our internal wiki. 

Overall we find it to be the wiki market leader, but we recently had some issues with some unexpected server crashes after installing Confluence 5.8.4.

After some investigation and feedback from Atlassian, it turned out to be [a bug](https://bugs.openjdk.java.net/browse/JDK-8068400?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel) with the built in JRE (build 1.8.0\_45-b14).

The workaround for now is to install a pre-release JRE (build 1.8.0\_60-b21) which you can get from [here](https://jdk8.java.net/download.html).

Take note, ensure that when you install the new JRE, that you set the destination folder to the `confluence\jre` folder of your Confluence instance. This is required to override the built in JRE location utilised by Confluence, therefore replacing the buggy 1.8.0\_45-b14 with the fixed 1.8.0\_60-b21.

**NOTE:** If you use the system JRE for your confluence instance, then obviously ignore this section!
{: .notice--warning}

![](https://mcblogfiles.blob.core.windows.net/images/2015/07/2015-07-01_21-59-13_01.png)
Click on **Change Destination Folder**

![](https://mcblogfiles.blob.core.windows.net/images/2015/07/2015-07-01_21-59-14.png)
Click on **Change** and set the Confluence\jre folder to your location then click on **Install**.

That is all that is required. Restart your confluence service and you should not the crashing issues again.

We will have to wait for a new version of Confluence that has the officially released from Oracle JRE that has the fix to get past this workaround. _(and no, the just released version 5.8.5 does not have it yet)_