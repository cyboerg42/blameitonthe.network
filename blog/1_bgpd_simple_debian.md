# simple bgp session with debian 10

install quagga

```
apt-get update && apt-get install quagga-bgpd
```

now enable the bgpd systemd service

```
systemctl enable bgpd
systemctl start bgpd
```

get into the quagga vtysh

```
vtysh
```

enable it and get into configure mode

```
enable
configure terminal
```

add the neigbor

```
router bgp 64812
bgp router-id 100.64.12.1

network 192.168.1.0/24

neighbor 100.64.12.2 remote-as 64813
neighbor 100.64.12.2 password som3securestr!ng42
neighbor 100.64.12.2 route-map ROUT_64813 out
neighbor 100.64.12.2 route-map RIN_64813 in

!
```

but what is this route map? - good point. it's a rule-set, what announcements we will send/forward/receive to/from this peer. we'll configure it later.
now we'll configure an prefix list for each route-map.

```
ip prefix-list LOUT_64813 seq 5 permit 192.168.1.0/24
!
ip prefix-list LIN_64813 seq 5 permit 192.168.0.0/16
!
```

now we configure the route-map :)

```
route-map ROUT_64813 permit 10
match ip address prefix-list LOUT_64813
set community none
!
route-map RIN_64813 permit 10
match ip address perfix-list LIN_64813
!
write
```

again! some new shit. what's about those communities? - simple, it's arguments that you parse to this peer. those arguments are used for all kind of automatic routeing stuff. but we'll keep that as silent as possible for now.

finally restart bgpd

```
systemctl restart bgpd
```

enable ipv4 forwarding! ;)

```
echo "1" > /proc/sys/net/ipv4/ip_forward
```

and tell iptables to allow forwarding between the interfaces...

```
iptables -A FORWARD -s 192.168.1.0/24 -i eth0 -o eth1 -j ACCEPT
iptables -A FORWARD -S 192.168.0.0/16 -i eth1 -o eth0 -j ACCEPT
```
