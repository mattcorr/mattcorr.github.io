---
title:  "Issue with BTDF not deploying to GAC in multi server installs"
header:
categories: 
  - BizTalk
tags:
  - biztalk
  - btdf
  - 
---
We utilise the [BizTalk Deployment Framework (BTDF)](https://biztalkdeployment.codeplex.com) for deploying our BizTalk applications. It is a very useful utility and would strongly recommend checking it out if you are not currently utilising it for your BizTalk deployments.

However we did notice a small error with BTDF Version 5.5 in a given set of circumstances recently. This post will detail what this error is and how to work around it until a fix has been applied.

## What is the error?
In multi-server environments, for non primary servers where the **DeployBizTalkMgmtDB** flag is set to false, the GAC'ing of the application DLLs happens right at the very end. These are BizTalk artefact DLLs for schemas, maps, pipeline etc.

## So? What is wrong with that?
As part of your installation process if you need to create any web endpoints (receive locations) for your BizTalk application, they would be done in the **CustomPostDeployTarget** step in your Deployment.btdfproj file.
These websites will require the artefact DLLs to already be in the GAC. The CustomPostDeployTarget step is run _before_ the GAC'ing of the DLLs so it will fail.

This does not occur on BizTalk Servers where the DeployBizTalkMgmtDB flag is true which is the case for most test environments. This is because the all artifacts are added via BTSTask _before_ the CustomPostDeployTarget step and this includes putting them into the GAC.

If you are not doing anything in your CustomPostDeployTarget step, then you will not have this issue. _(I did say this was for given set of circumstances!)_

## Why does this happen?
We need to look at the BizTalkFramework.targets file (available via [the source code](https://biztalkdeployment.codeplex.com/SourceControl/latest)) to see:

```xml
<PropertyGroup>
    <DeployBizTalkMgmtDBfalseDependsOn>
      CustomPreInitialize;
      FrameworkInitialize;
      CustomPostInitialize;
      PreprocessBindings;
      PreprocessFiles;
      PreprocessAndConfigureLog4net;
      CustomDeployTarget;
      ConditionalHostStop;
      DeploySharedAssemblies;
      DeployExternalAssemblies;
      DeployComponents;
      DeployPipelineComponents;
      DeployCustomFunctoids;
      DeployVDirs;
      DeployBtsNtSvcExeConfig;
      CustomPostDeployTarget;
      CustomFinalDeploy
    </DeployBizTalkMgmtDBfalseDependsOn>
  </PropertyGroup>
 
  <!-- Used for a deployment that will NOT deploy assemblies to the BizTalk management database.
        Only one server in a BizTalk group should actually deploy assemblies to the management database. -->
  <Target Name="DeployBizTalkMgmtDB_false" DependsOnTargets="$(DeployBizTalkMgmtDBfalseDependsOn)">
    <!-- Support server deployments that do not include BizTalk management database. -->
    <!-- Need to handle these differently than standard targets - just putting in gac... -->
 
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(SchemasQualified)&quot;" Condition="'$(IncludeSchemas)' == 'true' and '%(Identity)' == '%(Identity)'" />
 
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(PipelinesQualified)&quot;" Condition="'$(IncludePipelines)' == 'true' and '%(Identity)' == '%(Identity)'" />
 
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(TransformsQualified)&quot;" Condition="'$(IncludeTransforms)' == 'true' and '%(Identity)' == '%(Identity)'" />
 
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(OrchestrationsQualified)&quot;" Condition="'$(IncludeOrchestrations)' == 'true' and '%(Identity)' == '%(Identity)'" />
  </Target>
```

As you can see the the **DeployBizTalkMgmtDB_false** target has a dependency on the other group of targets. When we look at the targets in that group, we can see the CustomPostDeployTarget there.

## Why is this an issue?
This is something is going to remain undetected until you get to your Prod like environment or Production itself if you have more than one BizTalk Server deployed. Suddenly applications that have not had any issues deploying previously in single server environments will be failing.

## Okay... so how do we fix it?
There is a [work item](https://biztalkdeployment.codeplex.com/workitem/11016) in place to fix this, but in the meantime, I would suggest we take the code in the **DeployBizTalkMgmtDB_false** target from BizTalkFramework.targets as shown above and add it to the start of the CustomPostDeploy step in your Deployment.btdproj file.

```xml
<Target Name="CustomPostDeployTarget">
    <!-- Workaround for BTDF not deploying Artefacts to GAC when DeployBizTalkMgmtDB is false  -->
    <Message Text="Pre-deploying BizTalk Artefact DLLs into the GAC before rest of CustomPostDeployTarget executes..." Condition="'$(DeployBizTalkMgmtDB)' == 'false'"/>
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(SchemasQualified)&quot;" Condition="'$(IncludeSchemas)' == 'true' and '$(DeployBizTalkMgmtDB)' == 'false' and '%(Identity)' == '%(Identity)'" />
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(PipelinesQualified)&quot;" Condition="'$(IncludePipelines)' == 'true' and '$(DeployBizTalkMgmtDB)' == 'false' and '%(Identity)' == '%(Identity)'" />
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(TransformsQualified)&quot;" Condition="'$(IncludeTransforms)' == 'true' and '$(DeployBizTalkMgmtDB)' == 'false' and '%(Identity)' == '%(Identity)'" />
    <Exec Command="&quot;$(Gacutil)&quot; /f /i &quot;@(OrchestrationsQualified)&quot;" Condition="'$(IncludeOrchestrations)' == 'true' and '$(DeployBizTalkMgmtDB)' == 'false' and '%(Identity)' == '%(Identity)'" />
    
    <!-- ================================================================================================================================== -->
    
    <Message Text="Running SQL scripts for $(ProjectName)..." Condition="'$(DeployBizTalkMgmtDB)' == 'true'"/>
    <!-- Only run the SQL QUery if the BizTalk BAM Datasource variable is set-->
    <Exec Condition="'$(BizTalkBAM_DataSource)' != '#{BizTalkBAM_DataSource}' and '$(DeployBizTalkMgmtDB)' == 'true'" Command="sqlcmd.exe -S $(BizTalkBAM_DataSource) -d BAMPrimaryImport -E -i &quot;..\SolutionItems\SQL\Create_ESB_BamViews.sql&quot;" />
    <!-- ================================================================================================================================== -->
```

The only change required is tweaking the condition parameter to only do this is the **DeployBizTalkMgmtDB** flag is false.

Yes this will result with GAC'ing the DLLs twice when you do a deploy and DeployBizTalkMgmtDB is false, but this is harmless and the deployment will not fail.

Hopefully the fix will be applied soon and we won't need to worry about this workaround in the future!
