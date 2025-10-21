## НАСТРОЙКА L3-СВЯЗНОСТИ МЕЖДУ ЦОДАМИ ЧЕРЕЗ VxLAN И BGP EVPN

 ### Исходные данные

Имеется два ЦОДа, IP-фабрики каждого из которых имеют свои автономные системы BGP. Необходимо обеспечить сетевую связность между сегментами как в пределах каждого ЦОДа, так и между сегментами разных ЦОДов; использовать VxLAN и BGP EVPN.

<img width="712" height="411" alt="image" src="https://github.com/user-attachments/assets/c19e1465-18ae-426a-957b-1143986b0729" />

### Структурная схема

Структурная схема масштабируемой IP-фабрики для ЦОДов в классической топологии Clos может иметь примерно следующий вид:

<img width="894" height="456" alt="image" src="https://github.com/user-attachments/assets/360658fe-be42-42a1-be6d-3058ebf22de3" />

### Физическая топология

Для реализации проекта собрана минимальная физическая топология (PNETLab на образах Huawei CloudEngine 12800), где каждое устройство выполняет функции VTEP. Все клиенты (SRV-1, SRV-2, SRV-3, SRV-4) находятся в разных подсетях, поэтому нужно обеспечить полную L3-связность между ними:

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
6.	Настройка VTEP1 в качестве RR для VTEP2 и VTEP3. Настройка VTEP6 в качестве RR для VTEP7 и VTEP8. 
7.	Настройка IP-адресов назначения для VxLAN-туннелей
8.	Настройка distributed VXLAN gateway на VTEP2, VTEP3, VTEP7 и VTEP8
9.	Настройка default route на VTEP1 и VTEP6. 

Часть 2

10.	Настройка соседства eBGP EVPN между VTEP1 и VTEP6
11.	Настройка пересылки EVPN-маршрутов на VTEP1 и VTEP6

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

### 6.	Настройка VTEP1 в качестве RR для VTEP2 и VTEP3. Настройка VTEP6 в качестве RR для VTEP7 и VTEP8. 

Настраиваем VTEP1 (и VTEP6)  в качестве RR, чтобы иметь возможность устанавливать соседства BGP EVPN между VTEP2 и VTEP3, и между VTEP7 и VTEP8

```
bgp 100
 l2vpn-family evpn
  undo policy vpn-target
  peer 2.2.2.2 reflect-client
  peer 3.3.3.3 reflect-client
```

### 7.	Настройка IP-адресов назначения для VxLAN-туннелей на VTEP'ах

Настройка на VTEP1 (аналогично для VTEP6):

```
interface Nve1
 source 1.1.1.1
```

Настройка на VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):

```
interface Nve1
 source 2.2.2.2
 vni 10 head-end peer-list protocol bgp
```

### 8.	Настройка distributed VXLAN gateway на VTEP2, VTEP3, VTEP7 и VTEP8

Настройка на VTEP2 (аналогично для VTEP3, VTEP7, VTEP8):

```
interface Vbdif10
 ip address 192.168.10.1 255.255.255.0
 arp distribute-gateway enable
 mac-address 0000-5e00-2222
 arp collect host enable
```

### 9. Настройка default route на VTEP1 и VTEP6. 

Каждый из VTEP’ов обеспечивает связность в пределах своей AS, а также между AS.

VTEP1:
```
ip route-static vpn-instance vpn1 0.0.0.0 0.0.0.0 NULL0
bgp 100
 ipv4-family vpn-instance vpn1
  import-route static
  default-route imported
  import-route direct
```

VTEP6:
```
ip route-static vpn-instance vpn1 0.0.0.0 0.0.0.0 NULL0
bgp 200
 ipv4-family vpn-instance vpn1
  default-route imported
  import-route direct
  import-route static
```

Проверим VXLAN-based связность в пределах каждого ЦОДа:
```
<VTEP2>disp vxlan tunnel 
Number of vxlan tunnel : 2
Tunnel ID   Source                Destination           State  Type     Uptime
-----------------------------------------------------------------------------------
4026531841  2.2.2.2               3.3.3.3               up     dynamic  0238h10m  
4026531842  2.2.2.2               1.1.1.1               up     dynamic  0238h08m  
```
```
<SRV-1> disp ip int br
Interface                   IP Address/Mask    Physical Protocol VPN           
MEth0/0/0                   unassigned         up       down     --            
NULL0                       unassigned         up       up(s)    --            
Vlanif10                    192.168.10.10/24   up       up       --            

<SRV-1>ping 192.168.20.10
  PING 192.168.20.10: 56  data bytes, press CTRL_C to break
    Reply from 192.168.20.10: bytes=56 Sequence=1 ttl=253 time=38 ms
    Reply from 192.168.20.10: bytes=56 Sequence=2 ttl=253 time=7 ms
    Reply from 192.168.20.10: bytes=56 Sequence=3 ttl=253 time=9 ms
    Reply from 192.168.20.10: bytes=56 Sequence=4 ttl=253 time=8 ms
    Reply from 192.168.20.10: bytes=56 Sequence=5 ttl=253 time=8 ms

  --- 192.168.20.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 7/14/38 ms
```
```
<VTEP-7>disp vxlan tunnel 
Number of vxlan tunnel : 2
Tunnel ID   Source                Destination           State  Type     Uptime
-----------------------------------------------------------------------------------
4026531841  7.7.7.7               8.8.8.8               up     dynamic  0238h02m  
4026531842  7.7.7.7               6.6.6.6               up     dynamic  0238h00m  
```

```
<SRV-3>disp ip int br
Interface                   IP Address/Mask    Physical Protocol VPN           
MEth0/0/0                   unassigned         up       down     --            
NULL0                       unassigned         up       up(s)    --            
Vlanif30                    192.168.30.10/24   up       up       --            
<SRV-3>ping 192.168.40.10
  PING 192.168.40.10: 56  data bytes, press CTRL_C to break
    Reply from 192.168.40.10: bytes=56 Sequence=1 ttl=253 time=44 ms
    Reply from 192.168.40.10: bytes=56 Sequence=2 ttl=253 time=8 ms
    Reply from 192.168.40.10: bytes=56 Sequence=3 ttl=253 time=10 ms
    Reply from 192.168.40.10: bytes=56 Sequence=4 ttl=253 time=10 ms
    Reply from 192.168.40.10: bytes=56 Sequence=5 ttl=253 time=10 ms

  --- 192.168.40.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 8/16/44 ms
```

### 10.	Настройка соседства eBGP EVPN между VTEP1 и VTEP6

Настройка VTEP1:
```
bgp 100
 peer 6.6.6.6 as-number 200
 peer 6.6.6.6 ebgp-max-hop 255
 peer 6.6.6.6 connect-interface LoopBack0
 l2vpn-family evpn
  peer 6.6.6.6 enable
```

Настройка VTEP6:
```
bgp 200
 peer 1.1.1.1 as-number 100
 peer 1.1.1.1 ebgp-max-hop 255
 peer 1.1.1.1 connect-interface LoopBack0
 l2vpn-family evpn
  peer 1.1.1.1 enable
```

### 11.	Настройка пересылки EVPN-маршрутов на VTEP1 и VTEP6

Настройка VTEP1:
```
bgp 100
 l2vpn-family evpn
  peer 2.2.2.2 import reoriginate
  peer 3.3.3.3 import reoriginate
  peer 6.6.6.6 advertise route-reoriginated evpn ip
```

Настройка VTEP6:
```
bgp 200
 l2vpn-family evpn
  peer 7.7.7.7 import reoriginate
  peer 8.8.8.8 import reoriginate
  peer 1.1.1.1 advertise route-reoriginated evpn ip
```

После выполнения настроек выше IP-префиксы между ЦОДами будут транслироваться через VXLAN-туннель:

```
<VTEP-1>disp ip routing-table  vpn-instance vpn1  protocol bgp 

Destination/Mask    Proto   Pre  Cost        Flags NextHop         Interface

   192.168.10.0/24  IBGP    255  0             RD  2.2.2.2         VXLAN
   192.168.10.1/32  IBGP    255  0             RD  2.2.2.2         VXLAN
  192.168.10.10/32  IBGP    255  0             RD  2.2.2.2         VXLAN
   192.168.20.0/24  IBGP    255  0             RD  3.3.3.3         VXLAN
   192.168.20.1/32  IBGP    255  0             RD  3.3.3.3         VXLAN
  192.168.20.10/32  IBGP    255  0             RD  3.3.3.3         VXLAN
   192.168.30.0/24  EBGP    255  0             RD  6.6.6.6         VXLAN
   192.168.30.1/32  EBGP    255  0             RD  6.6.6.6         VXLAN
   192.168.40.0/24  EBGP    255  0             RD  6.6.6.6         VXLAN
   192.168.40.1/32  EBGP    255  0             RD  6.6.6.6         VXLAN
```

### ПРОВЕРКА

Проверяем IP-связность между SRV:

```
<SRV-1>disp ip int br
Interface                   IP Address/Mask    Physical Protocol VPN           
MEth0/0/0                   unassigned         up       down     --            
NULL0                       unassigned         up       up(s)    --            
Vlanif10                    192.168.10.10/24   up       up       --            

<SRV-1>ping 192.168.20.10
  PING 192.168.20.10: 56  data bytes, press CTRL_C to break
    Reply from 192.168.20.10: bytes=56 Sequence=1 ttl=253 time=22 ms
    Reply from 192.168.20.10: bytes=56 Sequence=2 ttl=253 time=8 ms
    Reply from 192.168.20.10: bytes=56 Sequence=3 ttl=253 time=11 ms
    Reply from 192.168.20.10: bytes=56 Sequence=4 ttl=253 time=9 ms
    Reply from 192.168.20.10: bytes=56 Sequence=5 ttl=253 time=6 ms

  --- 192.168.20.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 6/11/22 ms

<SRV-1>ping 192.168.30.10
  PING 192.168.30.10: 56  data bytes, press CTRL_C to break
    Reply from 192.168.30.10: bytes=56 Sequence=1 ttl=251 time=24 ms
    Reply from 192.168.30.10: bytes=56 Sequence=2 ttl=251 time=11 ms
    Reply from 192.168.30.10: bytes=56 Sequence=3 ttl=251 time=12 ms
    Reply from 192.168.30.10: bytes=56 Sequence=4 ttl=251 time=11 ms
    Reply from 192.168.30.10: bytes=56 Sequence=5 ttl=251 time=10 ms

  --- 192.168.30.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 10/13/24 ms
 
<SRV-1>ping 192.168.40.10
  PING 192.168.40.10: 56  data bytes, press CTRL_C to break
    Reply from 192.168.40.10: bytes=56 Sequence=1 ttl=251 time=18 ms
    Reply from 192.168.40.10: bytes=56 Sequence=2 ttl=251 time=12 ms
    Reply from 192.168.40.10: bytes=56 Sequence=3 ttl=251 time=12 ms
    Reply from 192.168.40.10: bytes=56 Sequence=4 ttl=251 time=13 ms
    Reply from 192.168.40.10: bytes=56 Sequence=5 ttl=251 time=9 ms

  --- 192.168.40.10 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 9/12/18 ms    
```
