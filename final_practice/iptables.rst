Port Forwarding
================

ip:

    * iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 80 -j DNAT --to-destination 192.168.122.80:80

    * iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 22001 -j DNAT --to-destination 192.168.122.80:22

    * iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 22002 -j DNAT --to-destination 192.168.122.81:22

    * iptables -I FORWARD -m state -d 192.168.122.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT

eth0:

    * iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.122.81:80
    * iptables -A FORWARD -p tcp -d 192.168.122.80 --dport 8080 -j ACCEPT

wlan0:

    * iptables -A PREROUTING -t nat -i wlan0 -p tcp --dport 80 -j DNAT --to 192.168.122.80:80
    * iptables -A FORWARD -p tcp -d 192.168.122.80 --dport 80 -j ACCEPT

    * iptables -D PREROUTING -t nat -i wlan0 -p tcp --dport 80 -j DNAT --to 192.168.122.80:80
    * iptables -D FORWARD -p tcp -d 192.168.122.80 --dport 80 -j ACCEPT

iptables-save

SSH
----

# iptables -I FORWARD -m state -d 192.168.122.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT
# iptables -t nat -I PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.122.80:2222

Delete rules
=============

Execute the same commands but replace the "-A" with "-D". For example:

iptables -A INPUT -i eth0 -p tcp --dport 443 -j ACCEPT

becomes

iptables -D INPUT -i eth0 -p tcp --dport 443 -j ACCEPT

    * iptables -A PREROUTING -p tcp -d 192.168.1.39 --dport 80 -j DNAT --to-destination 192.168.122.80:80
    * iptables -A FORWARD -p tcp -d 192.168.122.81 --dport 8080 -j ACCEPT

