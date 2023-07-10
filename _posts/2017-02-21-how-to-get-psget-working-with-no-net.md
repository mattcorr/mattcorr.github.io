---
title:  "How to get PowerShellGet working with no Internet access"
header:
  image: "https://mcblogfiles.blob.core.windows.net/images/2017/02/powershellnuget-header.jpg"
  caption: "Castlepoint, New Zealand"
excerpt: "Assistance for setting up local PowerShellGet with no external internet access."
categories: 
  - PowerShell
tags:
  - workaround
  - powershell
---
As most of you know I am a big fan of [PowerShell](https://msdn.microsoft.com/en-us/powershell/mt173057.aspx). With Powershell 5.0, we now have PowerShellGet which means it is simple to install modules from the Internet via the `Install-Module` command.

```powershell
Install-Module gac
```

This works great if your computer is directly connected to the Internet and can see the global [PowerShell repository](http://www.powershellgallery.com/).

But what about if your VM is hidden behind a corporate firewall and has very limited or no internet access?
This makes it impossible to get the packages as shown below.

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_21-58-15.png)
First we might want to install a package. Oh... we need to install NuGet as well? No problem. Click Yes!

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_21-58-16.png)
But since we have no net access, NuGet can't be installed, so we get a sea of red text.

## What can we do?

One option is to create your own NuGet server behind the firewall and download and push packages to that.

There are already some [good articles](http://www.hanselman.com/blog/HowToHostYourOwnNuGetServerAndPackageFeed.aspx) about [how to do that](https://blogs.msdn.microsoft.com/powershell/2014/05/20/setting-up-an-internal-powershellget-repository/).

Once that is set up and you have published some packages to it, you need to register the repository so when you run `Install-Module`, PowerShell knows to look internally rather than externally.

A sample command for registering a PowerShell Repository is:

```powershell
Register-PSRepository -Name "LocalPowerShell" -SourceLocation "http://<servername>/nugetpowershell/nuget" -InstallationPolicy Trusted -PackageManagementProvider 'nuget'
```

But when you run it, you are again prompted to install NuGet, which will still fail due to no Internet access.

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_21-58-16b.png)

## So how can we get NuGet installed?

This seems to be the only way that works.(that I have seen!) 
We can see from the dialog message that it is expecting to be located in one of two folders.

One of them is **C:\Program Files\PackageManagement\ProviderAssemblies**.

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_21-58-16c.png)

So looking on another computer _(that **does** have Internet access)_ ensure that the NuGet package is installed and explore that folder, we can see:

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_22-12-21.png)

So if we copy that DLL over to the VM without internet access, and store it in the same folder structure, this seems to work.

**IMPORTANT NOTE:** After you copy the DLL over, ensure you restart your PowerShell ISE program!
{: .notice--danger}

## Time for Take Two
With the DLL in place and ISE restarted, lets run the command again to register the PowerShell repository:

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_22-17-38.png)
It still seems to give an error! **This can be ignored!** 
If we run the command `Get-PSRepository` we can see it was correctly registered.

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_22-18-29.png)

Now we can install modules from the local repository with no issues. Awesome!

**TIP:** If you want to find out what modules are available on any repository, enter `Find-Module -Repository <name of local repository>`

![](https://mcblogfiles.blob.core.windows.net/images/2017/02/2017-02-21_22-19-12.png)

This was the only way I was able to get this to work on servers with no Internet access. If there is a better way, please let me know!
{: .notice--warning}