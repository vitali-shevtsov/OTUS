## Топология и адресация для настройки VxLAN EVPN Multihoming

<img width="648" height="422" alt="lab7_topo" src="https://github.com/user-attachments/assets/ddd3a343-9e76-46ec-a5f7-fe3e04372494" />

Оборудование - Huawei CE12800.

SRV1 и SRV2 находятся в разных сетях, и чтобы их соединить настроим distributed VXLAN gateways. 

Leaf1 и Leaf3 находятся в AS200, Leaf2 - AS300, Spine - AS100. 
Leaf1, Leaf2, Leaf3 и Spine используют AS100 для BGP EVPN.

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
  peer 2.2.2.2 reflect-client
  peer 3.3.3.3 enable
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

8. Настройка анонсов IRB-маршрутов (на примере Spine) между Spine и Leaf'ами.
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

 ```
<Leaf2>disp ip routing-table vpn-instance vpn1
 ```
 ``` 
Proto: Protocol        Pre: Preference
Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
------------------------------------------------------------------------------
Routing Table : vpn1
         Destinations : 5        Routes : 5         

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

     100.1.1.10/32  IBGP    255  0             RD  2.2.2.2         VXLAN
      200.1.1.0/24  Direct  0    0             D   200.1.1.1       Vbdif20
      200.1.1.1/32  Direct  0    0             D   127.0.0.1       Vbdif20
    200.1.1.255/32  Direct  0    0             D   127.0.0.1       Vbdif20
255.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
 ``` 

 ### Проверка связности между клиентами

 SRV1:
 
 ```
<client1>disp cur int Vlanif 10
#
interface Vlanif10
 ip address 100.1.1.10 255.255.255.0
 ```
 ```
<client1>disp ip routing-table 
Proto: Protocol        Pre: Preference
Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
------------------------------------------------------------------------------
Routing Table : _public_
         Destinations : 8        Routes : 8         

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

        0.0.0.0/0   Static  60   0             RD  100.1.1.1       Vlanif10
      100.1.1.0/24  Direct  0    0             D   100.1.1.10      Vlanif10
     100.1.1.10/32  Direct  0    0             D   127.0.0.1       Vlanif10
    100.1.1.255/32  Direct  0    0             D   127.0.0.1       Vlanif10
      127.0.0.0/8   Direct  0    0             D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0             D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0             D   127.0.0.1       InLoopBack0
 ```
 ``` 
<client1>ping -c 5 200.1.1.10    
  PING 200.1.1.10: 56  data bytes, press CTRL_C to break
    Reply from 200.1.1.10: bytes=56 Sequence=1 ttl=253 time=11 ms
    Reply from 200.1.1.10: bytes=56 Sequence=2 ttl=253 time=12 ms
    Reply from 200.1.1.10: bytes=56 Sequence=3 ttl=253 time=8 ms
    Reply from 200.1.1.10: bytes=56 Sequence=4 ttl=253 time=10 ms
    Reply from 200.1.1.10: bytes=56 Sequence=5 ttl=253 time=9 ms

  --- 200.1.1.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 8/10/12 ms
 ``` 

