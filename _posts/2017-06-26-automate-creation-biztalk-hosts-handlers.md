---
title:  "Automating BizTalk Hosts, Host Instances and Handlers"
header:
 image: /assets/images/2017/06/cathedralcove-header.jpg
 caption: "Cathedral Cove, New Zealand"
 teaser: /assets/images/2017/06/cathedralcove-tn.jpg
excerpt: "Details how you can create your BizTalk Hosts, Host Instances and Adapter Handlers in seconds rather than hours!"
categories: 
  - BizTalk
  - PowerShell
  - Automation
---

# Summary
When setting up a new BizTalk environment, often there is a list of client specific Hosts to be created.

By default, BizTalk one Host defined. 

For each Host, a host Instance needs to be created for each server in the BizTalk group.
Also additional Handlers for each of the BizTalk Adapters can be created as well.

This can all be performed via the BizTalk Administration console, but this will take a heck of a lot of clicking and a lot of time.

This triggered my Automation passion, so I have come up with a script that handles all this for you.

# How does it work?

This uses an xml configuration file that contains the host, instances and handlers to create.

## XML Configuration
See below for a sample:

NOTE: You can use from the xml below or from the github location [here](https://github.com/mattcorr/powershell-scripts/tree/master/BizTalk/Hosts).

```xml
<?xml version="1.0" encoding="utf-8"?>
<AdapterMappingConfiguration>
    <HostNames>
        <Host name="ReceieveHost" group="BizTalk Application Users" isolated="false" tracking="false" Is32Bit="false" trusted="false">
            <Instance server="SERVERNAMEHERE" account="\svc_bts_receive" password="PASSWORDHERE" />
        </Host>
        <Host name="SendHost" group="BizTalk Application Users" isolated="false" tracking="false" Is32Bit="false" trusted="false">
            <Instance server="PASSWORDHERE" account="\svc_bts_send" password="PASSWORDHERE" />
        </Host>
        <Host name="TrackingHost" group="BizTalk Application Users" isolated="false" tracking="true" Is32Bit="false" trusted="false">
            <Instance server="PASSWORDHERE" account="\svc_bts_track" password="PASSWORDHERE" />
        </Host>
    </HostNames>
    <Adapters>
        <Adapter name="FILE" send="true" receive="true" isoReceive="false" />
        <Adapter name="FTP" send="true" receive="true" isoReceive="false" />
        <Adapter name="HTTP" send="true" receive="false" isoReceive="true" />
        <Adapter name="MQSeries" send="true" receive="true" isoReceive="false" />
        <Adapter name="MSMQ" send="true" receive="true" isoReceive="false" />
        <Adapter name="POP3" send="false" receive="true" isoReceive="false" />
        <Adapter name="SMTP" send="true" receive="false" isoReceive="false" />
        <Adapter name="SOAP" send="true" receive="false" isoReceive="true" />
        <Adapter name="SQL" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-BasicHttp" send="true" receive="false" isoReceive="true" />
        <Adapter name="WCF-Custom" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-CustomIsolated" send="false" receive="false" isoReceive="true" />
        <Adapter name="WCF-NetMsmq" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-NetNamedPipe" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-NetTcp" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-SQL" send="true" receive="true" isoReceive="false" />
        <Adapter name="WCF-WSHttp" send="true" receive="false" isoReceive="true" />
    </Adapters>
</AdapterMappingConfiguration>
```

The following nodes will need customisation for your specific environment

### Host Node
* **name** = the name of the host
* **group** = the local or domain group for this host
* **islocated/tracking/Is32Bit/trusted** = boolean flags you can set for each host

### Instance Node
* **server** = the server name to install a host instance 
* **account** = the account the host instance runs as
* **password** = the password for the account

### Adapters
The list of BizTalk Adapters to create handlers for based on the defined Hosts
There are boolean toggles for if the send and receive are to be created. It is not expected these should change.

Users would only just include the adapters they have on their environment and add new rows for customer or additional adapters installed.

## PowerShell Scripts

The scripts required are available from my public PowerShell repo on github [here](https://github.com/mattcorr/powershell-scripts/tree/master/BizTalk/Hosts).

There are 3 files.

* **Add-HostInstanceHandlers.ps1** - the main script for adding the components required
* **BizTalkScripts.psm1** - the module used to provide the low level functions
* **Remove-HostInstanceHandlers.ps1** - undoes the work of the Add-HostInstanceHandlers script if desired.

Refer to the help in each of the files for details on what they do.






