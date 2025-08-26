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
## 2) Проверка настроек 
### SuperSpine:
```
SuperSpine#show isis database

IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
SuperSpine.00-00    * 0x00000007   0x51DB        754               0/0/0
Spine11.00-00         0x00000004   0x635C        584               0/0/0
Spine11.01-00         0x00000002   0x6DC5        606               0/0/0
Spine12.00-00         0x00000004   0xD69D        681               0/0/0
Spine12.03-00         0x00000002   0x8567        567               0/0/0
Spine21.00-00         0x00000004   0x4896        538               0/0/0
Spine21.03-00         0x00000002   0xBD2F        667               0/0/0
```
```
SuperSpine#show isis neighbors

System Id      Type Interface IP Address      State Holdtime Circuit Id
Spine11        L2   Fa0/0     10.0.2.2        UP    9        Spine11.01
Spine21        L2   Fa3/0     10.1.0.2        UP    9        Spine21.03
Spine12        L2   Gi5/0     10.0.3.2        UP    9        Spine12.03
```
```
SuperSpine#show ip route isis
     220.220.220.0/32 is subnetted, 1 subnets
i L2    220.220.220.220 [115/20] via 10.1.0.2, FastEthernet3/0
     210.210.210.0/32 is subnetted, 1 subnets
i L2    210.210.210.210 [115/20] via 10.1.0.2, FastEthernet3/0
     21.0.0.0/32 is subnetted, 1 subnets
i L2    21.21.21.21 [115/10] via 10.1.0.2, FastEthernet3/0
     110.0.0.0/32 is subnetted, 1 subnets
i L2    110.110.110.110 [115/20] via 10.0.3.2, GigabitEthernet5/0
                        [115/20] via 10.0.2.2, FastEthernet0/0
i L2 192.168.4.0/24 [115/20] via 10.1.0.2, FastEthernet3/0
     10.0.0.0/30 is subnetted, 9 subnets
i L2    10.1.2.0 [115/20] via 10.1.0.2, FastEthernet3/0
i L2    10.1.1.0 [115/20] via 10.1.0.2, FastEthernet3/0
i L2    10.0.0.0 [115/20] via 10.0.2.2, FastEthernet0/0
i L2    10.0.1.0 [115/20] via 10.0.2.2, FastEthernet0/0
i L2    10.0.4.0 [115/20] via 10.0.3.2, GigabitEthernet5/0
i L2    10.0.5.0 [115/20] via 10.0.3.2, GigabitEthernet5/0
     11.0.0.0/32 is subnetted, 1 subnets
i L2    11.11.11.11 [115/10] via 10.0.2.2, FastEthernet0/0
     12.0.0.0/32 is subnetted, 1 subnets
i L2    12.12.12.12 [115/10] via 10.0.3.2, GigabitEthernet5/0
i L2 192.168.1.0/24 [115/20] via 10.0.3.2, GigabitEthernet5/0
                    [115/20] via 10.0.2.2, FastEthernet0/0
i L2 192.168.2.0/24 [115/20] via 10.0.3.2, GigabitEthernet5/0
                    [115/20] via 10.0.2.2, FastEthernet0/0
     120.0.0.0/32 is subnetted, 1 subnets
i L2    120.120.120.120 [115/20] via 10.0.3.2, GigabitEthernet5/0
                        [115/20] via 10.0.2.2, FastEthernet0/0
i L2 192.168.3.0/24 [115/20] via 10.1.0.2, FastEthernet3/0

```
Все сети на SuperSpine прилетели, в том числе клиентские.

### Spine11:
```
Spine11#show isis database

IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
Spine11.00-00       * 0x00000005   0x6EFA        945               1/0/0
Spine12.00-00         0x00000005   0x1B1A        1044              1/0/0
Leaf11.00-00          0x00000003   0x1199        420               0/0/0
Leaf11.01-00          0x00000003   0xB3D5        1040              0/0/0
Leaf11.02-00          0x00000003   0xAEB7        1086              0/0/0
Leaf12.00-00          0x00000004   0xB23C        1080              0/0/0
Leaf12.01-00          0x00000002   0x160E        430               0/0/0
Leaf12.02-00          0x00000003   0x0B39        1131              0/0/0
IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
SuperSpine.00-00      0x00000008   0x4FDC        1186              0/0/0
Spine11.00-00       * 0x00000004   0x635C        321               0/0/0
Spine11.01-00       * 0x00000003   0x6BC6        1164              0/0/0
Spine12.00-00         0x00000004   0xD69D        414               0/0/0
Spine12.03-00         0x00000003   0x8368        1076              0/0/0
Spine21.00-00         0x00000005   0x4697        969               0/0/0
Spine21.03-00         0x00000002   0xBD2F        400               0/0/0
```
Видим, что на Spine две Link State базы, поскольку работает в роли IS-IS Level-1/2.

```
Spine11#show isis neighbor

System Id      Type Interface IP Address      State Holdtime Circuit Id
Leaf11         L1   Gi1/0     10.0.0.1        UP    7        Leaf11.01
SuperSpine     L2   Fa0/0     10.0.2.1        UP    23       Spine11.01
Leaf12         L1   Gi2/0     10.0.1.1        UP    9        Leaf12.02
```

```
Spine11#show ip route isis
     220.220.220.0/32 is subnetted, 1 subnets
i L2    220.220.220.220 [115/30] via 10.0.2.1, FastEthernet0/0
     210.210.210.0/32 is subnetted, 1 subnets
i L2    210.210.210.210 [115/30] via 10.0.2.1, FastEthernet0/0
     1.0.0.0/32 is subnetted, 1 subnets
i L2    1.1.1.1 [115/10] via 10.0.2.1, FastEthernet0/0
     21.0.0.0/32 is subnetted, 1 subnets
i L2    21.21.21.21 [115/20] via 10.0.2.1, FastEthernet0/0
     110.0.0.0/32 is subnetted, 1 subnets
i L1    110.110.110.110 [115/10] via 10.0.0.1, GigabitEthernet1/0
i L2 192.168.4.0/24 [115/30] via 10.0.2.1, FastEthernet0/0
     10.0.0.0/30 is subnetted, 9 subnets
i L2    10.1.2.0 [115/30] via 10.0.2.1, FastEthernet0/0
i L1    10.0.3.0 [115/30] via 10.0.1.1, GigabitEthernet2/0
                 [115/30] via 10.0.0.1, GigabitEthernet1/0
i L2    10.1.1.0 [115/30] via 10.0.2.1, FastEthernet0/0
i L2    10.1.0.0 [115/20] via 10.0.2.1, FastEthernet0/0
i L1    10.0.4.0 [115/20] via 10.0.0.1, GigabitEthernet1/0
i L1    10.0.5.0 [115/20] via 10.0.1.1, GigabitEthernet2/0
     12.0.0.0/32 is subnetted, 1 subnets
i L1    12.12.12.12 [115/20] via 10.0.1.1, GigabitEthernet2/0
                    [115/20] via 10.0.0.1, GigabitEthernet1/0
i L1 192.168.1.0/24 [115/10] via 10.0.0.1, GigabitEthernet1/0
i L1 192.168.2.0/24 [115/10] via 10.0.1.1, GigabitEthernet2/0
     120.0.0.0/32 is subnetted, 1 subnets
i L1    120.120.120.120 [115/10] via 10.0.1.1, GigabitEthernet2/0
i L2 192.168.3.0/24 [115/30] via 10.0.2.1, FastEthernet0/0
```
Все сети на месте.

### Leaf22:
```
Leaf22#show isis database

IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
Spine21.00-00         0x00000005   0x8529        828               1/0/0
Leaf21.00-00          0x00000004   0x1F9C        777               0/0/0
Leaf21.01-00          0x00000003   0xB36F        771               0/0/0
Leaf22.00-00        * 0x00000004   0x64BF        917               0/0/0
Leaf22.01-00        * 0x00000003   0x12CC        668               0/0/0
```
```
Leaf22#show isis neighbors

System Id      Type Interface IP Address      State Holdtime Circuit Id
Spine21        L1   Gi2/0     10.1.2.2        UP    22       Leaf22.01
```
```
Leaf22#show ip route isis
     210.210.210.0/32 is subnetted, 1 subnets
i L1    210.210.210.210 [115/20] via 10.1.2.2, GigabitEthernet2/0
     21.0.0.0/32 is subnetted, 1 subnets
i L1    21.21.21.21 [115/10] via 10.1.2.2, GigabitEthernet2/0
     10.0.0.0/30 is subnetted, 3 subnets
i L1    10.1.1.0 [115/20] via 10.1.2.2, GigabitEthernet2/0
i L1    10.1.0.0 [115/20] via 10.1.2.2, GigabitEthernet2/0
i L1 192.168.3.0/24 [115/20] via 10.1.2.2, GigabitEthernet2/0
i*L1 0.0.0.0/0 [115/10] via 10.1.2.2, GigabitEthernet2/0
```
Leaf получил сети от соседа по IS-IS Level-1 и имеет 0/0 через Spine. 
