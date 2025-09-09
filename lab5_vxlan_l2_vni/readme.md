## Топология для настройки VxLAN EVPN для L2

![lab5_vxlan_l2_vni](https://github.com/user-attachments/assets/caa8cd95-3a25-4423-a8b0-1d506d3b914c)

## IP-план

![ipplan_lab5](https://github.com/user-attachments/assets/bbc30c01-93b3-4703-82ba-b9978278c284)

### Исходные данные
- необходимо обеспечить L2-связность между двумя ВМ VPC и VPC4 (первая находится во VLAN 30, вторая - VLAN 20);
- в качестве IGP в опорной сети - OSPF;
- Bridge domain ID - 20;
- VNI ID - 5020;
- значение RD для EVPN inst - 12:1 и 31:2;
- значение RT для EVPN inst - 2:2;
- сетевое оборудование - Huawei CE12800.

## Настройка

### 1. Настройка адресации и протокола маршрутизации для underlay-сети
### На примере Spine1 (аналогично для остальных сетевых хостов):
```
#
interface GE1/0/0
 ip address 10.1.0.1 255.255.255.252
#
interface GE1/0/1
 ip address 10.1.1.1 255.255.255.252
#
interface GE1/0/2
 ip address 10.1.2.1 255.255.255.252
#
ospf 1
 area 0.0.0.0
  network 1.1.1.1 0.0.0.0
  network 10.1.0.0 0.0.0.3
  network 10.1.1.0 0.0.0.3
  network 10.1.2.0 0.0.0.3
#
```
### После настройки на всех хостах проверяем связность, пингуем loopback-интерфейсы: 
```
<Spine1>ping 11.11.11.11
  PING 11.11.11.11: 56  data bytes, press CTRL_C to break
    Reply from 11.11.11.11: bytes=56 Sequence=1 ttl=255 time=4 ms
    Reply from 11.11.11.11: bytes=56 Sequence=2 ttl=255 time=4 ms
    Reply from 11.11.11.11: bytes=56 Sequence=3 ttl=255 time=3 ms
    Reply from 11.11.11.11: bytes=56 Sequence=4 ttl=255 time=3 ms
    Reply from 11.11.11.11: bytes=56 Sequence=5 ttl=255 time=3 ms
   
<Spine1>ping 33.33.33.33
  PING 33.33.33.33: 56  data bytes, press CTRL_C to break
    Reply from 33.33.33.33: bytes=56 Sequence=1 ttl=255 time=10 ms
    Reply from 33.33.33.33: bytes=56 Sequence=2 ttl=255 time=4 ms
    Reply from 33.33.33.33: bytes=56 Sequence=3 ttl=255 time=4 ms
    Reply from 33.33.33.33: bytes=56 Sequence=4 ttl=255 time=6 ms
    Reply from 33.33.33.33: bytes=56 Sequence=5 ttl=255 time=3 ms

<Spine1>ping 2.2.2.2
  PING 2.2.2.2: 56  data bytes, press CTRL_C to break
    Reply from 2.2.2.2: bytes=56 Sequence=1 ttl=254 time=22 ms
    Reply from 2.2.2.2: bytes=56 Sequence=2 ttl=254 time=6 ms
    Reply from 2.2.2.2: bytes=56 Sequence=3 ttl=254 time=6 ms
    Reply from 2.2.2.2: bytes=56 Sequence=4 ttl=254 time=6 ms
    Reply from 2.2.2.2: bytes=56 Sequence=5 ttl=254 time=5 ms
```
### 2. Настраиваем пользовательские интерфейсы на Leaf1 и Leaf3 
### Leaf1
```
#
bridge-domain 20
#
interface GE1/0/3.1 mode l2
 encapsulation dot1q vid 30
 bridge-domain 20
#
```
### Leaf3
```
#
bridge-domain 20
#
interface GE1/0/3.1 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20
#
```
### 3. Включаем EVPN в качестве control plane для VxLAN на всех сетевых хостах
### На примере Leaf1
```
#
evpn-overlay enable
#
```
### 4. Настраиваем BGP EVPN соседство. Leaf1 и Leaf3 настраиваем как BGP EVPN-пиры
### На примере Spine1
```
#
bgp 100
 peer 11.11.11.11 as-number 100
 peer 11.11.11.11 connect-interface LoopBack0
 peer 33.33.33.33 as-number 100
 peer 33.33.33.33 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 11.11.11.11 enable
  peer 33.33.33.33 enable
 #
 l2vpn-family evpn
  undo policy vpn-target
  peer 11.11.11.11 enable
  peer 11.11.11.11 reflect-client
  peer 33.33.33.33 enable
  peer 33.33.33.33 reflect-client
#
```
### На примере Leaf1

```
#
bgp 100
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 1.1.1.1 enable
 #
 l2vpn-family evpn
  policy vpn-target
  peer 1.1.1.1 enable
#
```
### 5. Настраиваем EVPN инстансы на Leaf1 и Leaf3
### Leaf1:
```
#
bridge-domain 20
 vxlan vni 5020
 evpn
  route-distinguisher 12:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
```
### Leaf3:
```
#
bridge-domain 20
 vxlan vni 5020
 evpn
  route-distinguisher 31:2
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
```
### 6. Настраиваем туннельные интерфейсы NVE на Leaf1 и Leaf3

### Leaf1:
```
#
interface Nve1
 source 11.11.11.11
 vni 5020 head-end peer-list protocol bgp
#
```
### Leaf3:
```
#
interface Nve1
 source 33.33.33.33
 vni 5020 head-end peer-list protocol bgp
#
```
