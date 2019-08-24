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


