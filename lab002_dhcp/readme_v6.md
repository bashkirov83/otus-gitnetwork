# Настраиваем R1
ipv6 unicast-routing
interface ethernet 0/0
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:2::1/64
interface ethernet 0/1.100
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:1::1/64
ipv6 route ::/0 2001:db8:acad:2::2

# Настраиваем R2
ipv6 unicast-routing
interface ethernet 0/0
ipv6 address fe80::2 link-local
ipv6 address 2001:db8:acad:2::2/64
interface ethernet 0/1
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:3::1/64
ipv6 route ::/0 2001:db8:acad:2::1

R1#ping 2001:db8:acad:3::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:ACAD:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

**PC1 алгоритм SLAAC, получение адреса**
VPCS> ip auto
GLOBAL SCOPE      : 2001:db8:acad:1:2050:79ff:fe66:6805/64
ROUTER LINK-LAYER : aa:bb:cc:00:01:10

#  DHCPv6 сервер на R1
ipv6 dhcp pool R1-STATELESS
dns-server 2001:db8:acad::254
domain-name statless.com
interface ethernet 0/1.100
ipv6 nd other-config-flag
ipv6 dhcp server R1-STATELESS

# Statefull DHCPv6 сервер на R1
ipv6 dhcp pool R2-STATEFUL
address prefix 2001:db8:acad:3:aaa::/80
dns-server 2001:db8:acad::254
domain-name statefull.com
interface ethernet 0/0
ipv6 dhcp server R2-STATEFUL

# Relay на R2
interface ethernet 0/1
ipv6 nd managed-config-flag
ipv6 dhcp relay destination 2001:db8:acad:2::1 e0/0

