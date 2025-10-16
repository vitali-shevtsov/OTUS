## НАСТРОЙКА L3-СВЯЗНОСТИ МЕЖДУ ЦОДАМИ ЧЕРЕЗ VxLAN И BGP EVPN

 ### Исходные данные

Имеется два ЦОДа, IP-фабрики каждого из которых имеют свои автономные системы BGP. Необходимо обеспечить сетевую связность между сегментами как в пределах каждого ЦОДа, так и между сегментами разных ЦОДов; использовать VxLAN и BGP EVPN.

<img width="712" height="411" alt="image" src="https://github.com/user-attachments/assets/c19e1465-18ae-426a-957b-1143986b0729" />

### Структурная схема

Структурная схема масштабируемой IP-фабрики для ЦОДов в классической топологии Clos может иметь примерно следующий вид:

<img width="894" height="456" alt="image" src="https://github.com/user-attachments/assets/360658fe-be42-42a1-be6d-3058ebf22de3" />

### Физическая топология

Для реализации проекта собрана минимальная физическая топология, где каждое устройство выполняет функции VTEP. Все клиенты (SRV-1, SRV-2, SRV-3, SRV-4) находятся в разных подсетях, поэтому нужно обеспечить полную L3-связность между ними:

<img width="854" height="496" alt="image" src="https://github.com/user-attachments/assets/c52e64a9-6b3e-4962-bd83-1a68e9e7d793" />

### Схема VxLAN-туннелей

Логическая связность между сетевыми хостами будет реализована посредством VxLAN-туннелей следующим образом:

![Project topo](https://github.com/user-attachments/assets/c56e9de0-496f-4ce1-81eb-a68f0018b2ef)



### Последовательность настройки


Часть 1

1.	Настройка динамической маршрутизации между VTEP’ами для обеспечения L3-связности между ними
2.	Настройка клиентских портов на VTEP2, VTEP3, VTEP7, VTEP8
3.	Настройка EVPN-инстансов на VTEP2, VTEP3, VTEP7, VTEP8 и их привязка к BD
4.	Настройка VPN-инстансов на каждом VTEP и их привязка к соответствующим интерфейсам VBDIF на VTEP2, VTEP3, VTEP7, VTEP8.
5.	Установление отношений соседства BGP EVPN между VTEP1 и VTEP2, VTEP1 и VTEP3, а также между VTEP6 и VTEP7, VTEP6 и VTEP8
6.	Настройка VTEP1 в качестве RR для VTEP2 и VTEP3. Настройка VTEP6 в качестве RR для VTEP7 и VTEP8. Это позволит устанавливать соседства BGP EVPN между VTEP2 и VTEP3, и между VTEP7 и VTEP8
7.	Настройка IP-адресов назначения для VxLAN-туннелей на VTEP2, VTEP3, VTEP7 и VTEP8
8.	Настройка distributed VXLAN gateway на VTEP2, VTEP3, VTEP7 и VTEP8
9.	Настройка default route на VTEP1 and VTEP6. Каждый из VTEP’ов обеспечивает связность в пределах своей AS, а также между AS.

Часть 2

1.	Настройка соседства BGP EVPN между VTEP1 и VTEP6
2.	Настройка регенерации EVPN-маршрутов на VTEP1 и VTEP6

### IP-план

 ![project ip plan](https://github.com/user-attachments/assets/8f05db71-d565-4ea9-a806-a9a24f6bc39b)

### Таблица инстансов BGP EVPN

![project evpn inst~](https://github.com/user-attachments/assets/e9d2fb0d-0df9-4516-88b7-14460593bc54)


### Таблица VPN-инстансов

![project vpn inst](https://github.com/user-attachments/assets/af62acbb-e6a3-4ef0-9c2e-e8f50f28dca7)

## Настройка

### 1. Настройка адресации и протокола маршрутизации для underlay-сети
На примере VTEP1 (аналогично для остальных сетевых хостов):

```
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.0.1.2 255.255.255.252
#
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip address 10.0.2.1 255.255.255.252
#
interface GE1/0/3
 undo portswitch
 undo shutdown
 ip address 10.0.6.1 255.255.255.0
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
#
ospf 1
 area 0.0.0.0
  network 1.1.1.1 0.0.0.0
  network 10.0.1.0 0.0.0.3
  network 10.0.2.0 0.0.0.3
  network 10.0.6.0 0.0.0.255
```

### 2. Настройка клиентских портов на VTEP2, VTEP3, VTEP7, VTEP8

На примере VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):
```
bridge-domain 10
#
interface GE1/0/2
 port link-type trunk
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10

```

### 3.	Настройка EVPN на VTEP2, VTEP3, VTEP7, VTEP8 и их привязка к BD

На примере VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):
```
bridge-domain 10
 vxlan vni 10
 evpn
  route-distinguisher 2:10
  vpn-target 10:1 export-extcommunity
  vpn-target 1:100 export-extcommunity
  vpn-target 10:1 import-extcommunity
```

### 4.	Настройка VPN-инстансов на каждом VTEP и их привязка к соответствующим интерфейсам VBDIF на VTEP2, VTEP3, VTEP7, VTEP8.
Настройка на VTEP1 (аналогично для VTEP6):

```
ip vpn-instance vpn1
 ipv4-family
  route-distinguisher 1:100
  vpn-target 1:100 export-extcommunity evpn
  vpn-target 10:100 export-extcommunity evpn
  vpn-target 1:100 import-extcommunity evpn
  vpn-target 10:100 import-extcommunity evpn
 vxlan vni 100
```

На примере VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):

```
ip vpn-instance vpn1
 ipv4-family
  route-distinguisher 2:100
  vpn-target 1:100 export-extcommunity evpn
  vpn-target 1:100 import-extcommunity evpn
 vxlan vni 100
#
interface Vbdif10
 ip binding vpn-instance vpn1
```
### 5. Установление отношений соседства BGP EVPN между VTEP1 и VTEP2, VTEP1 и VTEP3, а также между VTEP6 и VTEP7, VTEP6 и VTEP8
Настройка на VTEP1 (аналогично для VTEP6):
```
bgp 100
 router-id 1.1.1.1
 peer 2.2.2.2 as-number 100
 peer 2.2.2.2 connect-interface LoopBack0
 peer 3.3.3.3 as-number 100
 peer 3.3.3.3 connect-interface LoopBack0
 #
 l2vpn-family evpn
  peer 2.2.2.2 enable
  peer 2.2.2.2 advertise irb
  peer 3.3.3.3 enable
  peer 3.3.3.3 advertise irb
 #
 ipv4-family vpn-instance vpn1
  import-route direct
  advertise l2vpn evpn
```
Настройка на VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):
```
bgp 100
 router-id 2.2.2.2
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 connect-interface LoopBack0
 #
l2vpn-family evpn
  peer 1.1.1.1 enable
  peer 1.1.1.1 advertise irb
#
 ipv4-family vpn-instance vpn1
  import-route direct
  advertise l2vpn evpn
 #
 
```
