### Топология, лаба запускалась на Cisco IOS:
<img width="877" height="627" alt="Topology" src="https://github.com/user-attachments/assets/cc22e7a7-c68b-4785-80ad-e5591a3d53dd" />

### IPv4-план:
![ip plan](https://github.com/user-attachments/assets/9ba0c2ac-322b-4703-abee-9c978c49bc6f)

## 1) Настройка маршрутизаторов, включая задание IPv4-адресов на интерфейсах. Базовая настройка IS-IS без опциональных параметров
SuperSpine настраиваем как level-2, маршрутизаторы Leaf - как level-1, маршрутизаторы Spine остаются по умолчанию в роли level 1-2.
Loopback-интерфейсы и сети клиентских интерфейсов f0/0 анонсируем по IS-IS, но IS-IS на них не запускаем (passive-interface).

### SuperSpine:
```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface FastEthernet0/0
 ip address 10.0.2.1 255.255.255.252
 ip router isis
!
interface FastEthernet3/0
 ip address 10.1.0.1 255.255.255.252
 ip router isis
!
interface GigabitEthernet5/0
 ip address 10.0.3.1 255.255.255.252
 ip router isis
!
router isis
 net 49.0001.0010.0100.1001.00
 is-type level-2-only
 passive-interface Loopback0
```
### Spine11:
```
interface Loopback0
 ip address 11.11.11.11 255.255.255.255
!
interface FastEthernet0/0
 ip address 10.0.2.2 255.255.255.252
 ip router isis
!
interface GigabitEthernet1/0
 ip address 10.0.0.2 255.255.255.252
 ip router isis
!
interface GigabitEthernet2/0
 ip address 10.0.1.2 255.255.255.252
 ip router isis
!
router isis
 net 49.0002.0110.1101.1011.00
 passive-interface Loopback0
```
### Spine12:
```
interface Loopback0
 ip address 12.12.12.12 255.255.255.255
!
interface GigabitEthernet1/0
 ip address 10.0.5.2 255.255.255.252
 ip router isis
!
interface GigabitEthernet2/0
 ip address 10.0.4.2 255.255.255.252
 ip router isis
!
interface GigabitEthernet5/0
 ip address 10.0.3.2 255.255.255.252
 ip router isis
!
router isis
 net 49.0002.0120.1201.2012.00
 passive-interface Loopback0
```
### Spine21:
```
interface Loopback0
 ip address 21.21.21.21 255.255.255.255
!
interface GigabitEthernet1/0
 ip address 10.1.1.2 255.255.255.252
 ip router isis
!
interface GigabitEthernet2/0
 ip address 10.1.2.2 255.255.255.252
 ip router isis
!
interface FastEthernet3/0
 ip address 10.1.0.2 255.255.255.252
 ip router isis
!
router isis
 net 49.0003.0210.2102.1021.00
 passive-interface Loopback0
```
### Leaf11:
```
interface Loopback0
 ip address 110.110.110.110 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.1.254 255.255.255.0
!
interface GigabitEthernet1/0
 ip address 10.0.0.1 255.255.255.252
 ip router isis
!
interface GigabitEthernet2/0
 ip address 10.0.4.1 255.255.255.252
 ip router isis
!
router isis
 net 49.0002.1101.1011.0110.00
 is-type level-1
 passive-interface FastEthernet0/0
 passive-interface Loopback0
```
### Leaf12:
```
interface Loopback0
 ip address 120.120.120.120 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.2.254 255.255.255.0
!
interface GigabitEthernet1/0
 ip address 10.0.5.1 255.255.255.252
 ip router isis
!
interface GigabitEthernet2/0
 ip address 10.0.1.1 255.255.255.252
 ip router isis
!
router isis
 net 49.0002.1201.2012.0120.00
 is-type level-1
 passive-interface FastEthernet0/0
 passive-interface Loopback0
```
### Leaf21:
```
interface Loopback0
 ip address 210.210.210.210 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.3.254 255.255.255.0
!
interface GigabitEthernet1/0
 ip address 10.1.1.1 255.255.255.252
 ip router isis
!
router isis
 net 49.0003.2102.1021.0210.00
 is-type level-1
 passive-interface FastEthernet0/0
 passive-interface Loopback0
```
### Leaf22:
```
interface Loopback0
 ip address 220.220.220.220 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.4.254 255.255.255.0
!
interface GigabitEthernet2/0
 ip address 10.1.2.1 255.255.255.252
 ip router isis
!
router isis
 net 49.0003.2202.2022.0220.00
 is-type level-1
 passive-interface FastEthernet0/0
 passive-interface Loopback0
```
