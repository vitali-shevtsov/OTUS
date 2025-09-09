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
