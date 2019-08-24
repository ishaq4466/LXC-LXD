LXC/LXD COntainers:
LXC=>Linux Container or CLI for lxd
LXD=> Linux daemon

* They are full OS containers i.e machine containers
* Where as Docker are Application container which contains only the 
specific binaries which the application required
* LXC/LXD containers full as linux hypervisors 
* Docker container supports some enhace version of LXD like versioning,port-mapping 
UFS therefore LXD is the backed version of LXD containers
* Very light weight VM
* Can run LXD inside LXD

1. Installation of LXC/LXD:
$ sudo apt-get install lxd lxd-client 
$ sudo lxd init

2. Trouble shooting the installation
Networking on the host has failed. Remove and re-add the lxdbr0 Linux Bridge.    
if you get an error about a missing parent for an ethernet device, what is the solution
sudo dpk-reconfigure -p medium lxd
lxd init

3. Conatiner creation, Listing, stopping the containers
* Listing all the running lxd containers
lxc list 
* Launching a containers from image
lxc launch <image name> <container name>
lxc launch ubuntu:16.04 ubuntu
lxc launch images:alpine/3.6 web

lxc stop ubuntu
lxc start ubuntu
lxc info ubuntu
lxc delete --force ubuntu

* Listing all the lxd images and coping the images from the remotes
lxc image list 
lxc image copy images:alpine/3.5 local: --alias alp 
* Execution of commands from inside or outside of container
lxc exec web -- apk add nginx
lxc file push index.html web/var/www/index.html
lxc exec web -- rc-update add nginx default
lxc exec ubuntu -- /bin/bash
lxc exec ubuntu -- apt-get update && apt-get install git
lxc file edit [container]/[path/to/file]


Snapshots:
* Creating a snapshot of running container
lxc snapshot web snap1 
* Restroing the web container to previous state of snap0
lxc restore web snap0
* Creating a copy of containers from the snapshots
lxc copy web/snap1 web2
lxc start web2
lxc copy web1 web2
web will the new container which will be craeted from web container's 
snapshot



##4.Remotes 
#Congiguring an lxd remote on other machine
1. Get the local ip of that configuration machine
ifconfig

2. Configure the remote the following command

lxc config set core.https_address "[::]:8443"
lxc config set core.trust_password secret


3. Add the remote to the first machine
lxc remote add second <ip-addr>:8443 --password=secret

4. For nested container i.e **lxd inside lxd** set secuity nesting true 
lxc config set ubuntu security.nesting true

# Publishing, exporting, examing the container's images
By creating the images helps us to distribute and even export all images
images are tarballs
```
mkdir web-img
lxc publish web/snap0 --alias nginx-alpine
#lxc publish <container>/<snapshot> --alias <img-tag>
lxc image list
#lxc image export <img-name> <dir>/
lxc image export nginx-alpine web-img/
```


zfsutils-linux


# This is a default site configuration which will simply return 404, preventing
# chance access to any other virtualhost.

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/;
}

#ubuntu
1.
export LOCAL_IP=10.176.106.107
export REMOTE_IP=10.176.106.67
ip link add contgre type gretap remote $REMOTE_IP local $LOCAL_IP ttl 225
2.
brctl addif lxdbr0 contgre
ip link set contgre up
7.
lxc remote add bravo $REMOTE_IP:8443 --password=secret

#ubuntu1
3.
export LOCAL_IP=10.176.106.67
export REMOTE_IP=10.176.106.107
4.
brctl addbr multibr0
ifconfig multibr0 up
lxd init
5.
ip link add contgre type gretap remote $REMOTE_IP local $LOCAL_IP ttl 225
brctl addif multibr0 contgre
ip link set contgre up
6.
lxc config set core.https_address "[::]:8443"
lxc config set core.trust_password secret


