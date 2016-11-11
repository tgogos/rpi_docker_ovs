How to install Docker and OVS (openvswitch) on Raspberry Pi 3
=====================================

## Test environment
 - Raspberry Pi 3
 - Host OS: Raspbian Jessie Lite image (downloaded [2016-09-23-raspbian-jessie-lite.zip](http://director.downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2016-09-28/2016-09-23-raspbian-jessie-lite.zip) from the official site)
 - Docker image: `docker pull resin/rpi-raspbian:jessie-20160831`

1. Install docker
--------------
###### (more information can be found here: https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi)
 - run `curl -sSL https://get.docker.com | sh`

You can then search and pull images from the docker hub: https://hub.docker.com Examples of images tested:

 - resin/rpi-raspbian
 - resin/rpi-raspbian:jessie-20160831
 - armhf/ubuntu

---

2. Install Openvswitch
-------------------
###### (more information can be found here: http://containertutorials.com/network/ovs_docker.html)

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

---


Docker - quick reference
========================
## Images
 - search for images on https://hub.docker.com
 - pull image example: `docker pull resin/rpi-raspbian`
 - list available (already pulled) images `docker images`
 - delete image example: `docker rmi resin/rpi-raspbian`

## Containers
 - run a new container example: `docker run -it resin/rpi-raspbian`
    - docker creates a `new` container based on the image selected and assigns a random name, for example `lonely_keller`
    - run a new container providing name: `docker run --name ndpi_test -it resin/rpi-raspbian`
    - changes to this container (apt-get install... etc) will not affect future containers being generated based on the same image
 - start the container with: `docker start lonely_keller`
 - stop the container with: `docker stop lonely_keller`
 - rename a container with: `docker rename old_container_name new_container_name`
 - get access to the console of every running container with: `docker attach container_name`
 - return back to the host by pressing `Ctrl+p` and then `Ctrl+q`
 - list the running containers: `docker ps`
 - list all the containers: `docker ps -a`
 - remove container: `docker rm container_name`
 - remove all containers: `docker rm $(docker ps -a -q)`
  - maybe you will have to stop all containers first: `docker stop $(docker ps -a -q)`


### Possible states
figure from: [https://docs.docker.com/engine/reference/api/docker_remote_api/](https://docs.docker.com/engine/reference/api/docker_remote_api/)
 ![docker events](event_state.png)


## Docker networking
[https://blog.docker.com/2016/01/webinar-qa-docker-networking/](https://blog.docker.com/2016/01/webinar-qa-docker-networking/)

`docker0` (172.17.0.1/16) is the default bridge network that new containers are attached to if nothing else is specified.
### Create a new network and run a container
```bash
docker network create frontend # (172.18.0.1/16)
docker network ls
docker run  -d --name rose --net=frontend busybox top
#busybox is the docker image to pull (if not already available) and then start with command "top"
docker exec rose ifconfig
```
### Ping the container
```bash
docker run --rm busybox ping -c 4 rose
# you get ping: bad address 'rose'
# the above is because of network isolation. By default docker0 is used so 'rose' cannot be found.
# it works only if network is specified, like this:
docker run --rm --net=frontend busybox ping -c 4 rose
```

### Create another network and connect the container
```bash
docker network create backend # (172.19.0.1/16)
docker network connect backend rose
docker exec rose ifconfig
```

### Disconnect from a network and destroy it
```bash
docker network disconnect backend rose
docker exec rose ifconfig
docker network rm backend
docker network ls
```

## Copy files from host to container
`docker cp /path/filename container_name:/path/filename`
