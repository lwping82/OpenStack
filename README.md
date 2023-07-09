# Setting up single node OpenStack
1. Install OpenStack into Ubuntu
   sudo snap install microstack --beta --devmode
   sudo microstack init --auto --purge
2. Instance created in OpenStack is fully locked down by default. Execute following to enable it to hook up to host's network.
   '# su -
   '# micostack.openstack
   '# subnet set --dhcp external-subnet
   '# subnet set --dhcp test-subnet
   '# subnet set --dns-nameserver 8.8.8.8 external-subnet
   '# subnet set --dns-nameserver 8.8.8.8 test-subnet
   '# network set --share external
   '# network set --share test
3. Common default ports already been taken by OpenStack. Port forwarding is needed is standard port is to be used within the instance.
   a. Setup host machine port forwarding
       sudo iptables -t nat -A POSTROUTING -s 10.20.20.1/24 ! -d 10.20.20.1/24 -j MASQUERADE
       sudo iptables -t nat -A PREROUTING -p tcp --dport 8090 -j DNAT --to-destination 10.20.20.80:8090
       sudo iptables -t nat -A POSTROUTING ! -o lo -j MASQUERADE
4. To open port in OpenStack's instance
   Browse to network > security groups
   select the implemented rules
   click on "Manage Rules"
   Add the rule (Ingress and/or Egress depending on requirement)

5. To retrieve Microstack default password
   sudo snap get microstack config.credentials.keystone-password

# Create a service that is always listening in host machine
1. sudo vim /etc/systemd/system/listen-8090.service
2. Place below into the file
   [Unit]
   Description=Permanently listen at :::8090
   After=network-online.target

   [Service]
   User=root
   ExecStart=/usr/bin/nc -l6 8090
   ExecStop=/usr/bin/killall -s KILL nc
   Restart=always
   RestartSec=1

   [Install]
   WantedBy=multi-user.target

3. Managing the service
   sudo systemctl daemon-reload
   sudo systemctl enable listen-8090.service
   sudo systemctl start listen-8090.service
   sudo systemctl status listen-8090.service
   sudo systemctl stop listen-8090.service
   sudo systemctl disable listen-8090.service

# Setting up Docker
1. sudo snap install docker
2. docker --version
3. sudo docker run hello-world
4. sudo docker images
5. sudo docker ps -a
6. sudo docker ps

# Test sample application in Docker
1. docker run --rm -it -p 8090:80 yeasy/simple-web:latest
   
