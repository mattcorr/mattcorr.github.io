---
title:  "Storing BizTalk application version via BTDF"
header:
categories: 
  - BizTalk
tags:
  - btdf
  - powershell
---
## The issue
A few weeks ago I saw a _very_ interesting [Integration Monday](http://www.integrationusergroup.com) talk session by [Toon Vanhoutte](http://www.codit.eu/blog/authors/toon-vanhoutte/) which you can view [here](http://www.integrationusergroup.com/?event=biztalk-alm).

One thing that really stood out for me was how he utilised the [BizTalk Deployment Framework (BTDF)](https://biztalkdeployment.codeplex.com) BizTalk Application Description field.

We also utilise BTDF and find it a very capable tool for automating the deployment for single BizTalk applications and their related artefacts. 

## So why would we bother?
If you frequently compile your BizTalk applications _(i.e. via automated builds for each check-in)_ it can be tricky to keep track of which version is deployed in which environment.

So by setting the BizTalk Description field in the BTDF to the following:

```xml
<BizTalkAppDescription>Version: $(ProductVersion) - Deployed: $([System.DateTime]::Now.ToString("dd-MM-yyyy HH:mm:ss"))</BizTalkAppDescription>
```
allows us to easily query it in two ways. 

1) We can do it manually via the BizTalk Admin Console.

![](https://blogresourcestorage.blob.core.windows.net/images/2015/09/image001.png)

2) We can utilise BizTalk PowerShell to query it via a script.

## ow would we query it via PowerShell?
With BizTalk 2013 and later, you get a copy of BizTalk PowerShell in the folder:

`C:\Program Files (x86)\Microsoft BizTalk Server 2013 R2\SDK\Utilities\PowerShell`

or alternatively, you can get the latest from [codeplex](http://psbiztalk.codeplex.com/)

## Ok so I can see this folder. Now what?
First we need to load the module so we can utilise it.

Open up Windows ISE in Administrator mode and run:

```powershell
Import-Module  'C:\Program Files (x86)\Microsoft BizTalk Server 2013 R2\SDK\Utilities\PowerShell\BizTalkFactory.PowerShell.Extensions.dll'
``` 

By default this will create a PSDrive for you. If it doesn't or you need something specific then you can use the line below to create one for you.

```powershell
New-PSDrive -Name BizTalk -PSProvider BizTalk -Root BizTalk:\ -Instance . -Database BizTalkMgmtDb -Scope Global 
```

This BizTalk PSDrive is where you can navigate the BizTalk Admin Console like a directory. 

Run:

```powershell
cd Biztalk:\Applications
```

If you type `dir` (or the more accurate PowerShell command `Get-ChildItem`), you are likely to get a scrolling list of the properties of your applications. 
![](https://blogresourcestorage.blob.core.windows.net/images/2015/09/2015-09-29_21-58-33.png)

To get more useful information type:
 
```powershell
dir | Select-Object -Property name, description, status
```

This selects just the name, description and status properties for each Application and displays them on the screen.
![](https://blogresourcestorage.blob.core.windows.net/images/2015/09/2015-09-29_22-01-45_01-1.png)

This is more like it now, but we still have the Version in a string field. It would be great to isolate that so we can do proper comparisons.

This is where regex comes to the rescue!

Now I know that regex has a **very** steep learning curve.  If you want to learn more about it, I highly recommend checking out and playing around with [https://regex101.com](https://regex101.com).
See below for how I was able to determine the regex required for getting the version number out of the string.
![](https://blogresourcestorage.blob.core.windows.net/images/2015/09/image008.png)

So with this knowledge, we can now format the query with slightly more complex PowerShell to look like:

```powershell
dir | Select-Object Name, @{n='Version';e={([regex]::Match($_.description, '\d+.\d+.\d+.\d+')).Value }},status
```

![](https://blogresourcestorage.blob.core.windows.net/images/2015/09/2015-09-29_22-03-39.png)

## Okay. But where would I use this?
Say you use something like [Octopus Deploy](https://octopus.com) for your deployment automation. Sure, its UI does have all the versions listed for each application deployed in each environment on display. To confirm the versions that Octopus _says_ is deployed with the versions that actually _are_ deployed, you can do a compare between the version number stored above and what is returned via the Octopus REST API.

I find it ideal to use the Description field to store the Version number and other metadata as it is easily readable by both humans and scripts. :)

Hope this helps other developers or at least gives them some ideas!