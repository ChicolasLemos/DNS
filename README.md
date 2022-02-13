# DHCP AND DNS

On AWS you need to select your sever and go to actions, networking, change scouce/destination check and do stop!!!

Server (this will give internet to the client)
```
sudo hostnamectl set-hostname name.domain
nano key.pem (add the public key in here)
chmod 600 key.pem
sudo su -
apt update && apt upgrade -y
apt install netfilter-persistent iptables-persistent -y
iptables -F
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
netfilter-persistent save
nano /etc/sysctl.conf
 - uncomment net.ipv4.ip_forward=1
sysctl -p
systemctl restart network
```
Client (net)
```
sudo hostnamectl set-hostname name.domain
nano /etc/netplan/50-cloud-init.yaml (dont use tabs!)
 -add dhcp4-overrides:
          use-dns: false
          use-routes: false
      nameservers:
          addresses: [google dns or yours if you already have]
      gateway4: server ip
 save
netplan try and enter
ping 1.1.1.1 to see if you have net
```
Server DNS
```
apt install bind9
cd /etc/bind
nano named.conf.options
 - uncomment forwarders {
             8.8.8.8;
             };
cp db.local db.yourdomain
cp db.127 db.(first 2 ips from your private ip)
nano db.yourdomain
 - ctrl+\ then change localhost to your domain
 - change the ips to yours
 - change or add the sites that you want
nano db.ip
 - ctrl+\ change the localhost
 - change the ips
remember this is the reverser zone
nano named.conf.default-zones
 - copy the examples of zones
nano named.conf.local
 - put the zones here and change to yours
systemctl restart bind9
dig yoursite
```
DHCP
```
apt install isc-dhcp-server
nano /etc/default/isc-dhcp-server
 - INTERFACEv4="eth0 or eth1"
cd /etc/dhcp/
nano dhcpd.conf
 - remove the # on authoritative
 - subnet x.x.x.x netmask x.x.x.x {
   range x.x.x.x x.x.x.x;
   option domain-name-servers ns1.internal.DOMAIN;
   option domain-name "internal.DOMAIN";
   option subnet-mask x.x.x.x;
   option routers x.x.x.x;
   option broadcast-address x.x.x.x;
   default-lease-time 800;
   max-lease-time 7200;
 }
systemctl enable –now isc-dhcp-server
systemctl status isc-dhcp-server

test

apt install build-essential
cd ~
git clone https://github.com/saravana815/dhtest.git
cd dhtest/
make

duplicate your session
./dhtest -m 00:00:11:22:33:44 -i eth0
in the other do:
tcpdump -i eth0 -nv port 67
