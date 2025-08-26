### Топология, лаба запускалась на Cisco IOS:
<img width="877" height="627" alt="Topology" src="https://github.com/user-attachments/assets/cc22e7a7-c68b-4785-80ad-e5591a3d53dd" />

### IPv4-план:
![ip plan](https://github.com/user-attachments/assets/9ba0c2ac-322b-4703-abee-9c978c49bc6f)

## 1) Настройка маршрутизаторов, включая задание IPv4-адресов на интерфейсах. Базовая настройка IS-IS без опциональных параметров

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
