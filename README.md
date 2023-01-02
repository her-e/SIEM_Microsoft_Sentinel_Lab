## Microsoft Azure Sentinel (SIEM) World Map RDP Failure
## Description *
In this lab, I am going to configure NTFS permissions by adding security groups and partitioning drives in order to map drives using Group Policy. The purpose of this lab will ensure users can access folders based on their least privileged access. Configuring and running this lab will help develop our understanding how NTFS permissions, Security Group, and Group Policy works. I will also configure user account with identity and Access regarding location and job titles.

In this lab, I have already created a domain controller on a virtual machine that has two network adapters. This will connect to our outside internet by NAT and the other network adapter (NIC) will connect to our VirtualBox private network. This will allow our client's PC to connect to the internet through the domain controller. This will allow us to apply Group Policy Objects and provision, maintain, and deprovision users in Active Directory.


<p align="center">
<img src=".png" height="85%" width="85%" alt="SIEM"/>
<br>

## Important:

We’re intentionally setting up a vulnerable ecosystem, some of these practices are simply unrealistic in a working real world solution.
The whole intention is getting ourselves a practice ground to get our hands dirty and gain some knowledge with AD’s numerous vulnerabilities and defensive methodologies.
  
<br>

## What is Security Information and Event Management (SIEM)?

A SIEM system is a tool that an organization uses to collect, analyze, and perform security operations on its computer systems. Those systems can be hardware appliances, applications, or both.

In its simplest form, a SIEM system enables you to:

Collect and query logs.
Do some form of correlation or anomaly detection.
Create alerts and incidents based on your findings.
A SIEM system might offer functionality such as:

- **Log management:** The ability to collect, store, and query the log data from resources within your environment.

- **Alerting:** A proactive look inside the log data for potential security incidents and anomalies.

- **Visualization:** Graphs and dashboards that provide visual insights into your log data.

- **Incident management:** The ability to create, update, assign, and investigate incidents that have been identified.

- **Querying data:** A rich query language, similar to that for log management, that you can use to query and understand your data.


## What is Microsoft Sentinel?

Microsoft Sentinel is a cloud-native SIEM system that a security operations team can use to:

Get security insights across the enterprise by collecting data from virtually any source.
Detect and investigate threats quickly using built-in machine learning and Microsoft threat intelligence.
Automate threat responses by using playbooks and by integrating Azure Logic Apps.
Unlike traditional SIEM solutions, to run Microsoft Sentinel, you do not need to install any servers either on-premises or in the cloud. Microsoft Sentinel is a service that you deploy in Azure. You can get up and running with Sentinel in just a few minutes in the Azure portal.

Microsoft Sentinel is tightly integrated with other cloud services. Not only can you quickly ingest logs, but you can also use other cloud services natively (for example, authorization and automation).

Microsoft Sentinel helps you enable end-to-end security operations. Including collection, detection, investigation, and response:

<p align="center">
<img src="https://imgur.com/azGVweJ.png" height="85%" width="85%" alt="SIEM"/>


[Source: Microsoft](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-sentinel/2-what-is-azure-sentinel)

<br>

## Create an Azure Virtual Machine
Make sure to create a Resource Group before creating Virtual Machine.
- **Resource Group:** HoneypotLab
- **Name:** Honeypot-vm
- **Region:** US East 2
- **Size:** Standard D2as v4 (2 vcpus, 8 GiB memory)


<p align="center"> 
<img src="https://imgur.com/9diVHlc.png" height="80%" width="80%" alt="SIEM"/>

Azure Security Feature: The creation of the virtual machine places our VM into a Virtual Network (vnet), which our Virtual machine is assigned an IP Address and Network Interface. We will use Network Security Group (NSG) to filter network traffic on Azure resources. Our Network Security Group will be based on dictating source and destination and network ports that are allowed or denied.

- Create Inbound Rule within **Network Security Group (NSG)** to observe Brute-force attacks into the VM (This will open all ports and allow all network traffic into the VM):

<p align="center"> 
<img src="https://imgur.com/9hz1vUN.png" height="50%" width="40%" alt="SIEM"/>

## Create Log Analytics Workspace

Create Log Analytics Workspace to ingest Windows Event Logs from Azure Virtual Machine. 
<br/>
**Steps: Create Logs Analytics Workspace > Assign Resource Groups, Name, Region.**
- In this case, we will assign resource group "HoneypotLab", Name "law-honeypot", Region "East US 2"

<p align="center"> 
<img src="https://imgur.com/cDLHIwh.png" height="80%" width="80%" alt="SIEM"/>

- Enable Logs in "Microsoft Defender for Cloud" to enable the ability to gather logs from Azure Virtual Machine into "Log Analytics Workspace".

**Steps: Microsoft Defender for Cloud > Environment Settings > (Select Logs Analytics Workspace) > Enable All, but disable SQL Servers on Machines.**

<p align="center"> 
<img src="https://imgur.com/aSeuUO9.png" height="80%" width="80%" alt="SIEM"/>
<img src="https://imgur.com/WHNMDXQ.png" height="80%" width="80%" alt="SIEM"/>

After this step, connect Log Analytics Workspace with the Virtual Machine.

## Configure Microsoft Sentinel 
Add Log Analytics Workspace into Microsoft Sentinel.
<p align="center"> 
<img src="https://imgur.com/56oQQ16.png" height="80%" width="80%" alt="SIEM"/>

- Run PowerShell Script to Geo data from attackers within Virtual Machine.
<p align="center"> 
<img src="https://imgur.com/ByHFtub.png" height="80%" width="80%" alt="SIEM"/>

- Create Custom Log in Log Analytics Workspace.
- Create Custom Fields/Extract Fields to create fields to input in World Map data.
- Set up Map in Microsoft Sentinel within Worknooks and assign values and data based on Custom Fields.

## Event Viewer
Check Security for Windows Log and filter current log for EventID 4625 to analyze log on failure attempts.

<p align="center"> 
<img src="https://imgur.com/dL7fRcT.png" height="80%" width="80%" alt="SIEM"/>

## World Map Failed RDP Brute-force Attacks After 1 hour.

<p align="center"> 
<img src="https://imgur.com/WIRaGzQ.png" height="80%" width="80%" alt="SIEM"/>

## World Map Failed RDP Brute-force Attacks After 10 hours.

<p align="center"> 
<img src="https://imgur.com/WIRaGzQ.png" height="80%" width="80%" alt="SIEM"/>

## Security Concerns, just-in-time (JIT) access
Although we openned all ports through the Network Security Group (NSG) for this Microsoft Cloud Detection Lab (SIEM) to observe Brute-force attacks, we can reduce the attack surface by applying the just-in-time (JIT) access and re-configuring the Network Security Group to only enable certain ports.

The JIT security feature will only provide access when neccesarry based on time-based restrictions, RBAC roles, and Least Priviledge Principles.
For example, if anyone wants access to the Virtual Machine, access will only be given based on Requests, IP Address, and Job roles.
  
<br/>
Here is how to enable just-in-time (JIT) access. 

Microsoft Defender for Cloud > Workload Protection > Just-in-time VM access > Not Configured > Select VM > Enable JIT.
<p align="center"> 
<img src="https://imgur.com/H75QpAw.png" height="80%" width="80%" alt="SIEM"/>

Navigate to "Virtual Machine" settings and select "Connect". Select "My IP" and "Request Access".
<p align="center"> 
<img src="https://imgur.com/fS62Wht.png" height="80%" width="80%" alt="SIEM"/>

Maximum of three hours is allowed access using RDP Port 3389. The allowed source IP addresses is set to "Any". This can be changed under the Just-in-time VM access within Microsoft Defender for Cloud.

## Conclusion
