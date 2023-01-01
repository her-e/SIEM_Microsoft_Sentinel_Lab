## Microsoft Sentinel(SIEM) World Map RDP Failure
## Description *
In this lab, I am going to configure NTFS permissions by adding security groups and partitioning drives in order to map drives using Group Policy. The purpose of this lab will ensure users can access folders based on their least privileged access. Configuring and running this lab will help develop our understanding how NTFS permissions, Security Group, and Group Policy works. I will also configure user account with identity and Access regarding location and job titles.

In this lab, I have already created a domain controller on a virtual machine that has two network adapters. This will connect to our outside internet by NAT and the other network adapter (NIC) will connect to our VirtualBox private network. This will allow our client's PC to connect to the internet through the domain controller. This will allow us to apply Group Policy Objects and provision, maintain, and deprovision users in Active Directory.

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

[Source](https://www.datto.com/blog/what-is-ntfs-and-how-does-it-work)

<br>



## Conclusion
