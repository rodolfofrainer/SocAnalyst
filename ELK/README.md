# Create ELK server

## Azure Ubuntu server

Create a resource group named "Elk-myDfir-2025" and an Ubuntu VM on Azure.

I'm aiming for dirty cheap, Copilot has helped me by picking a cheaper disk model.

## Installing Elastic

`wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.4-amd64.deb`

installing the package: `dpkg elasticsearch-8.17.4-amd64.deb`

### A similar output will be displayed

    sudo dpkg -i elasticsearch-8.17.4-amd64.deb
    Selecting previously unselected package elasticsearch.
    (Reading database ... 94939 files and directories currently installed.)
    Preparing to unpack elasticsearch-8.17.4-amd64.deb ...
    Creating elasticsearch group... OK
    Creating elasticsearch user... OK
    Unpacking elasticsearch (8.17.4) ...
    Setting up elasticsearch (8.17.4) ...
    --------------------------- Security autoconfiguration information ------------------------------
    Authentication and authorization are enabled.
    TLS for the transport and HTTP layers is enabled and configured.
    The generated password for the elastic built-in superuser is : aHnBmDBnO+WgtAkT=DS7
    If this node should join an existing cluster, you can reconfigure this with
    '/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'
    after creating an enrollment token on your existing cluster.
    You can complete the following actions at any time:
    Reset the password of the elastic built-in superuser with
    '/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.
    Generate an enrollment token for Kibana instances with
    '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.
    Generate an enrollment token for Elasticsearch nodes with
    '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node'.
    -------------------------------------------------------------------------------------------------
    ### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    ### You can start elasticsearch service by executing
    sudo systemctl start elasticsearch.service

---

\* You can `echo "token: aHnBmDBnO+WgtAkT=DS7" > elastic` to save it into a file

### Getting local Ip

#### Use `ip a` to find your local address

---

    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:22:48:e5:cf:85 brd ff:ff:ff:ff:ff:ff
    inet <b>10.0.0.4</b>/24 metric 100 brd 10.0.0.255 scope global eth0

---

In my case it is 10.0.0.4, save/remember this number somehow

### Configuring elasticsearch

`sudo vim /etc/elasticsearch/elasticsearch.yml`

    1. Find the line "#network.host", uncomment and update the value with the IP address we just got.
    2. Fint the line "#http.port" and uncomment it
    3. Save and quit
