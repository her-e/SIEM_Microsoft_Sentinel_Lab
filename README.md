## Microsoft Azure Sentinel (SIEM) - World Map with live Cyberattacks
## Description
In this lab, I am going to create a Honeypot to lure attackers to my Azure Virtual Machine. This is an intentionally compromised system that allows attackers to exploit vulnerabilities for the sole purpose to study incidents. This lab will require the use of Microsoft Sentinel, Log Analytics Workspace, Azure Virtual Machines, Kusto Query Language, Powershell, Third-Party API, Network Security Group (NSG), Just-in-time (JIT) access, and more. The purpose of conducting this lab is to understand Microsoft Azure resources by creating a cloud detection environment with Azure Virtual Machines and Windows Event Viewer Logs to analyze Brute-force attacks. After creating the Azure environment, we will run a PowerShell script that will obtain latitude, longitude, country, username data from IP addresses associated with the failed log on attempts.

Lab Results:
<p align="center">
<img src="https://imgur.com/23vVkFZ.png" height="85%" width="85%" alt="SIEM"/>
<br>

## Important:

We’re intentionally setting up a vulnerable ecosystem, some of these practices are simply unrealistic in a working real world solution.
The whole intention is getting ourselves a practice ground to get our hands dirty and gain some knowledge with Microsoft Azure resources and defensive methodologies.
  
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

Create Inbound Rule within **Network Security Group (NSG)** to observe Brute-force attacks into the VM (This will open all ports and allow all network traffic into the VM):

<p align="center"> 
<img src="https://imgur.com/9hz1vUN.png" height="50%" width="40%" alt="SIEM"/>

Port exposed to the internet due to NSG inbound rule:
- RDP port 3389 
- SSH port 22
- MS SQL DB port 1433
- Oracle DB port 1521
- Mysql DB port 3306
- Postgres DB port 5432
- etc. (All Ports)

## Create Log Analytics Workspace

Create Log Analytics Workspace to ingest Windows Event Logs from Azure Virtual Machine. 
<br/>
**Steps: Create Logs Analytics Workspace > Assign Resource Groups, Name, Region.**
  
In this case, we will assign...
- **Resource Group:** HoneypotLab 
- **Name:** law-honeypot 
- **Region:** East US 2

<p align="center"> 
<img src="https://imgur.com/cDLHIwh.png" height="80%" width="80%" alt="SIEM"/>

Enable Logs in "Microsoft Defender for Cloud" to enable the ability to gather logs from Azure Virtual Machine into "Log Analytics Workspace".

**Steps: Microsoft Defender for Cloud > Environment Settings > (Select Logs Analytics Workspace) > Enable All, but disable SQL Servers on Machines.**

<p align="center"> 
<img src="https://imgur.com/aSeuUO9.png" height="80%" width="80%" alt="SIEM"/>
<img src="https://imgur.com/WHNMDXQ.png" height="80%" width="80%" alt="SIEM"/>

After this step, connect Log Analytics Workspace with the Virtual Machine.

## Configure Microsoft Sentinel 
Add Log Analytics Workspace into Microsoft Sentinel.
<p align="center"> 
<img src="https://imgur.com/56oQQ16.png" height="80%" width="80%" alt="SIEM"/>

- This PowerShell script will extract the IP addresses from our Windows logs and send them to a third-party API ([ipgeolocation.io](https://ipgeolocation.io/)) to determine their latitude, longitude, state, country, etc., and then send it back to our resources where we’ll use these details to create a custom log.

## PowerShell Script

```powershell
# Get API key from here: https://ipgeolocation.io/
$API_KEY      = "d4600b4efdef42b39828f5155041a457"
$LOGFILE_NAME = "failed_rdp.log"
$LOGFILE_PATH = "C:\ProgramData\$($LOGFILE_NAME)"

# This filter will be used to filter failed RDP events from Windows Event Viewer
$XMLFilter = @'
<QueryList> 
   <Query Id="0" Path="Security">
         <Select Path="Security">
              *[System[(EventID='4625')]]
          </Select>
    </Query>
</QueryList> 
'@

<#
    This function creates a bunch of sample log files that will be used to train the
    Extract feature in Log Analytics workspace. If you don't have enough log files to
    "train" it, it will fail to extract certain fields for some reason -_-.
    We can avoid including these fake records on our map by filtering out all logs with
    a destination host of "samplehost"
#>
Function write-Sample-Log() {
    "latitude:47.91542,longitude:-120.60306,destinationhost:samplehost,username:fakeuser,sourcehost:24.16.97.222,state:Washington,country:United States,label:United States - 24.16.97.222,timestamp:2021-10-26 03:28:29" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-22.90906,longitude:-47.06455,destinationhost:samplehost,username:lnwbaq,sourcehost:20.195.228.49,state:Sao Paulo,country:Brazil,label:Brazil - 20.195.228.49,timestamp:2021-10-26 05:46:20" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:52.37022,longitude:4.89517,destinationhost:samplehost,username:CSNYDER,sourcehost:89.248.165.74,state:North Holland,country:Netherlands,label:Netherlands - 89.248.165.74,timestamp:2021-10-26 06:12:56" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:40.71455,longitude:-74.00714,destinationhost:samplehost,username:ADMINISTRATOR,sourcehost:72.45.247.218,state:New York,country:United States,label:United States - 72.45.247.218,timestamp:2021-10-26 10:44:07" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:33.99762,longitude:-6.84737,destinationhost:samplehost,username:AZUREUSER,sourcehost:102.50.242.216,state:Rabat-Salé-Kénitra,country:Morocco,label:Morocco - 102.50.242.216,timestamp:2021-10-26 11:03:13" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-5.32558,longitude:100.28595,destinationhost:samplehost,username:Test,sourcehost:42.1.62.34,state:Penang,country:Malaysia,label:Malaysia - 42.1.62.34,timestamp:2021-10-26 11:04:45" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:41.05722,longitude:28.84926,destinationhost:samplehost,username:AZUREUSER,sourcehost:176.235.196.111,state:Istanbul,country:Turkey,label:Turkey - 176.235.196.111,timestamp:2021-10-26 11:50:47" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:55.87925,longitude:37.54691,destinationhost:samplehost,username:Test,sourcehost:87.251.67.98,state:null,country:Russia,label:Russia - 87.251.67.98,timestamp:2021-10-26 12:13:45" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:52.37018,longitude:4.87324,destinationhost:samplehost,username:AZUREUSER,sourcehost:20.86.161.127,state:North Holland,country:Netherlands,label:Netherlands - 20.86.161.127,timestamp:2021-10-26 12:33:46" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:17.49163,longitude:-88.18704,destinationhost:samplehost,username:Test,sourcehost:45.227.254.8,state:null,country:Belize,label:Belize - 45.227.254.8,timestamp:2021-10-26 13:13:25" | Out-File $LOGFILE_PATH -Append -Encoding utf8
    "latitude:-55.88802,longitude:37.65136,destinationhost:samplehost,username:Test,sourcehost:94.232.47.130,state:Central Federal District,country:Russia,label:Russia - 94.232.47.130,timestamp:2021-10-26 14:25:33" | Out-File $LOGFILE_PATH -Append -Encoding utf8
}

# This block of code will create the log file if it doesn't already exist
if ((Test-Path $LOGFILE_PATH) -eq $false) {
    New-Item -ItemType File -Path $LOGFILE_PATH
    write-Sample-Log
}

# Infinite Loop that keeps checking the Event Viewer logs.
while ($true)
{
    
    Start-Sleep -Seconds 1
    # This retrieves events from Windows EVent Viewer based on the filter
    $events = Get-WinEvent -FilterXml $XMLFilter -ErrorAction SilentlyContinue
    if ($Error) {
        #Write-Host "No Failed Logons found. Re-run script when a login has failed."
    }

    # Step through each event collected, get geolocation
    #    for the IP Address, and add new events to the custom log
    foreach ($event in $events) {


        # $event.properties[19] is the source IP address of the failed logon
        # This if-statement will proceed if the IP address exists (>= 5 is arbitrary, just saying if it's not empty)
        if ($event.properties[19].Value.Length -ge 5) {

            # Pick out fields from the event. These will be inserted into our new custom log
            $timestamp = $event.TimeCreated
            $year = $event.TimeCreated.Year

            $month = $event.TimeCreated.Month
            if ("$($event.TimeCreated.Month)".Length -eq 1) {
                $month = "0$($event.TimeCreated.Month)"
            }

            $day = $event.TimeCreated.Day
            if ("$($event.TimeCreated.Day)".Length -eq 1) {
                $day = "0$($event.TimeCreated.Day)"
            }
            
            $hour = $event.TimeCreated.Hour
            if ("$($event.TimeCreated.Hour)".Length -eq 1) {
                $hour = "0$($event.TimeCreated.Hour)"
            }

            $minute = $event.TimeCreated.Minute
            if ("$($event.TimeCreated.Minute)".Length -eq 1) {
                $minute = "0$($event.TimeCreated.Minute)"
            }


            $second = $event.TimeCreated.Second
            if ("$($event.TimeCreated.Second)".Length -eq 1) {
                $second = "0$($event.TimeCreated.Second)"
            }

            $timestamp = "$($year)-$($month)-$($day) $($hour):$($minute):$($second)"
            $eventId = $event.Id
            $destinationHost = $event.MachineName# Workstation Name (Destination)
            $username = $event.properties[5].Value # Account Name (Attempted Logon)
            $sourceHost = $event.properties[11].Value # Workstation Name (Source)
            $sourceIp = $event.properties[19].Value # IP Address
        

            # Get the current contents of the Log file!
            $log_contents = Get-Content -Path $LOGFILE_PATH

            # Do not write to the log file if the log already exists.
            if (-Not ($log_contents -match "$($timestamp)") -or ($log_contents.Length -eq 0)) {
            
                # Announce the gathering of geolocation data and pause for a second as to not rate-limit the API
                #Write-Host "Getting Latitude and Longitude from IP Address and writing to log" -ForegroundColor Yellow -BackgroundColor Black
                Start-Sleep -Seconds 1

                # Make web request to the geolocation API
                # For more info: https://ipgeolocation.io/documentation/ip-geolocation-api.html
                $API_ENDPOINT = "https://api.ipgeolocation.io/ipgeo?apiKey=$($API_KEY)&ip=$($sourceIp)"
                $response = Invoke-WebRequest -UseBasicParsing -Uri $API_ENDPOINT

                # Pull Data from the API response, and store them in variables
                $responseData = $response.Content | ConvertFrom-Json
                $latitude = $responseData.latitude
                $longitude = $responseData.longitude
                $state_prov = $responseData.state_prov
                if ($state_prov -eq "") { $state_prov = "null" }
                $country = $responseData.country_name
                if ($country -eq "") {$country -eq "null"}

                # Write all gathered data to the custom log file. It will look something like this:
                #
                "latitude:$($latitude),longitude:$($longitude),destinationhost:$($destinationHost),username:$($username),sourcehost:$($sourceIp),state:$($state_prov), country:$($country),label:$($country) - $($sourceIp),timestamp:$($timestamp)" | Out-File $LOGFILE_PATH -Append -Encoding utf8

                Write-Host -BackgroundColor Black -ForegroundColor Magenta "latitude:$($latitude),longitude:$($longitude),destinationhost:$($destinationHost),username:$($username),sourcehost:$($sourceIp),state:$($state_prov),label:$($country) - $($sourceIp),timestamp:$($timestamp)"
            }
            else {
                # Entry already exists in custom log file. Do nothing, optionally, remove the # from the line below for output
                # Write-Host "Event already exists in the custom log. Skipping." -ForegroundColor Gray -BackgroundColor Black
            }
        }
    }
}

```

Run PowerShell Script to obtain Geodata from attackers within Virtual Machine. (This is obtained from Windows Event Viewer Security Logs, EventID 4625)
<p align="center"> 
<img src="https://imgur.com/ByHFtub.png" height="80%" width="80%" alt="SIEM"/>

Use Kusto Query Language (KQL) to identify Security EventID 4625 (Failed Log on Attempts)
<p align="center"> 
<img src="https://imgur.com/sZfxKsu.png" height="80%" width="80%" alt="SIEM"/>

Further steps:
- Create **"Custom Log"** in Log Analytics Workspace to communicate file path for failed RDP files from Azure Virtual Machine. (Powershell Script generates a file for failed RDP attempts.)
- Create **"Custom Fields/Extract Fields"** to create fields to input in World Map data.
- Set up Map using Kusto Query Language (KQL) in Microsoft Sentinel within Workbooks and assign values and data based on Custom Fields.

Query to map data:<br/>
```
FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, Latitude_CF, Longitude_CF, Country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != "" 
```

## Windows Event Viewer
Check Security for Windows Log and filter current log for EventID 4625 to analyze log on failure attempts on Virtual Machine.

<p align="center"> 
<img src="https://imgur.com/dL7fRcT.png" height="80%" width="80%" alt="SIEM"/>

## World Map Failed RDP Brute-force Attacks After 1 hour.

<p align="center"> 
<img src="https://imgur.com/WIRaGzQ.png" height="80%" width="80%" alt="SIEM"/>

## World Map Failed RDP Brute-force Attacks After 10 hours.

<p align="center"> 
<img src="https://imgur.com/23vVkFZ.png" height="80%" width="80%" alt="SIEM"/>

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
