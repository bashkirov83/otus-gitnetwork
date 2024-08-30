# DHCP v4, часть 1

**Подсеть сети 192.168.1.0/24 для удовлетворения следующих требований:**

Одна подсеть, «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на маршрутизаторе R1).
Подсеть А: 192.168.1.0/26
Запишите первый IP-адрес в таблице адресации для маршрутизатора R1 e0/1.100.
Одна подсеть, «Подсеть B», поддерживающая 28 хостов (VLAN управления на маршрутизаторе R1).
Подсеть Б: 192.168.1.64/27
Запишите первый IP-адрес в таблице адресации для маршрутизатора R1 e0/1.200. Запишите второй IP-адрес в таблице адресов для S1 VLAN 200 и введите соответствующий шлюз по умолчанию.
Одна подсеть «Подсеть C», поддерживающая 12 хостов (клиентская сеть на маршрутизаторе R2).
Подсеть C: 192.168.1.96/28
Запишите первый IP-адрес в таблице адресации для маршрутизатора R2 e0/1.

# Настраиваем R1:
interface ethernet 0/1
interface ethernet 0/1.100
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
description Clients
interface ethernet 0/1.200
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
description Management
interface ethernet 0/1.1000
encapsulation dot1Q 1000 native
description Native
ip route 0.0.0.0 0.0.0.0 10.0.0.2

# Настраиваем R2:
interface ethernet 0/1
ip address 192.168.1.97 255.255.255.240
ip route 0.0.0.0 0.0.0.0 10.0.0.1

**Проверяем линк ping-ом**
R2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms 


# Настраиваем SW1:
vlan 100
name Clients
vlan 200
name Management
vlan 999
name Parking_Lot
vlan 1000
name Native
interface Vlan200
ip address 192.168.1.66 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 192.168.1.65
interface ethernet 0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 100,200,1000
switchport trunk native vlan 1000
interface Ethernet0/3
switchport mode access
switchport access vlan 100
spanning-tree portfast
interface range ethernet 0/1 - 2
switchport mode access
switchport access vlan 999
shutdown

# Настраиваем SW2:
interface vlan 1
ip address 192.168.1.98 255.255.255.240
no shutdown
interface range ethernet 0/1 - 2
shutdown
ip route 0.0.0.0 0.0.0.0 192.168.1.97

**Проверяем линк ping-ом SW2 to SW1**
S2#ping 192.168.1.66
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.66, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/5 ms

**Вопрос**
Какие ip-адреса будут иметь компьютеры, подключённые к данной сети и использующие DHCP? 
**Ответ**
Компьютерам будут назначены адреса ipv4 link-local



# Настройка DHCPv4 на R1
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.97 192.168.1.101
ip dhcp pool R1_Client_LAN
network 192.168.1.0/26
domain-name otus.lab
default-router 192.168.1.1
lease 2 12 30
ip dhcp pool R2_Client_LAN
network 192.168.1.96/28
domain-name ccna-lab.com
default-router 192.168.1.97
lease 2 12 30

# Проверка DHCPv4 на R1
R1#show ip dhcp pool

**Pool R1_Client_LAN** :
Utilization mark (high/low)    : 100 / 0
Subnet size (first/next)       : 0 / 0
Total addresses                : 62
Leased addresses               : 0
Pending event                  : none
1 subnet is currently in the pool :
Current index        IP address range                    Leased addresses
192.168.1.1          192.168.1.1      - 192.168.1.62      0

**Pool R2_Client_LAN :**
Utilization mark (high/low)    : 100 / 0
Subnet size (first/next)       : 0 / 0
Total addresses                : 14
Leased addresses               : 0
Pending event                  : none
1 subnet is currently in the pool :
Current index        IP address range                    Leased addresses
192.168.1.97         192.168.1.97     - 192.168.1.110     0

**Получение адреса на PC1**
VPCS> ip dhcp -r
DDORA IP 192.168.1.6/26 GW 192.168.1.1

**VPCS> ping 10.0.0.2**
84 bytes from 10.0.0.2 icmp_seq=1 ttl=254 time=1.171 ms
84 bytes from 10.0.0.2 icmp_seq=2 ttl=254 time=2.455 ms

# DHCP Relay на R2
interface ethernet 0/1
ip helper-address 10.0.0.1

**Получение адреса на PC2**
VPCS> ip dhcp -r
DDORA IP 192.168.1.102/28 GW 192.168.1.97

**VPCS> ping 10.0.0.1**
84 bytes from 10.0.0.1 icmp_seq=1 ttl=254 time=2.089 ms
84 bytes from 10.0.0.1 icmp_seq=2 ttl=254 time=3.836 ms