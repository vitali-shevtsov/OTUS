## Топология и адресация для настройки VxLAN EVPN Multihoming

<img width="648" height="422" alt="lab7_topo" src="https://github.com/user-attachments/assets/ddd3a343-9e76-46ec-a5f7-fe3e04372494" />

<img width="648" height="422" alt="lab7_topo" src="https://github.com/user-attachments/assets/5418ed4c-928f-4fee-978b-6012ec60f28e" />



Оборудование - Huawei CE12800.

SRV1 и SRV2 находятся в разных сетях, и чтобы их соединить настроим distributed VXLAN gateways. 

Leaf1 и Leaf3 находятся в AS200, Leaf2 - AS300, Spine - AS100. 
Leaf1, Leaf2, Leaf3 и Spine используют AS100 для BGP EVPN.

### План работы

1. Настроить eBGP между Spine и Leaf'ами.
2. Настроить Leaf1 и Leaf3 как Root bridge; Bridge ID на обоих устройствах одинаковый.
3. Настоить M-LAG между Leaf1 и Leaf3.
4. Настроить клиентские порты на Leaf'ах. 
5. Настроить EVPN в качестве VXLAN control plane.
6. Настроить Spine как BGP EVPN-пир для Leaf'ов.
7. Настроить Leaf'ы как BGP EVPN-пиры - RR-клиенты.
8. Настроить VPN- и EVPN-инстансы на Leaf'ах.
9. Настроить Leaf'ы как Layer 3 VXLAN gateways.
10. Настроить BGP между Spine и Leaf'ами для анонсов IRB-маршрутов.
 

### Начальные данные
 - VLAN'ы для клиентов 10 и 20
 - BD соответственно 10 и 20
 - VNI 10 и 20
 - VNI 5010 для VPN instance
 - RDs и RTs для EVPN и VPN-инстансов в таблице ниже
   
<img width="372" height="383" alt="rd rt lab7" src="https://github.com/user-attachments/assets/e05bbfcc-25bd-496e-a397-7e3e505a4275" />


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

Чтобы Leaf1 и Leaf3 могли обмениваться маршрутами в AS200 выполняем на каждом соответствующую команду: 

```
peer 10.1.0.1 allow-as-loop
```
```
peer 10.1.3.1 allow-as-loop
```

2. Настройка Leaf1 и Leaf3 как Root bridge; Bridge ID на обоих устройствах одинаковый.

```
stp root primary
stp bridge-address 0039-0039-0039
interface eth-trunk 10
stp edged-port enable
```

3. Настойка M-LAG между Leaf1 и Leaf3.

Создать Eth-Trunk в режиме LACP на Leaf1 и добавить в него физические интерфейсы. Зеркально настроить Leaf3.
```
interface eth-trunk 1
mode lacp-static
trunkport GE1/0/8
trunkport GE1/0/9
```
```
interface eth-trunk 10
mode lacp-static
trunkport GE1/0/3
```

Настройка Dynamic Fabric Service (DFS) на Leaf1 и Leaf3:
```
dfs-group 1
source ip 2.2.2.2
```
```
dfs-group 1
source ip 4.4.4.4
```
Настроить peer link между Leaf1 и Leaf3:
```
interface eth-trunk 1
undo stp enable
peer-link 1
```
Связать eth-trunk 10 с DFS-группой на Leaf1 и Leaf3:

 ```
interface eth-trunk 10
dfs-group 1 m-lag 1
 ```

4. Настройка клиентского порта на примере Leaf1.
```
bridge-domain 10
interface eth-trunk 10.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
```
5. Включение EVPN в качестве VXLAN control plane

```
evpn-overlay enable
```

6. Настройка BGP EVPN на Spine

```
bgp 100 instance evpn1
 peer 2.2.2.2 as-number 100
 peer 2.2.2.2 connect-interface LoopBack0
 peer 3.3.3.3 as-number 100
 peer 3.3.3.3 connect-interface LoopBack0
 peer 4.4.4.4 as-number 100
 peer 4.4.4.4 connect-interface LoopBack0
 #
 l2vpn-family evpn
  undo policy vpn-target
  peer 2.2.2.2 enable
  peer 2.2.2.2 reflect-client
  peer 3.3.3.3 enable
  peer 3.3.3.3 reflect-client
  peer 4.4.4.4 enable
  peer 4.4.4.4 reflect-client
```

7. Настройка BGP EVPN на Leaf1 (Leaf2, Leaf3 аналогично)
```
bgp 100 instance evpn1
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 connect-interface LoopBack0
 #
 l2vpn-family evpn
  peer 1.1.1.1 enable
```

8. Настройка VPN- и EVPN-инстансов на Leaf'ах (на примере Leaf1).
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

  9. Настройка Leaf'ов как Layer 3 VXLAN gateways (на примере Leaf1).
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

