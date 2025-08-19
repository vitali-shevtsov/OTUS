
<img width="878" height="447" alt="Topology" src="https://github.com/user-attachments/assets/e5343781-8475-48ac-822b-f7f8626f7776" />


![ip plan](https://github.com/user-attachments/assets/159fdeb7-0d0f-48fb-810d-d1845e5c4b42)

Настройка интерфейсов согласно IP-плана:

### Spine1:

```
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
