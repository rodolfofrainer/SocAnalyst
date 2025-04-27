# Fleet Server

Create a Ubuntu server

Aiming for the cheapest option on Azure we achieve $0.01/h

![FleetPrice](/images/fleetCost.jpg)

While the server is deploying I'll head to our elastic GUI and on the left side menu scroll to management and click on "Fleet"

![fleetElastic](/images/fleetElastic.jpg)

Add Fleet server and populate the required information

![Fleetsetup](/images/fleetFirstStepSetup.jpg)

If everything goes well you can go to step 2. Installing fleet server to a centralized host.
Just follow the instructions provided.

![success](/images/fleetSecondSuccessStepSetup.jpg)

Continue when given the choice and you should be met with this screen:

![thirdSetup](/images/fleetThirdStepSetup.jpg)

1. Name it and create the policy. In my case it is "WindowsPolicy"
2. Select Windows as platform
3. Go into the windows server VM
4. Open PowerShell as Administrator
5. Paste Selected command into Powershell and run

   \* If an error is thrown, check Firewall rules on fleet server, if none of that works try adding the `--insecure` flag at the end of the command

6. Once both the server and Agent are successfuly created data will begin to flow into the server

   ![HealthyFleet](/images/HealthyFleet.jpg)

7. On the discovery page the logs should be visible after a couple of minutes

   ![logs](/images/logs.jpg)
