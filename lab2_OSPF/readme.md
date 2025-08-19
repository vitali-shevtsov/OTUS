
<img width="878" height="447" alt="Topology" src="https://github.com/user-attachments/assets/e5343781-8475-48ac-822b-f7f8626f7776" />


![ip plan](https://github.com/user-attachments/assets/159fdeb7-0d0f-48fb-810d-d1845e5c4b42)

## 1) Настройка интерфейсов согласно IP-плана:
В лабе использовался Cisco IOS.
Loopback-интерфейсы будут использоваться в качестве router-id для процесса OSPF.

### Spine1:

```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255

interface GigabitEthernet1/0
 ip address 10.0.0.1 255.255.255.252
 negotiation auto
end

interface GigabitEthernet2/0
 ip address 10.0.1.1 255.255.255.252
 negotiation auto
end

interface GigabitEthernet3/0
 ip address 10.0.2.1 255.255.255.252
 negotiation auto
end
```

### Spine2:

```
interface Loopback0
 ip address 2.2.2.2 255.255.255.255

interface GigabitEthernet1/0
 ip address 10.0.4.1 255.255.255.252
 negotiation auto
end

interface GigabitEthernet2/0
 ip address 10.0.3.1 255.255.255.252
 negotiation auto
end

interface GigabitEthernet4/0
 ip address 10.0.5.1 255.255.255.252
 negotiation auto
end
```

### Leaf1:

```
interface Loopback0
 ip address 11.11.11.11 255.255.255.255

interface GigabitEthernet1/0
 ip address 10.0.0.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet2/0
 ip address 10.0.3.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet5/0
 ip address 192.168.1.1 255.255.255.0
 negotiation auto
end
```

### Leaf2:

```
interface Loopback0
 ip address 22.22.22.22 255.255.255.255

interface GigabitEthernet1/0
 ip address 10.0.4.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet2/0
 ip address 10.0.1.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet5/0
 ip address 192.168.2.1 255.255.255.0
 negotiation auto
end
```

### Leaf3:

```
interface Loopback0
 ip address 33.33.33.33 255.255.255.255

interface GigabitEthernet3/0
 ip address 10.0.2.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet4/0
 ip address 10.0.5.2 255.255.255.252
 negotiation auto
end

interface GigabitEthernet5/0
 ip address 192.168.3.1 255.255.255.0
 negotiation auto
end

interface GigabitEthernet6/0
 ip address 192.168.4.1 255.255.255.0
 negotiation auto
end
```

## 2) Базовая настройка OSPF:

### Spine1:

```
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
```

### Spine2:

```
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 network 10.0.3.0 0.0.0.3 area 0
 network 10.0.4.0 0.0.0.3 area 0
 network 10.0.5.0 0.0.0.3 area 0
```

### Leaf1:

```
router ospf 1
 router-id 11.11.11.11
 log-adjacency-changes
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.3.0 0.0.0.3 area 0
 network 192.168.1.0 0.0.0.255 area 0
```

### Leaf2:

```
router ospf 1
 router-id 22.22.22.22
 log-adjacency-changes
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.4.0 0.0.0.3 area 0
 network 192.168.2.0 0.0.0.255 area 0

```

### Leaf3:

```
router ospf 1
 router-id 33.33.33.33
 log-adjacency-changes
 network 10.0.2.0 0.0.0.3 area 0
 network 10.0.5.0 0.0.0.3 area 0
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
```

## 3) Проверка:
### Проверка сетевой связности на примере PC1 и PC2:
```
Checking for duplicate address...
PC1 : 192.168.1.2 255.255.255.0 gateway 192.168.1.1

PC1> ping 192.168.2.2
84 bytes from 192.168.2.2 icmp_seq=3 ttl=61 time=67.435 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=61 time=63.898 ms
84 bytes from 192.168.2.2 icmp_seq=5 ttl=61 time=61.055 ms

PC1> trace 192.168.2.2
trace to 192.168.2.2, 8 hops max, press Ctrl+C to stop
 1   192.168.1.1   7.001 ms  8.970 ms  9.446 ms
 2   10.0.0.1   31.131 ms  30.971 ms  30.032 ms
 3   10.0.1.2   51.177 ms  53.174 ms  52.147 ms
 4   *192.168.2.2   62.114 ms (ICMP type:3, code:3, Destination port unreachable)
```

### Статус OSPF-соседей на примере Spine1:
```
Spine1#show ip interface br | exclude FastEthernet0/0
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet1/0         10.0.0.1        YES NVRAM  up                    up
GigabitEthernet2/0         10.0.1.1        YES NVRAM  up                    up
GigabitEthernet3/0         10.0.2.1        YES NVRAM  up                    up
Loopback0                  1.1.1.1         YES manual up                    up

Spine1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       1   FULL/BDR        00:00:33    10.0.2.2        GigabitEthernet3/0
22.22.22.22       1   FULL/BDR        00:00:31    10.0.1.2        GigabitEthernet2/0
11.11.11.11       1   FULL/BDR        00:00:34    10.0.0.2        GigabitEthernet1/0
```
### Таблица маршрутизации на примере Spine1:
```
Spine1#show ip route sum
IP routing table name is Default-IP-Routing-Table(0)
IP routing table maximum-paths is 16
Route Source    Networks    Subnets     Overhead    Memory (bytes)
connected       0           4           288         544
static          0           0           0           0
ospf 1          4           3           504         952
  Intra-area: 7 Inter-area: 0 External-1: 0 External-2: 0
  NSSA External-1: 0 NSSA External-2: 0
internal        2                                   2312
Total           6           7           792         3808
Spine1#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     1.0.0.0/32 is subnetted, 1 subnets
C       1.1.1.1 is directly connected, Loopback0
O    192.168.4.0/24 [110/2] via 10.0.2.2, 00:39:29, GigabitEthernet3/0
     10.0.0.0/30 is subnetted, 6 subnets
C       10.0.2.0 is directly connected, GigabitEthernet3/0
O       10.0.3.0 [110/2] via 10.0.0.2, 00:39:29, GigabitEthernet1/0
C       10.0.0.0 is directly connected, GigabitEthernet1/0
C       10.0.1.0 is directly connected, GigabitEthernet2/0
O       10.0.4.0 [110/2] via 10.0.1.2, 00:39:29, GigabitEthernet2/0
O       10.0.5.0 [110/2] via 10.0.2.2, 00:39:29, GigabitEthernet3/0
O    192.168.1.0/24 [110/2] via 10.0.0.2, 00:39:29, GigabitEthernet1/0
O    192.168.2.0/24 [110/2] via 10.0.1.2, 00:39:29, GigabitEthernet2/0
O    192.168.3.0/24 [110/2] via 10.0.2.2, 00:39:29, GigabitEthernet3/0
```
