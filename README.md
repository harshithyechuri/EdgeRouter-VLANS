# EdgeOS VLAN Creation
EdgeOS CLI Commands For A Creating VLAN's

***Switch to Config mode***
```
configure
```


***VLAN Creation***
(In my case I am creating vlans for 'eth2')
```
set interfaces ethernet eth2 vif 100 address 192.168.100.1/24
set interfaces ethernet eth2 vif 100 description '*VLAN-Name*'

commit
```

***Create DHCP Server for the VLAN***
```
set service dhcp-server disabled false
set service dhcp-server shared-network-name Vlan-100
set service dhcp-server shared-network-name Vlan-100 authoritative enable
set service dhcp-server shared-network-name Vlan-100 subnet 192.168.100.0/24 default-router 192.168.100.1
set service dhcp-server shared-network-name Vlan-100 subnet 192.168.100.0/24 dns-server 8.8.8.8
set service dhcp-server shared-network-name Vlan-100 subnet 192.168.100.0/24 lease 86400
set service dhcp-server shared-network-name Vlan-100 subnet 192.168.100.0/24 start 192.168.100.10 stop 192.168.100.250

commit
```

***Set up FIREWALL for VLAN***
(If Needed)

*Create Network Group"*
```
set firewall group network-group LAN_NETWORKS description 'Private Network Group'
set firewall group network-group LAN_NETWORKS network 192.168.0.0/16
set firewall group network-group LAN_NETWORKS network 172.16.0.0/12
set firewall group network-group LAN_NETWORKS network 10.0.0.0/8
commit
```
*Set up rules for GUEST VLAN*
```
set firewall name VLAN100_IN
set firewall name VLAN100_IN description 'Firewall for Vlan-100' 
set firewall name VLAN100_IN default-action accept
set firewall name VLAN100_IN rule 10 action accept
set firewall name VLAN100_IN rule 10 description 'Allow'
set firewall name VLAN100_IN rule 10 protocol all
set firewall name VLAN100_IN rule 10 state established enable
set firewall name VLAN100_IN rule 10 state related enable
set firewall name VLAN100_IN rule 20 action drop
set firewall name VLAN100_IN rule 20 description 'Drop'
set firewall name VLAN100_IN rule 20 destination group network-group LAN_NETWORKS
set firewall name VLAN100_IN rule 20 protocol all

set firewall name VLAN100_LOACL
set firewall name VLAN100_LOACL description 'Firewall for Vlan-100'
set firewall name VLAN100_LOACL default-action drop
set firewall name VLAN100_LOACL rule 10 action accept
set firewall name VLAN100_LOACL rule 10 description 'Allow DNS'
set firewall name VLAN100_LOACL rule 10 log disable
set firewall name VLAN100_LOACL rule 10 protocol tcp_udp
set firewall name VLAN100_LOACL rule 10 destination port 53
set firewall name VLAN100_LOACL rule 20 action accept
set firewall name VLAN100_LOACL rule 20 description 'Allow DHCP'
set firewall name VLAN100_LOACL rule 20 log disable
set firewall name VLAN100_LOACL rule 20 protocol udp
set firewall name VLAN100_LOACL rule 20 destination port 67

commit
```

*Assign the VLAN interfaces*
```
set interfaces ethernet eth2 vif 100 firewall in name VLAN100_IN
set interfaces ethernet eth2 vif 100 firewall local name VLAN100_LOCAL

commit
save
```
