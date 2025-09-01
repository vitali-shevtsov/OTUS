
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
