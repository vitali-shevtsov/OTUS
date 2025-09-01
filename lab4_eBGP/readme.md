
<img width="762" height="448" alt="Topology_eBGP" src="https://github.com/user-attachments/assets/64c099a2-fbab-4757-aca2-d889b3bb3ea5" />

![ip plan](https://github.com/user-attachments/assets/d6f61070-4808-4736-90b7-e781034cf0e6)

## 1) Настройка маршрутизаторов, включая задание IPv4-адресов на интерфейсах. Базовая настройка eBGP
Оба Spine настраиваем в AS6500, каждый Leaf в своей отдельной AS

### Spine1:
```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet1/0
 ip address 10.0.1.1 255.255.255.252
!
interface GigabitEthernet2/0
 ip address 10.0.2.1 255.255.255.252
!
interface GigabitEthernet3/0
 ip address 10.0.3.1 255.255.255.252
!
router bgp 65000
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.0.1.2 remote-as 65100
 neighbor 10.0.1.2 password clos
 neighbor 10.0.2.2 remote-as 65200
 neighbor 10.0.2.2 password clos
 neighbor 10.0.3.2 remote-as 65300
 neighbor 10.0.3.2 password clos
 no auto-summary
```
### Spine2:
```
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface GigabitEthernet1/0
 ip address 10.0.6.1 255.255.255.252
!
interface GigabitEthernet2/0
 ip address 10.0.4.1 255.255.255.252
!
interface GigabitEthernet5/0
 ip address 10.0.5.1 255.255.255.252
 negotiation auto
!
router bgp 65000
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.0.4.2 remote-as 65100
 neighbor 10.0.4.2 password clos
 neighbor 10.0.5.2 remote-as 65300
 neighbor 10.0.5.2 password clos
 neighbor 10.0.6.2 remote-as 65200
 neighbor 10.0.6.2 password clos
 no auto-summary
```
Для Leaf настраиваем peer-group и анонсируем клиентские сети через команду network:

### Leaf1
```
interface Loopback0
 ip address 11.11.11.11 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.1.254 255.255.255.0
!
interface GigabitEthernet1/0
 ip address 10.0.1.2 255.255.255.252
!
interface GigabitEthernet2/0
 ip address 10.0.4.2 255.255.255.252
!
router bgp 65100
 no synchronization
 bgp log-neighbor-changes
 network 192.168.1.0
 neighbor SPINEs peer-group
 neighbor SPINEs password clos
 neighbor 10.0.1.1 remote-as 65000
 neighbor 10.0.1.1 peer-group SPINEs
 neighbor 10.0.4.1 remote-as 65000
 neighbor 10.0.4.1 peer-group SPINEs
 no auto-summary
```
 ### Leaf2
```
interface Loopback0
 ip address 22.22.22.22 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.2.254 255.255.255.0
!
interface GigabitEthernet1/0
 ip address 10.0.6.2 255.255.255.252
!
interface GigabitEthernet2/0
 ip address 10.0.2.2 255.255.255.252
!
router bgp 65200
 no synchronization
 bgp log-neighbor-changes
 network 192.168.2.0
 neighbor SPINEs peer-group
 neighbor SPINEs password clos
 neighbor 10.0.2.1 remote-as 65000
 neighbor 10.0.2.1 peer-group SPINEs
 neighbor 10.0.6.1 remote-as 65000
 neighbor 10.0.6.1 peer-group SPINEs
 no auto-summary
```
 ### Leaf3
```
interface Loopback0
 ip address 33.33.33.33 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.3.254 255.255.255.0
!
interface GigabitEthernet3/0
 ip address 10.0.3.2 255.255.255.252
!
interface GigabitEthernet5/0
 ip address 10.0.5.2 255.255.255.252
!
router bgp 65300
 no synchronization
 bgp log-neighbor-changes
 network 192.168.3.0
 neighbor SPINEs peer-group
 neighbor SPINEs password clos
 neighbor 10.0.3.1 remote-as 65000
 neighbor 10.0.3.1 peer-group SPINEs
 neighbor 10.0.5.1 remote-as 65000
 neighbor 10.0.5.1 peer-group SPINEs
 no auto-summary
```
## 2) Проверка
```
Spine1#show ip bgp summary
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.2        4 65100      57      57        4    0    0 00:51:45        1
10.0.2.2        4 65200      57      57        4    0    0 00:51:49        1
10.0.3.2        4 65300      57      57        4    0    0 00:51:47        1
```
```
Spine1#show ip bgp
   Network          Next Hop            Metric LocPrf Weight Path
*> 192.168.1.0      10.0.1.2                 0             0 65100 i
*> 192.168.2.0      10.0.2.2                 0             0 65200 i
*> 192.168.3.0      10.0.3.2                 0             0 65300 i
```
