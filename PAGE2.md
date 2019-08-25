##ProxyCreation
# Deploying mutli LXC containers web services and creating an nginx loadbalancer for the webservices

Step1: Create three containers identified by their root pages
Step 2: Creating a lb from nginx by changing the default.conf
lxc file edit lb/etc/nginx/conf.d/default.conf
```
#This is a default site configuration which will simply return 404, preventing
#chance access to any other virtualhost.
upstream lb-web{
        server web1;
        server web2;
        server web3;
}
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location / {
        proxy_pass http://lb-web;
        }
}
```

lxc exec lb -- /etc/init.d/nginx restart


Step 3: Map port from the lb container to the localhost

sysctl net.ipv4.ip_forward
return 1
Routinng the container port 80 to the localhost using iptables

sudo iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80\
-j DNAT --to $lbIp:80

#GaleraClusterOnLXC
Deploying a 2-node galera cluster on LXC container pairs
for demostrating Database replication 

1. Creating a ubuntu(node1) container and configuring maria-db server 

lxc launch images:ubuntu/16.04 node1
lxc exec node1 -- bin/bash
apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.utexas.edu/mariadb/repo/10.1/ubuntu xenial main'
apt-get update
apt-get install mariadb-server -y
mysql_secure_installation

sudo echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a && printf '\n%s\n' 'Ram-cache and Swap Cleared'

2. If the host machine dont have much RAM(<=512MB) than reducing the 
innoDB pool size for  "node1"
lxc file edit node1/etc/mysql/my.conf
Add the following line to my.conf
[mysqld]
innodb_buffer_pool_size=128M
\#innodb_log_buffer_size=8M
\#innodb_file_per_table=1
\#innodb_open_files=400
\#innodb_io_capacity=400
\#innodb_flush_method=O_DIRECT


3. Configuring the galera cluster configuration file galera.cnf in /etc/mysql/conf.d

#########################################
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://10.176.106.153,10.176.106.175"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="10.176.106.153"
wsrep_node_name="node1"
##################################################

4. Allow the corresponding firewall rules on node1 using ufw utiliy
apt-get install ufw -y
ufw enable
ufw allow 3306,4444,4567,4568/tcp
ufw allow 4567/udp
ufw status

exit


5. Take the snapshot of node1 container and start a new node2 from node1 snapshot
**Node2 container will be exact replication of Node1 container**
lxc snapshot node1 snap0
lxc copy node1/snap0 node2

6. Make the changes in galera.conf file for node2 

7. Stop the mysql service on both nodes
lxc exec node1 -- systemctl stop mysql
lxc exec node2 -- systemctl stop mysql

8. Start the galera cluster mode and test the following commands on node1 and node2

[node1]
lxc exec node1 -- galera_new_cluster
lxc exec node1 -- mysql -u root -p -e "show status like 'wsrep_cluster_size'"

[node2]
lxc exec node2 -- systemctl start mysql
lxc exec node2 -- mysql -u root -p -e "show status like 'wsrep_cluster_size'"
	
9. If all works fine than test for replication on node1 and node2
lxc exec node1 -- mysql -u root -p -e "show databases;"
lxc exec node2 -- mysql -u root -p -e "show databases;"
lxc exec node1 -- mysql -u root -p -e "create database galera_cluster_test;"
lxc exec node1 -- mysql -u root -p -e "show databases;"
lxc exec node2 -- mysql -u root -p -e "show databases;

For more detailed info:
https://fxdata.cloud/tutorials/configure-a-galera-cluster-with-mysql-5-6-on-ubuntu-16-04




