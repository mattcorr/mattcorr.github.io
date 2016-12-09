---
title:  "Check BizTalk application references in BTDF via PowerShell"
header:
categories: 
  - Automation
tags:
  - biztalk
  - btdf
  - powershell
---
At most client sites we utilise the [BizTalk Deployment Framework](https://biztalkdeployment.codeplex.com) for packaging up and deploying our BizTalk applications and their associated artefacts (BAM, BRE, custom SQL, WCF Services etc).

We also frequently have dependencies between BizTalk applications where a common Schema application *(Schemas only)* is referenced by one or more logic based applications *(usually Maps or Orchestrations)*.

This can cause issues if the logic applications are already deployed when an updated Schema application is ready to be deployed.

One way around this, is inside the Schemas application **Deployment.btdfproj** file to define a new **ItemGroup** as shown

```xml
 <ItemGroup>
    <AppsToRemove Include="MSCESB.Planning" />
    <AppsToRemove Include="MSCESB.Infrastructure" />
    <AppsToRemove Include="MSCESB.Quoting" />
    <AppsToRemove Include="MSCESB.PostalDeployment" />
  </ItemGroup>
```

and have the following code defined:

```xml
<Import Project="$(DeploymentFrameworkTargetsPath)BizTalkDeploymentFramework.targets" />
  <!--
    The Deployment Framework automatically packages most files into the server install MSI.
    However, if there are special folders that you need to include in the MSI, you can
    copy them to the folder $(RedistDir) in the CustomRedist target.
    To include individual files, add an ItemGroup with AdditionalFiles elements.
  -->
  <Target Name="CustomPostInitialize" Outputs="%(AppsToRemove.Identity)" Condition="'@(AppsToRemove)' != ''" >
    <!-- add the control / execute / terminate in here-->
    <Message Text="Removing Application - %(AppsToRemove.Identity)" Importance="High" />
    <GetBizTalkAppExists ApplicationName="%(AppsToRemove.Identity)">
      <Output TaskParameter="AppExists" PropertyName="AppExists1" />
    </GetBizTalkAppExists>
    <ControlBizTalkApp ApplicationName="%(AppsToRemove.Identity)" StopOption="$(ControlBizTalkAppStopOption)" Condition="'$(AppExists1)' == 'true'" ContinueOnError="true"/>
    <TerminateServiceInstances Application="%(AppsToRemove.Identity)" />
    <Exec Command="BTSTask.exe RemoveApp -ApplicationName:&quot;%(AppsToRemove.Identity)&quot;" Condition="'$(AppExists1)' == 'true'" ContinueOnError="true"/>
  </Target>
```

In the logic applications **Deployment.btdfproj** files we should utilise the standard ItemGroup with **AppsToReference**:

```xml  
<ItemGroup>
    <AppsToReference Include="MSCESB.ReferenceSystems.Contoso"/>
    <AppsToReference Include="MSCESB.BusinessSchemas"/>
</ItemGroup>
```

**NOTE:** MSCESB is a fictional "Matt Sample Company ESB" or even "Mexia Sample Company ESB" ;)
{: .notice--info}

This is great once it's all defined correctly. So when updated Schema applications are deployed, they will correctly un-deploy any defined BizTalk applications.

But when you have many applications controlled by developers across projects, it is likely some link definitions will be missed. This would be where the AppsToRemove would not have corresponding AppsToReference defined and vis versa.

## Yes. PowerShell. Again!
I have created a PowerShell script that will parse all Deployment.btdfproj files for multiple solutions and ensure where applications are referenced that they have matching corresponding references in related Deployment.btdfproj files.

<script src="https://gist.github.com/mattcorr/5d77365ff21835824a61.js"></script>

**NOTE:** This script might require modifications to paths and solution structures. It also assumes that the solution name has the same name as the BizTalk application it deploys.
{: .notice--success}

## Script Usage and Output
Calling it is simple. A single parameter that defines your root source code folder

```powershell
PS> .\Check-DependencyConfig.ps1 -RootLocalTFSPath C:\work\MSCESB
```

Sample output:

```
Checking MSCESB.Planning: C:\work\MSCESB\MSCESB.Planning\main\Deployment\Deployment.btdfproj
Apps to Reference:
MSCESB.ReferenceSystems.Contoso [Linked config file has matching AppsToRemove defined correctly.]
MSCESB.BusinessSchemas [Linked config file has matching AppsToRemove defined correctly.]
MSCESB.ReferenceSystems.Recode: [Linked config file NOT configured correctly.]

Checking MSCESB.ReferenceSystems.Recode: C:\work\MSCESB\MSCESB.ReferenceSystems.Recode\main\Deployment\Deployment.btdfproj
Apps to Remove:
MSCESB.Infrastructure [Linked config file NOT configured correctly.]
MSCESB.Planing [ERROR: Unable to find the Deployment.btdfproj file for 'MSCESB.Planing'. Is it spelt correctly?]

Checking MSCESB.ReferenceSystems.Planes: C:\work\MSCESB\MSCESB.ReferenceSystems.Planes\main\Deployment\Deployment.btdfproj
No Applications to Remove defined but app name sounds like a Schemas app? Should confirm this.
```

## So what is next?
So how do we get around the issue of all the un-deployed applications we have due to the updated schemas application being deployed?

There's two scenarios in this case:
1. If a schema has changed, then Logic based applications will also require updates to cater for that schema change *(i.e. map and/or orchestration changes)*.
2. There was a trivial schema change and no modifications to logic apps are required.

In the case of 1. then its a good idea that they were un-deployed. The applications can then be updated, checked in and deployed when refactored and tested.
In the case of 2. we would need an approach for determining what applications to redeploy and in what order. 

That's a topic for another blog post!
 