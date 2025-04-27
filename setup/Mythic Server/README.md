# Mythic Server

To run this service a ubuntu server is used

## On the ubuntu server

1. update & upgrade apt
2. install docker-compose `sudo apt install docker-compose`
3. install make `sudo apt install make`
4. clone mythic `git clone https://github.com/its-a-feature/Mythic --depth 1`

_\*It's recommended to run Mythic on a VM with at least 2CPU and 4GB Ram._

5. cd into the "Mythic" folder and run `./install_docker_ubuntu.sh`

\* If after the installation docker service stopped, restart it with `sudo systemctl restart docker`

6. run `sudo make`
7. run `sudo ./mythic-cli start`
8. Check that port 7443 is accessible
9. connect to `{IP_ADDRESS:7443}` using the credentials `mythic_admin` and the output from `sudo ~/Mythic/mythic-cli config get MYTHIC_ADMIN_PASSWORD`

![mythicLogin](/images/mythicLogin.png)

10. Create a new operation clicking on the "operation Chimera" and set it as the current operation, I've named mine operation My30Days

![newOperation](/images/newOperation.png)

### Creating Payload

1. On terminal `sudo -E ~/Mythic/mythic-cli install github https://github.com/MythicAgents/Apollo.git`

_More information here_

    `https://github.com/MythicAgents`

![installApollo](/images/installApollo.png)

Apollo is now visible on Payloads/C2 services

![apolloInstalled](/images/apolloInstalled.png)

2. `sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http`

_More information here_
https://github.com/MythicC2Profiles

![mythicHttpProfile](/images/mythicHttpProfile.png)

Agent and C2 installed

![agentNc2](/images/agentNc2.png)

3. On mythic GUI click on `payloads>actions>Generate New Payload`

- target: windows
- Payload type: WinExe
- Commands included: All ">>"
- C2 profile included: http
- callback host: http://{MYTHIC_SERVER_IP_ADDRESS}
- Create payload
- copy "Download payload" link
