## Топология для настройки VxLAN EVPN для L3

<img width="482" height="420" alt="lab6_topo" src="https://github.com/user-attachments/assets/d85dc045-b792-4899-93e5-6bd10b7089ed" />

Для примера используется минимальная топология с одним Spine и парой Leaf'ов.

SRV1 и SRV2 находятся в разных сетях, и чтобы их соединить настроим distributed VXLAN gateways. 

Leaf1 находится в AS200, Leaf2 - AS300, Spine - AS100. 
Leaf1, Leaf2, Spine используют AS100 для BGP EVPN.

### План работы

1. Настроить eBGP между Spine и Leaf1 и между Spine и Leaf2.
2. Настроить клиентские порты на Leaf'ах.
3. Настроить EVPN в качестве VXLAN control plane.
4. Настроить Spine как BGP EVPN-пир для Leaf'ов.
5. Настроить Leaf'ы как BGP EVPN-пиры - RR-клиенты.
6. Настроить VPN- и EVPN-инстансы на Leaf'ах.
7. Настроить Leaf'ы как Layer 3 VXLAN gateways.
8. Настроить BGP между Spine и Leaf'ами для анонсов IRB-маршрутов.

### Начальные данные
 - VLAN'ы для клиентов 10 и 20
 - BD соответственно 10 и 20
 - VNI 10 и 20
 - VNI 5010 для VPN instance
 - RDs и RTs для EVPN и VPN-инстансов в таблице ниже
   
<img width="372" height="262" alt="rd rt" src="https://github.com/user-attachments/assets/66234e02-3545-45dc-96bb-ecd34e260983" />
