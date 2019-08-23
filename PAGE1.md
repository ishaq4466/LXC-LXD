LXC/LXD COntainers:
LXC=>Linux Container
LXD=> Linux daemon

* They are full OS containers i.e machine containers
* Where as Docker are Application container which contains only the 
specific binaries which the application required
* LXC/LXD containers full as linux hypervisors 
* Docker container supports some enhace version of LXD like versioning 
UFS therefore LXD is the backed version of LXD containers


1. Installation of LXC/LXD:
$ sudo apt-get install lxd lxd-client 
$ sudo lxd init

2. Trouble shooting the installation
sudo dpk-reconfigure -p medium lxd

3. Conatiner creation, Listing, stopping the containers
* Listing all the running lxd containers
lxc list 
* Launching a containers from image
lxc launch <image name> <container name>
lxc launch ubuntu:16.04 ubuntu
lxc stop ubuntu
lxc start ubuntu
lxc info ubuntu

* Listing all the lxd images and coping the images from the remotes
lxc image list 
lxc image copy images:alpine/3.5 local: --alias alp 
* Execution of commands from inside or outside of container

lxc exec ubuntu -- /bin/bash
lxc exec ubuntu -- apt-get update && apt-get install git














