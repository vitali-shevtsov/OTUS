## Топология для настройки VxLAN EVPN для L3

<img width="482" height="420" alt="lab6_topo" src="https://github.com/user-attachments/assets/d85dc045-b792-4899-93e5-6bd10b7089ed" />

Для примера используется минимальная топология с одним Spine и парой Leaf'ов. Huawei CE12800.

SRV1 и SRV2 находятся в разных сетях, и чтобы их соединить настроим distributed VXLAN gateways. 

Leaf1 находится в AS200, Leaf2 - AS300, Spine - AS100. 
Leaf1, Leaf2, Spine используют AS100 для BGP EVPN.

### План работы

1. Настроить eBGP между Spine и Leaf1 и между Spine и Leaf2.
2. Настроить клиентские порты на Leaf'ах.
3. Настроить EVPN в качестве VXLAN control plane.
4. Настроить Spine как BGP EVPN-пир для Leaf'ов.
5. Настроить Leaf'ы как BGP EVPN-пиры - RR-клиенты.
6. Настроить VPN- и EVPN-инстансы на Leaf'ах.
7. Настроить Leaf'ы как Layer 3 VXLAN gateways.
8. Настроить BGP между Spine и Leaf'ами для анонсов IRB-маршрутов.

### Начальные данные
 - VLAN'ы для клиентов 10 и 20
 - BD соответственно 10 и 20
 - VNI 10 и 20
 - VNI 5010 для VPN instance
 - RDs и RTs для EVPN и VPN-инстансов в таблице ниже
   
<img width="372" height="262" alt="rd rt" src="https://github.com/user-attachments/assets/66234e02-3545-45dc-96bb-ecd34e260983" />

## Настройка

1. Настройка EBGP на примере Spine (на Leaf'ах аналогично):
```
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
```
```
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.0.1 255.255.255.252
#
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip address 10.1.2.1 255.255.255.252
```
```
bgp 100
 peer 10.1.0.2 as-number 200
 peer 10.1.2.2 as-number 300
 #
 ipv4-family unicast
  network 1.1.1.1 255.255.255.255
  peer 10.1.0.2 enable
  peer 10.1.2.2 enable
```

2. Настройка клиентского порта на Leaf1 (Leaf2 аналогично).
```
bridge-domain 10
interface GE1/0/3.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
```
3. Включение EVPN в качестве VXLAN control plane

```
evpn-overlay enable
```

4. Настройка BGP EVPN на Spine

```
bgp 100 instance evpn1
 peer 2.2.2.2 as-number 100
 peer 2.2.2.2 connect-interface LoopBack0
 peer 3.3.3.3 as-number 100
 peer 3.3.3.3 connect-interface LoopBack0
 #
 l2vpn-family evpn
  undo policy vpn-target
  peer 2.2.2.2 enable
  peer 2.2.2.2 advertise irb
  peer 2.2.2.2 reflect-client
  peer 3.3.3.3 enable
  peer 3.3.3.3 advertise irb
  peer 3.3.3.3 reflect-client
```

5. Настройка BGP EVPN на Leaf1 (Leaf2 аналогично)
```
bgp 100 instance evpn1
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 connect-interface LoopBack0
 #
 l2vpn-family evpn
  peer 1.1.1.1 enable
```

6. Настройка VPN- и EVPN-инстансов на Leaf'ах (на примере Leaf1).
```
ip vpn-instance vpn1
vxlan vni 5010
 ipv4-family
  route-distinguisher 11:11
  vpn-target 11:1 export-extcommunity evpn
  vpn-target 11:1 import-extcommunity evpn
#
bridge-domain 10
 vxlan vni 10
 evpn
  route-distinguisher 10:1
  vpn-target 10:1 export-extcommunity
  vpn-target 10:1 import-extcommunity
  vpn-target 11:1 export-extcommunity
```

  7. Настройка Leaf'ов как Layer 3 VXLAN gateways (на примере Leaf1).
```
interface Nve1
 source 2.2.2.2
 vni 10 head-end peer-list protocol bgp
```
```
interface Vbdif10
 ip binding vpn-instance vpn1
 ip address 100.1.1.1 255.255.255.0
 vxlan anycast-gateway enable
 arp collect host enable
```

8. Настройка BGP между Spine и Leaf'ами для анонсов IRB-маршрутов (на примере Spine).
```
bgp 100 instance evpn1
 l2vpn-family evpn
  peer 2.2.2.2 advertise irb
  peer 3.3.3.3 advertise irb
 ``` 

### Проверка
 ``` 
<Leaf2>disp vxlan tunnel
 ```
 ``` 
Number of vxlan tunnel : 1
Tunnel ID   Source                Destination           State  Type     Uptime
-----------------------------------------------------------------------------------
4026531842  3.3.3.3               2.2.2.2               up     dynamic  00:05:05  
 ``` 


