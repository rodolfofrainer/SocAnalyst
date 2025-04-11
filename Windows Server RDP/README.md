# Windows server

## New Resource Group

I will create a new resource group so it's easy to separate this server resources from ELK's.

## The server

I will be using a `Windows Server 2016` with Azure Spot Discount.

All other options are set as the cheapest and weakest versions in order to save money as this will not be a demanding server.

\* Connection through port 3389 has to be allowed or you will be locked out out of the server

## Sysmon

- Download Sysmon from the following page

        https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

- For this project the olaf config file will be used

        https://raw.githubusercontent.com/olafhartong/sysmon-modular/refs/heads/master/sysmonconfig.xml

### Installing

1.  Extract Sysmon's zip file and move olaf config into the newly created folder.
2.  On PowerShell with administrator privileges navigate into the sysmon's folder and use the following command

        .\Sysmon64.exe -i sysmonconfig.xml

3.  Accept the terms and conditions
