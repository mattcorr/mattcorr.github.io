---
title:  "Remote PowerShell via Azure Automation with Azure AD"
header:
categories: 
  - Automation
tags:
  - azure
  - powershell
---

## So why use Azure Automation?
Azure Automation is a quick and easy way to automate repeatable tasks against resources in your Azure account using PowerShell Workflow. No additional servers are required to host Automation as it is already handled automatically by Azure. Also if you already are experienced with PowerShell, you can use Automation already!

The purpose of this article is to demonstrate how to run PowerShell on Remote Azure VMs via Azure Automation using Azure AD for authentication.
{: .notice--info}

PowerShell Workflow is a slightly cut down version of PowerShell that utilises runbooks as the PowerShell scripts that are executed to perform your tasks. 

When you create a new runbook you are given an empty script to populate:

<script src="https://gist.github.com/mattcorr/a0977e0cc2bdcb9275c7.js"></script>

The demo below will show some sample workflow runbook scripts.

### Where is automation used?
Azure automation is utilised heavily at [Mexia](http://www.mexia.com.au) (the company I work for) for the backups and fail-overs for our [Confluence](http://www.atlassian.com) site. Our primary site is hosted on an Azure VM in Sydney _(the Australia East Azure region)_ and our secondary site on an Azure VM hosted in Melbourne _(the Australia South East Azure region)_.

Every night we perform backups on the primary site and store them in Azure File Storage. Every week we toggle primary/secondary sites between Sydney and Melbourne. This allows us to test our fail-over processes frequently for peace of mind. All of these tasks are scheduled Azure Automation runbooks.

### Sounds great! So what's this about Azure AD?
Originally we were using SSL certificates for authentication. But after [some research](http://blogs.msdn.com/b/microsoft_press/archive/2015/03/06/free-ebook-microsoft-azure-essentials-azure-automation.aspx), I found that utilising Azure AD was the preferred way of authentication for Azure Automation.

After looking into it to see how it was done, I too am now a convert to this approach over SSL certificates. It is much easier to configure and less code is required to perform authentication. Originally I was going to blog about how to use Azure AD, but after discovering that Joe Levy has already covered this in great details via [his blog post here](http://azure.microsoft.com/blog/2014/08/27/azure-automation-authenticating-to-azure-using-azure-active-directory/), there no point  repeating what he has already covered.

Instead I will build on Joe's post and detail how _(once authenticated via Azure AD)_ how you can easily run remote PowerShell scripts.

## On with the demo!

### Creating an Azure VM and Automation account
First we need an Azure VM to practise with. These can be created via the Azure portal. Lets call this VM **azureautodemo**.

![New Azure VM](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-12_16-02-35.png)

If you have not already, create an Automation account. Do this by first selecting **Automation** and click on **Create**.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-22_23-21-23.png)

Give it a name and ensure you select the region closest to you.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-12_16-27-34.png)

Next, follow the steps from [Joe's blog post](http://azure.microsoft.com/blog/2014/08/27/azure-automation-authenticating-to-azure-using-azure-active-directory/) for setting up an Azure AD account.

For this demo I called the account to use *Azure Automation*. Seems unlikely anyone should get confused by that!

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-12_16-15-19.png)

### Automation Assets
I prefer to use assets as much as possible to abstract out configuration of easily updatable variables.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-12_22-44-05.png)

I have created three for the purpose of this demo. These are:

|  Asset Name    |    Description    |
|----------------|-------------------|
|**Automation Account** | credentials asset containing the login details for the automation account *(Azure Automation)* |
|**VM Account** | credentials asset containing the login details to login to the VM *(azureautodemo)* |
|**Subscription Name** | string asset containing the name of the subscription |

### Creating the runbooks
When creating a collection of runbooks, it is recommended to start from the bottom up. Create all the granular scripts first, publish them and build up the main script that calls them as required.

For this demo, I will be creating a folder on the azureautodemo VM remotely. 
Pretty simple, but it will show how this all works.

### Create-Folder runbook

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-12_23-09-42.png)

Create a new runbook called **Create-Folder** and use the script below for the content.

<script src="https://gist.github.com/mattcorr/b1b8a596924d41446f56.js"></script>

So what is happening here? Well let's see. There are three parameters:

* The credentials for logging into the azureautodemo VM
* The URI object for connecting to the azureautodemo VM *(more on this shortly)*
* The name of the folder to create on the azureautodemo VM

The InlineScript command is used call the Invoke-Command. This Invoke-Command is used to run the PowerShell to create the folder remotely.
We pass in the three workflow parameters to the Invoke-Command script
The folder is passed in again as a parameter to a scriptblock that is executed on the remote VM.

InlineScript is a way to call standard PowerShell commands that are not in PowerShell Workflow. `Invoke-Command` is one of those commands.
It is important to note the _$Using:_ syntax for passing in variables in the InlineScript block.
{: .notice--info}

We have an **$options** variable to holds a SessionOption property defined to skip the Cert CA Check. This is required for the Invoke-Command call to work. If it is not present the call will fail. 

The rest of the code should be pretty self-explanatory.

The final step is to publish the Create-Folder runbook to allow it to be called from the Azure-AD-Test runbook. This is done by clicking on **Publish** from the bottom tool bar.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-21_22-56-46.png)

### Azure-AD-Test runbook

Create a new runbook with the name **Azure-AD-Test** and use the following code:

<script src="https://gist.github.com/mattcorr/f496ec7e4bc0d7333ea6.js"></script>

So what is happening in this runbook?

* Get the values for the assets we created
* Use Add-Azure and Select-AzureSubscription to authenticate via Azure AD
* The URI for the remote connection to the azureautodemo VM is obtained. We pass in the name of the VM here. _(This could be an automation asset too if required)_
* Finally call the **Create-Folder** runbook and pass in the URI, the credentials for logging into the azureautodemo VM and the folder to create

This can be tested without publishing, by clicking on **Test**.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-21_22-56-48.png)

Before it starts *(yes it does take a while as it does a lot behind the scenes)*, lets look at the C:\ on the azureautodemo VM to see the list of folders.

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-21_22-56-47.png)

The runbook will eventually run. Unfortunately any runbook testing takes a long time to execute. I recall reading somewhere that a local application might be released soon that allows us to test runbook scripts locally first before uploading to Azure. This should help make testing more time effective!

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-21_23-37-12.png)

Eventually the folder is created!

![](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2015/04/2015-04-21_23-36-50.png)

## Conclusion
While just creating a folder is rather basic, it does highlight how you can easily create scripts with parameters that you can use to run PowerShell on remote VMs.

From the demonstration above, I hope I have shown how you can use Azure Automation with Azure AD for authentication in order to do whatever we want via PowerShell to do whatever we want on the VMs in our subscriptions.

I think this is a pretty simple process, but if you feel there is a better way, please let me know! 