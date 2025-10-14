## НАСТРОЙКА L3-СВЯЗНОСТИ МЕЖДУ ЦОДАМИ ЧЕРЕЗ VxLAN И BGP EVPN

 ### Исходные данные

Имеется два ЦОДа, необходимо обеспечить сетевую связность между сегментами как в пределах каждого ЦОДа, так и между сегментами разных ЦОДов; использовать VxLAN и BGP EVPN.

<img width="712" height="411" alt="image" src="https://github.com/user-attachments/assets/c19e1465-18ae-426a-957b-1143986b0729" />

### Структурная схема

Структурная схема масштабируемой IP-фабрики для ЦОДов в классической топологии Clos может иметь примерно следующий вид:

<img width="894" height="456" alt="image" src="https://github.com/user-attachments/assets/360658fe-be42-42a1-be6d-3058ebf22de3" />

### Физическая топология

Для реализации проекта собрана минимальная физическая топология, где каждое устройство выполняет функции VTEP. Все клиенты (SRV-1, SRV-2, SRV-3, SRV-4) находятся в разных подсетях, поэтому нужно обеспечить полную L3-связность между ними:

<img width="854" height="496" alt="image" src="https://github.com/user-attachments/assets/c52e64a9-6b3e-4962-bd83-1a68e9e7d793" />

### Схема VxLAN-туннелей

Логическая связность между сетевыми хостами будет реализована посредством VxLAN-туннелей следующим образом:

<img width="926" height="568" alt="image" src="https://github.com/user-attachments/assets/2976a752-5fc0-4f96-bd93-8df1526f3f6c" />

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

![project evpn inst](https://github.com/user-attachments/assets/7df4eb99-cedb-4f8f-b61d-edccf7b532de)

### Таблица VPN-инстансов

![project vpn inst](https://github.com/user-attachments/assets/af62acbb-e6a3-4ef0-9c2e-e8f50f28dca7)

## Настройка

### 1. Настройка адресации и протокола маршрутизации для underlay-сети
### На примере VTEP1 (аналогично для остальных сетевых хостов):
