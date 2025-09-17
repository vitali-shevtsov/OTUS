## Топология для настройки VxLAN EVPN для L3

<img width="482" height="420" alt="lab6_topo" src="https://github.com/user-attachments/assets/d85dc045-b792-4899-93e5-6bd10b7089ed" />

Для примера используется минимальная топология с одним Spine и парой Leaf'ов.
SRV1 и SRV2 находятся в разных сетях, и чтобы их соединить настроим distributed VXLAN gateways. 
Leaf1 находится в AS200, Leaf2 - AS300, Spine - AS100. 
Leaf1, Leaf2, Spine используют AS100 для BGP EVPN.
