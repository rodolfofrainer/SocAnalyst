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

### Starting the service

1. `systemctl daemon-reload`
2. `systemctl enable elasticsearch.service`
3. `systemctl start elasticsearch.service`
4. `systemctl status elasticsearch.service`

Elasticsearch is installed and running

![ElasticSearchStatus](/images/elasticSearch%20status.jpg)

## Installing Kibana

1. On `https://www.elastic.co/downloads/kibana` pick the desired platform and `wget` it.
   In my case that's `https://artifacts.elastic.co/downloads/kibana/kibana-8.17.4-amd64.deb`
2. Run `dpkg -i <filename>`
3. We need to configure somethings on the service, `vim /etc/kibana/kibana.yml`
4. Uncomment `#server.port: 5601`
5. Uncomment `#server.host: "localhost"` and change the value to "0.0.0.0" or your ip address from "ip a"
6. `systemctl daemon-reload`
7. `systemctl enable kibana.service`
8. `systemctl start kibana.service`
9. `systemctl status kibana.service`

![KibanaStatus](/images/kibana%20status.jpg)

### Generating elastic search token

1.  `cd /usr/share/elasticsearch/bin`

    \* `ls` produces the following output:

        elasticsearch                          elasticsearch-geoip             elasticsearch-setup-passwords
        elasticsearch-certgen                  elasticsearch-keystore          elasticsearch-shard
        elasticsearch-certutil                 elasticsearch-node              elasticsearch-sql-cli
        elasticsearch-cli                      elasticsearch-plugin            elasticsearch-sql-cli-8.17.4.jar
        elasticsearch-create-enrollment-token  elasticsearch-reconfigure-node  elasticsearch-syskeygen
        elasticsearch-croneval                 elasticsearch-reset-password    elasticsearch-users
        elasticsearch-env                      elasticsearch-saml-metadata     systemd-entrypoint
        elasticsearch-env-from-file            elasticsearch-service-tokens

2.  `elasticsearch-create-enrollment-token` needs to be invoked. run the following command `./elasticsearch-create-enrollment-token --scope kibana`
3.  Save the token into `~/kibanaToken` (this is not safe but this way we won't lose the output)

### Accessing Kibana

Currently Kibana is reacheable through port 5601, however our "Network Security Group" doesn't allow traffic through this port.

1.  On Azure's portal access "Resource Groups" and the Resource with type "Network security group" > Settings > Inbound security Rules > Add > Destination port ranges = 5601
    ![inboundRuleConfig](/images/azureInboundRuleConfig.jpg)
2.  On the VM's terminal > sudo ufw allow 5601
3.  sudo ufw allow 9200 (this will be used for the fleet server)
4.  Ensure `elasticsearch.service` and `kibana.service` are running.
5.  On a browser on your pc navigate to `http://<YOUR PUBLIC IP ADDRESS>:5601`
6.  Paste the token generated earlier into the field
7.  `sudo /usr/share/kibana/bin/kibana-verification-code` and copy the code into the field
8.  The service will now start and ask for credentials. The credentials were created during the elasticsearch setup and I've saved them on the file "elastic".

        User: elastic
        Password: aHnBmDBnO+WgtAkT=DS7
        *If the password is lost or forgotten the command `/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic' should be used.

![elasticDashBoard](/images/elasticDashboard.jpg)

8.  Finally on the burger menu on the left side select "Alerts"

        The error "Failed to retrieve detection engine privileges" has been raised.

9.  Run the command `sudo /usr/share/kibana/bin/kibana-encryption-keys generate` is run and it generates a assortment of keys.
10. Copy them into `/etc/kibana/kibana.yml` and restart the service.

![elasticAlert](/images/elasticAlerts.jpg)

## Data ingestion

After Sysmon's installation into the server, it is time to create policies that will allow kibana to receive the logs

Clicking the button "Add integration" on the dashboard, look for "Custom Windows Event Logs".

![Custom Windows Event Logs](/images/Custom%20Windows%20Event%20Logs.jpg)

Add the integration and label it accordingly, mine are called "Defender Logs" and "Sysmon Logs".

![addPolicy](/images/addPolicy.png)

### Channel name

To find the Channel name the event viewer on the Windows server is used. Navigate to: Event Viewer > Applications and Services Logs > Microsoft > Windows > {Service}

#### Service Names

Sysmon: Sysmon
Defender: Windows Defender

On both of these services the cannel name is in: {Service} > Operational > Properties > Full Name

![fullName](/images/operationalChannelName.png)

On sysmon that's all there is, no need for further configuration.

---

### Defender

On Windows defender Policy > Custom Windows event logs > Event ID

add the numbers 1116,1117,5001

## ![CustomEventIds](/images/customEventIds.png)

Add both of policies to the windows policy

![addToPolicy](/images/addToPolicy.png)

If everything was done properly you should see somthing like this

![HealthyPolicy](/images/healthyPolicy.jpg)

\* **_If the metrics are not visible, this could mean a problem with communication, check if all appropriate ports are reacheable; If more than 1 VLan is used, as in my case, make sure that Azure's vnet resource has a "peering" configured to allow communication between vlans_**
