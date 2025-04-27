# Ubuntu Server

This server will be used only to gather logs from possible attacks, the minimum specs are recommended

## Deployment Settings

At this point I've run out of vCPUs on Azure's free tier.

That meant that I would have to create my VMs on another region.

### During creation

Be sure that the IP range is appropriate for the vLan peering, it might need further configuration. In my case I've configured it with 10.2.0.0/24 being the range, which produce the VM with the address of 10.2.0.4, not conflicting with existing addresses.

Configure the vLan peering on Azure's portal

## On kibana GUI

Go to Management>Fleet>Agent Polocies> Create agent Policy

Name it `LinuxPolicy`

Go back to Agents>Add agent

1. Select `LinuxPolicy`
2. copy the command in item 3 and run it on the server's terminal
3. Confirm that agent enrollment after installation complete
4. On `Discovery` search for agent.name field and see if server is shown
