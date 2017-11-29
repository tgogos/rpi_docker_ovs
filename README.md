How to install Docker and OVS (openvswitch) on Raspberry Pi 3
=====================================

## Test environment
 - Raspberry Pi 3
 - Host OS: Raspbian Jessie Lite image (downloaded [2016-09-23-raspbian-jessie-lite.zip](http://director.downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2016-09-28/2016-09-23-raspbian-jessie-lite.zip) from the official site)
 - Docker image: `docker pull resin/rpi-raspbian:jessie-20160831`

### How to prepare your SD card for the Raspberry Pi:
 - go to https://www.raspberrypi.org/downloads/raspbian/ and download the image
 - flash your SD card. There are many ways, something like this might help: `sudo ddrescue -D --force 2016-09-23-raspbian-jessie-lite.img /dev/sdb`

### 1. Install docker

(more information can be found here: https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi)
 - run `curl -sSL https://get.docker.com | sh`

You can then search and pull images from the docker hub: https://hub.docker.com Examples of images tested:

 - resin/rpi-raspbian
 - resin/rpi-raspbian:jessie-20160831
 - armhf/ubuntu

---

### 2. Install Openvswitch

(more information can be found here: http://containertutorials.com/network/ovs_docker.html)

`sudo apt-get install openvswitch-switch`

### Install ovs-docker utility
```bash
cd /usr/bin
wget https://raw.githubusercontent.com/openvswitch/ovs/master/utilities/ovs-docker
chmod a+rwx ovs-docker
```

### Create an OVS bridge

Here we will be adding a new OVS bridge and configuring it, so that we can get the containers connected on the different network.

```bash
ovs-vsctl add-br ovs-br1
ifconfig ovs-br1 173.16.1.1 netmask 255.255.255.0 up
```

 - Add a port from OVS bridge to the Docker Container
 - Create two ubuntu Docker Containers

```bash
docker run -t -i --name container1 ubuntu /bin/bash
docker run -t -i --name container2 ubuntu /bin/bash
```
 - Connect the container to OVS bridge

```bash
ovs-docker add-port ovs-br1 eth1 container1 --ipaddress=173.16.1.2/24
ovs-docker add-port ovs-br1 eth1 container2 --ipaddress=173.16.1.3/24
```
 - Test the connection between two containers connected via OVS bridge using Ping command

### Reset OVS
```bash
#!/bin/bash
sudo systemctl stop openvswitch
sudo systemctl disable neutron-openvswitch-agent

sudo systemctl stop openvswitch
sudo rm -rf /var/log/openvswitch/*
sudo rm -rf /etc/openvswitch/conf.db
sudo systemctl start openvswitch
sudo ovs-vsctl show

```
