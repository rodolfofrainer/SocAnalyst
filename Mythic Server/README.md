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
