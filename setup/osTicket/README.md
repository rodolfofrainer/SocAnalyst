# osTicket

## VM

A simple Windows 10 server, with minimum specs will suffice

## Services

### XAMPP

`https://www.apachefriends.org/download.html`

1. Navigate to the instalation folder and edit the properties file

_by default `C:\xampp\properties.ini`_

2. Change the line `apache_domainname={PUBLIC_IP_ADDRESS}`

_Create firewall rules allowing 80,443 inbound traffic_

3. Start the Apache and MySQL services
4. Open Apache's Admin panel with XAMPP Control Panel

![xamppControlPanel](/images/xamppControlPanel.png)

5. Create a new user

![newSqlUser](/images/newSqlUser.png)

_I've given mine ALL PRIVILEGES_

6. Create a new table with the name `myticketdb`

### osTicketing

`https://osticket.com/download/`

1. Extract all files and paste `scripts` and `upload` into `C:\xampp\htdocs\osticket`

2. `C:\xampp\htdocs\osticket\upload` rename `ost-sampleconfig.ini` to `ost-config.ini`

3. On a browser go to `localhost/osticket/upload` and follow the steps requested

4. `cd C:\xampp\htdocs\osTicket\upload\include`
5. `icacls ./ost-config.php /reset`
6. it can be accessed from any computer at `{IP_ADDRESS/osticket/upload}`
7. account can be accessed through `{IP}/osticket/upload/scp`
   ![ticketControlPanel](/images/ticketControlPanel.png)
