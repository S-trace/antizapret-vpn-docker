# Antizapret VPN server
Easy-to-start docker container with antizapret vpn server for selfhosting.

## About
Easy-to-use docker image based upon original [Atnizapret LXD image](https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/). 

## Improvements
 - [Apple DNS fix](https://github.com/xtrime-ru/antizapret-vpn-docker/blob/master/patches/kresd.conf#L3);
 - [RU domains excluded from antizapret](https://github.com/xtrime-ru/antizapret-vpn-docker/blob/master/patches/kresd.conf#L13);
 - [IDN domains fix](https://github.com/xtrime-ru/antizapret-vpn-docker/blob/master/patches/fix.sh#L5);
 - [Additional domains list](https://github.com/xtrime-ru/antizapret-vpn-docker/blob/master/config/include-hosts-custom.txt);
 - Switch to Ubuntu 24.04 from Debian 10;
 - Upgrade to OpenVPN 2.6+ and install [openvpn-dco](https://openvpn.net/as-docs/tutorials/tutorial--turn-on-openvpn-dco.html) kernel extension for maximum performance;
 - Rules for Youtube, Google, Microsoft, OpenAI
 - Start sequence optimization. Container start times reduced from minutes to seconds. 


## Installation
0. Install docker
    ```shell
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

1. Copy this repository, build container, and run it.
    ```shell
    git clone https://github.com/xtrime-ru/antizapret-vpn-docker.git antizapret
    cd antizapret
    docker compose pull
    docker compose up -d
    ```
2. Download .ovpn configuration file for your openvpn client from `keys/client` folder. 
There will be udp and tcp versions of the config. For better performance use upd.
Tcp version will be better for unstable conditions.

## Update:
 
```shell
git pull
docker compose pull
docker compose up -d
```
## Enable OpenVPN Data Channel Offload (DCO)
OpenVPN Data Channel Offload (DCO) provides performance improvements by moving the data channel handling to the kernel space, 
where it can be handled more efficiently and with multi-threading.
TLDR: increase speed and reduce CPU usage for server.

Unfortunately kernel extensions cant be installed in docker.   
Install it on **host** machine

Ubuntu 24.04+:
```shell
apt update && apt upgrade

# Please reboot your system after upgrade!

apt install -y efivar
apt install -y openvpn-dco-dkms
```

Ubuntu 20.04+:
```shell
apt update && apt upgrade

# Please reboot your system after upgrade!

apt install -y efivar dkms linux-headers-$(uname -r)
wget http://de.archive.ubuntu.com/ubuntu/pool/universe/o/openvpn-dco-dkms/openvpn-dco-dkms_0.0+git20231103-1_all.deb
dpkg -i openvpn-dco-dkms_0.0+git20231103-1_all.deb
```

## Keys persistence
Server keys are stored in `keys/server/` and client keys - in `keys/client/`.
Keys are persistent between container and host restarts.

To generate new keys - remove files and container again:
```shell
docker compose down
rm -rf keys/{client,server}/keys/*.{crt,key}
docker compose up -d
```

## Additional domains
Any domain and/or IP can be added or excluded from list with [config files](https://github.com/xtrime-ru/antizapret-vpn-docker/tree/master/config)
This lists are added/excluded to/from automatically generated lists of domains and IP's. 
To apply changes: reboot container and wait few minutes for new rules generation.

## Environment Variables
You can define this variables in docker-compose file for your needs
 - `DNS=1.1.1.1` - DNS server to resolve domains. By default - system/docker dns
 - `DNS_RU=77.88.8.8` - Russian DNS server. Used to fix issues with geo zones mismatch for domains like apple.com

## Extra information
[OpenWrt setup guide](https://github.com/xtrime-ru/antizapret-vpn-docker/blob/master/docs/guide_OpenWrt.txt) - how to setup OpenWrt router with this solution to keep LAN clients happy.

## Links
- Link to original project website: https://antizapret.prostovpn.org
- Repositories:
    - https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/
    - https://bitbucket.org/anticensority/antizapret-pac-generator-light/src/master/
