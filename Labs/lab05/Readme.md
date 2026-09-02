# EIGRP. Настройка EIGRP named-mode для IPv4 и IPv6 в офисе С.-Петербург.

## Задание:
1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.
5. Настройка осуществляется одновременно для IPv4 и IPv6.

## Адресация IPv4

Используем /30 для линков между маршрутизаторами.

| Линк         | Сеть            | Адреса                                         |
| ------------ | --------------- | ---------------------------------------------- |
| R18 — R16    | 10.20.42.4/30   | R18 (0/0) 10.20.42.5, R16 (0/1) 10.20.42.6     |
| R18 — R17    | 10.20.42.0/30   | R18 (0/1) 10.20.42.1, R17 (0/1) 10.20.42.2     |
| R17 — SW9    |                 | R17 (0/0) -> SW9 (0/3)                         |
| R17 — SW10   |                 | R17 (0/2) -> SW10 (1/0)                        |
| R16 — R32    | 10.20.42.10/30  | R16 (0/3) 10.20.42.11, R32 (0/0) 10.20.42.12   |
| R16 — SW9    | /30             | R16 (0/0) -> SW10 (0/3)                        |
| R16 — SW10   | /30             | R16 (0/2) -> SW9 (1/0)                         |
| R16 Loopback | 10.20.42.16/32  |                                                |
| R17 Loopback | 10.20.42.17/32  |                                                |
| R18 Loopback | 10.20.42.18/32  |                                                |
| R32 Loopback | 10.20.42.32/32  |                                                |

## Адресация IPv6

| Линк         | IPv6-сеть              | Адреса                         |
| ------------ | ---------------------- | ------------------------------ |
| R18 — R16    | `2001:DB8:42:4::/64`   | R18 (0/0)  ::1, R16 (0/1)  ::2 |
| R18 — R17    | `2001:DB8:42:0::/64`   | R18 (0/1)  ::1, R17 (0/1)  ::2 |
| R16 — R32    | `2001:DB8:42:10::/64`  | R18 (0/3)  ::1, R32 (0/0)  ::2 |
| R16 Loopback | `2001:DB8:100::16/128` |                                |
| R17 Loopback | `2001:DB8:100::17/128` |                                |
| R18 Loopback | `2001:DB8:100::18/128` |                                |
| R32 Loopback | `2001:DB8:100::32/128` |                                |


## Настройка маршрутизаторов

#### R18

```
ipv6 cef
ipv6 unicast-routing
!
interface Ethernet0/0
descr R16
 ip address 10.20.42.5 255.255.255.252
 ipv6 address 2001:DB8:42:4::1/64
 no shutdown
!
interface Ethernet0/1
descr R17
 ip address 10.20.42.1 255.255.255.252
 ipv6 address 2001:DB8:42:0::1/64
 no shutdown
!
interface Loopback0
 ip address 10.20.42.18 255.255.255.255
 ipv6 address 2001:DB8:100::18/128
!
ip route 0.0.0.0 0.0.0.0 10.52.200.5
ip route 0.0.0.0 0.0.0.0 10.52.250.13 10
!
ipv6 route ::/0 2001:DB8:52:200::1
ipv6 route ::/0 2001:DB8:52:250::1 10
!
router eigrp SPB
!
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 10.20.42.18
  network 10.20.42.0 0.0.0.255
  network 10.52.200.4 0.0.0.3
  network 10.52.250.12 0.0.0.3
  !       
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  af-interface Loopback0
   no passive-interface
  exit-af-interface
!
 address-family ipv6 unicast autonomous-system 100
  eigrp router-id 10.20.42.18
!
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !       
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  eigrp router-id 10.20.42.18
 exit-address-family
```

#### R16

```
ipv6 cef
ipv6 unicast-routing
!
interface Ethernet0/0
 description SW10
 no ip address
 no shutdown
!
interface Ethernet0/1
description R18
 ip address 10.20.42.6 255.255.255.252
 ipv6 address 2001:DB8:42:4::2/64
 no shutdown
!
interface Ethernet0/2
 description SW9
 no ip address
 no shutdown
!
interface Ethernet0/3
description R32
 ip address 10.20.42.9 255.255.255.252
 ipv6 address 2001:DB8:42:10::1/64
 no shutdown
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.20.20.1 255.255.255.252
 ipv6 address 2001:DB8:20::1/64
!
interface Ethernet0/2.10
 encapsulation dot1Q 10
 ip address 10.20.10.1 255.255.255.252
 ipv6 address 2001:DB8:10::1/64
!
interface Loopback0
 ip address 10.20.42.16 255.255.255.255
 ipv6 address 2001:DB8:100::16/128
!
ip prefix-list DEFAULT seq 5 permit 0.0.0.0/0
ipv6 prefix-list DEFAULT-v6 seq 5 permit ::/0
!
router eigrp SBP
 !
 address-family ipv4 unicast autonomous-system 100
  !
  eigrp router-id 10.20.42.16
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0.20
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   summary-address 10.20.43.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   summary-address 0.0.0.0 0.0.0.0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  !
  network 10.20.42.0 0.0.0.255
  network 10.20.43.0 0.0.0.7
  network 10.20.20.0 0.0.0.3
  !
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 100
  !
  eigrp router-id 10.20.42.16
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   summary-address ::/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
```

#### R17

```
ipv6 cef
ipv6 unicast-routing
!
interface Ethernet0/0
 description SW9
 no ip address
 no shutdown
!
interface Ethernet0/1
 description R18
 ip address 10.20.42.2 255.255.255.252
 ipv6 address 2001:DB8:42:0::2/64
 no shutdown
!
interface Ethernet0/2
 description SW10
 no ip address
 no shutdown
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.20.10.2 255.255.255.252
 ipv6 address 2001:DB8:10::2/64
!
interface Ethernet0/2.20
 encapsulation dot1Q 20
 ip address 10.20.20.2 255.255.255.252
 ipv6 address 2001:DB8:20::2/64
!
interface Loopback0
 ip address 10.20.42.17 255.255.255.255
 ipv6 address 2001:DB8:100::17/128

router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   summary-address 10.20.43.0 255.255.255.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2.20
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  !
  network 10.20.42.0 0.0.0.255
  network 10.20.43.0 0.0.0.15
  network 10.20.20.0 0.0.0.3
  !
  eigrp router-id 10.20.42.17
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 100
  !
  eigrp router-id 10.20.42.17
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
```

#### R32

```
ipv6 cef
ipv6 unicast-routing
!
interface Ethernet0/0
 description R16
 ip address 10.20.42.10 255.255.255.252
 ipv6 address 2001:DB8:42:10::2/64
 no shutdown

interface Loopback0
 ip address 10.20.42.32 255.255.255.255
 ipv6 address 2001:DB8:100::32/128

router eigrp R32
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  !
  network 10.20.42.0 0.0.0.255
  !
  eigrp router-id 10.20.42.32
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 100
  !
  eigrp router-id 10.20.42.32
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
```

#### SW9

```
interface Port-channel1
 description SW10  
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Ethernet0/0
 description SW10  
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/1
 description SW10  
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/2
 description VPC8  
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
!
interface Ethernet0/3
 description R17
 switchport trunk allowed vlan 10,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Ethernet1/0
 description R16  
 switchport trunk allowed vlan 10,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Vlan99
 ip address 10.20.99.9 255.255.255.0
!
ip default-gateway 10.20.99.17
ip forward-protocol nd
!
no ip http server
no ip http secure-server
```

#### SW10

```
interface Port-channel1
 description SW9
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Ethernet0/0
 description SW9
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode passive
!
interface Ethernet0/1
 description SW9
 switchport trunk allowed vlan 10,20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode passive
!
interface Ethernet0/2
 description VPC10
 switchport access vlan 20
 switchport mode access
 spanning-tree portfast
!
interface Ethernet0/3
 description R16
 switchport trunk allowed vlan 20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Ethernet1/0
 description R17
 switchport trunk allowed vlan 20,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Vlan99
 ip address 10.20.99.10 255.255.255.0
!
ip default-gateway 10.20.99.16
ip forward-protocol nd
!
no ip http server
no ip http secure-server
```

## Диагностика

```
show ip interface brief
show ipv6 interface brief
show ip eigrp neighbors
show ipv6 eigrp neighbors

show ip eigrp topology
show ipv6 eigrp topology

show ip route 0.0.0.0
```

#### R18

```
R18#show ip route 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 10.52.200.5
      Route metric is 0, traffic share count is 1


R18#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.20.42.5      YES NVRAM  up                    up      
Ethernet0/1                10.20.42.1      YES NVRAM  up                    up      
Ethernet0/2                10.52.200.6     YES NVRAM  up                    up      
Ethernet0/3                10.52.250.14    YES NVRAM  up                    up      
Loopback0                  10.20.42.18     YES NVRAM  up                    up


R18#show ipv6 interface brief
Ethernet0/0            [up/up]
    FE80::A8BB:CCFF:FE01:2000
    2001:DB8:42:4::1
Ethernet0/1            [up/up]
    FE80::A8BB:CCFF:FE01:2010
    2001:DB8:42::1
Ethernet0/2            [up/up]
    FE80::A8BB:CCFF:FE01:2020
    2001:DB8:52:200::2
Ethernet0/3            [up/up]
    FE80::A8BB:CCFF:FE01:2030
    2001:DB8:52:250::2
Loopback0              [up/up]
    FE80::A8BB:CCFF:FE01:2000
    2001:DB8:100::18


R18#show ip eigrp neighbors
EIGRP-IPv4 VR(SPB) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.20.42.6              Et0/0                    11 00:01:19    8   100  0  13
0   10.20.42.2              Et0/1                    12 00:01:20   13   100  0  6


R18#show ipv6 eigrp neighbors
EIGRP-IPv6 VR(SPB) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   Link-local address:     Et0/0                    11 00:01:43    6   100  0  13
    FE80::A8BB:CCFF:FE01:10
0   Link-local address:     Et0/1                    12 00:01:43    7   100  0  9
    FE80::A8BB:CCFF:FE01:1010


R18#show ip eigrp topology
EIGRP-IPv4 VR(SPB) Topology Table for AS(100)/ID(10.20.42.18)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 10.20.42.18/32, 1 successors, FD is 163840
        via Connected, Loopback0
P 10.20.20.0/30, 2 successors, FD is 196608000
        via 10.20.42.2 (196608000/131072000), Ethernet0/1
        via 10.20.42.6 (196608000/131072000), Ethernet0/0
P 10.20.42.16/32, 1 successors, FD is 131153920
        via 10.20.42.6 (131153920/163840), Ethernet0/0
P 10.20.42.32/32, 1 successors, FD is 196689920
        via 10.20.42.6 (196689920/131153920), Ethernet0/0
P 10.52.250.12/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/3
P 10.20.42.0/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/1
P 10.20.42.17/32, 1 successors, FD is 131153920
        via 10.20.42.2 (131153920/163840), Ethernet0/1
P 10.20.42.8/30, 1 successors, FD is 196608000
        via 10.20.42.6 (196608000/131072000), Ethernet0/0
P 10.20.42.4/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/0
P 10.52.200.4/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/2


R18#show ipv6 eigrp topology
EIGRP-IPv6 VR(SPB) Topology Table for AS(100)/ID(10.20.42.18)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 2001:DB8:42::/64, 1 successors, FD is 131072000
        via Connected, Ethernet0/1
P 2001:DB8:43:2::/64, 1 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:10 (196608000/131072000), Ethernet0/0
P 2001:DB8:43:4::/64, 1 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:1010 (196608000/131072000), Ethernet0/1
P 2001:DB8:100::17/128, 1 successors, FD is 131153920
        via FE80::A8BB:CCFF:FE01:1010 (131153920/163840), Ethernet0/1
P 2001:DB8:42:4::/64, 1 successors, FD is 131072000
        via Connected, Ethernet0/0
P 2001:DB8:100::32/128, 1 successors, FD is 196689920
        via FE80::A8BB:CCFF:FE01:10 (196689920/131153920), Ethernet0/0
P 2001:DB8:10::/64, 2 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:10 (196608000/131072000), Ethernet0/0
        via FE80::A8BB:CCFF:FE01:1010 (196608000/131072000), Ethernet0/1
P 2001:DB8:20::/64, 2 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:10 (196608000/131072000), Ethernet0/0
        via FE80::A8BB:CCFF:FE01:1010 (196608000/131072000), Ethernet0/1
P 2001:DB8:100::16/128, 1 successors, FD is 131153920
        via FE80::A8BB:CCFF:FE01:10 (131153920/163840), Ethernet0/0
P 2001:DB8:43:1::/64, 1 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:10 (196608000/131072000), Ethernet0/0
P 2001:DB8:52:200::/64, 1 successors, FD is 131072000
        via Connected, Ethernet0/2
P 2001:DB8:42:10::/64, 1 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:10 (196608000/131072000), Ethernet0/0
P 2001:DB8:100::18/128, 1 successors, FD is 163840
        via Connected, Loopback0
P 2001:DB8:43:3::/64, 1 successors, FD is 196608000
        via FE80::A8BB:CCFF:FE01:1010 (196608000/131072000), Ethernet0/1
P 2001:DB8:52:250::/64, 1 successors, FD is 131072000
        via Connected, Ethernet0/3
```

#### R16

```
R16#show ip route 0.0.0.0  
Routing entry for 0.0.0.0/0, supernet
  Known via "eigrp 100", distance 5, metric 1280, candidate default path, type internal
  Redistributing via eigrp 100
  Routing Descriptor Blocks:
  * directly connected, via Null0
      Route metric is 1280, traffic share count is 1
      Total delay is 2 microseconds, minimum bandwidth is 8000000 Kbit
      Reliability 255/255, minimum MTU 1514 bytes
      Loading 1/255, Hops 0
R16#
R16#
R16#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  up                    up      
Ethernet0/0.20             10.20.20.1      YES NVRAM  up                    up      
Ethernet0/1                10.20.42.6      YES NVRAM  up                    up      
Ethernet0/2                unassigned      YES NVRAM  up                    up      
Ethernet0/2.10             10.20.10.1      YES NVRAM  up                    up      
Ethernet0/3                10.20.42.9      YES NVRAM  up                    up      
Loopback0                  10.20.42.16     YES NVRAM  up                    up      
R16#
R16#show ip eigrp neighbors
EIGRP-IPv4 VR(SBP) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
2   10.20.20.2              Et0/0.20                 11 00:18:49   21   126  0  7
1   10.20.42.5              Et0/1                    13 00:19:26   11   100  0  11
0   10.20.42.10             Et0/3                    11 00:19:26    1  4500  0  3
R16#
R16#show ip route eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

D*    0.0.0.0/0 is a summary, 00:19:31, Null0
      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
D        10.20.42.0/30 [90/1536000] via 10.20.42.5, 00:18:53, Ethernet0/1
                       [90/1536000] via 10.20.20.2, 00:18:53, Ethernet0/0.20
D        10.20.42.17/32 [90/1024640] via 10.20.20.2, 00:18:53, Ethernet0/0.20
D        10.20.42.18/32 [90/1024640] via 10.20.42.5, 00:18:53, Ethernet0/1
D        10.20.42.32/32 [90/1024640] via 10.20.42.10, 00:18:48, Ethernet0/3
D        10.52.200.4/30 [90/1536000] via 10.20.42.5, 00:18:53, Ethernet0/1
D        10.52.250.12/30 [90/1536000] via 10.20.42.5, 00:18:53, Ethernet0/1
R16#
R16#
R16#
R16#show ip eigrp topology
EIGRP-IPv4 VR(SBP) Topology Table for AS(100)/ID(10.20.42.16)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 10.20.42.18/32, 1 successors, FD is 131153920
        via 10.20.42.5 (131153920/163840), Ethernet0/1
P 10.20.20.0/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/0.20
P 10.20.42.16/32, 1 successors, FD is 163840
        via Connected, Loopback0
P 10.20.42.32/32, 1 successors, FD is 131153920
        via 10.20.42.10 (131153920/163840), Ethernet0/3
P 10.52.250.12/30, 1 successors, FD is 196608000
        via 10.20.42.5 (196608000/131072000), Ethernet0/1
P 10.20.42.0/30, 2 successors, FD is 196608000
        via 10.20.20.2 (196608000/131072000), Ethernet0/0.20
        via 10.20.42.5 (196608000/131072000), Ethernet0/1
P 10.20.42.17/32, 1 successors, FD is 131153920
        via 10.20.20.2 (131153920/163840), Ethernet0/0.20
P 0.0.0.0/0, 1 successors, FD is 163840
        via Summary (163840/0), Null0
P 10.20.42.8/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/3
P 10.20.42.4/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/1
P 10.52.200.4/30, 1 successors, FD is 196608000
        via 10.20.42.5 (196608000/131072000), Ethernet0/1
```

#### R17

```
R17#show ip route 0.0.0.0
% Network not in table
R17#
R17#
R17#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  up                    up      
Ethernet0/0.10             10.20.10.2      YES NVRAM  up                    up      
Ethernet0/1                10.20.42.2      YES NVRAM  up                    up      
Ethernet0/2                unassigned      YES NVRAM  up                    up      
Ethernet0/2.20             10.20.20.2      YES NVRAM  up                    up      
Ethernet0/3                unassigned      YES NVRAM  administratively down down    
Loopback0                  10.20.42.17     YES NVRAM  up                    up      
R17#
R17#
R17#show ip eigrp neighbors
EIGRP-IPv4 VR(SPB) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.20.20.1              Et0/2.20                 14 00:28:02 1597  5000  0  14
0   10.20.42.1              Et0/1                    11 00:28:39 1272  5000  0  10
R17#
R17#
R17#show ip eigrp topology
EIGRP-IPv4 VR(SPB) Topology Table for AS(100)/ID(10.20.42.17)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 10.20.42.18/32, 1 successors, FD is 131153920
        via 10.20.42.1 (131153920/163840), Ethernet0/1
P 10.20.20.0/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/2.20
P 10.20.42.16/32, 1 successors, FD is 131153920
        via 10.20.20.1 (131153920/163840), Ethernet0/2.20
P 10.20.42.32/32, 1 successors, FD is 196689920
        via 10.20.20.1 (196689920/131153920), Ethernet0/2.20
P 10.52.250.12/30, 1 successors, FD is 196608000
        via 10.20.42.1 (196608000/131072000), Ethernet0/1
P 10.20.42.0/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/1
P 10.20.42.17/32, 1 successors, FD is 163840
        via Connected, Loopback0
P 10.20.42.8/30, 1 successors, FD is 196608000
        via 10.20.20.1 (196608000/131072000), Ethernet0/2.20
P 10.20.42.4/30, 2 successors, FD is 196608000
        via 10.20.20.1 (196608000/131072000), Ethernet0/2.20
        via 10.20.42.1 (196608000/131072000), Ethernet0/1
P 10.52.200.4/30, 1 successors, FD is 196608000
        via 10.20.42.1 (196608000/131072000), Ethernet0/1
```

#### R32

```
R32#show ip route 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "eigrp 100", distance 90, metric 1024640, candidate default path, type internal
  Redistributing via eigrp 100
  Last update from 10.20.42.9 on Ethernet0/0, 00:29:57 ago
  Routing Descriptor Blocks:
  * 10.20.42.9, from 10.20.42.9, 00:29:57 ago, via Ethernet0/0
      Route metric is 1024640, traffic share count is 1
      Total delay is 1002 microseconds, minimum bandwidth is 10000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 1
R32#
R32#
R32#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.20.42.10     YES NVRAM  up                    up      
Ethernet0/1                unassigned      YES NVRAM  administratively down down    
Ethernet0/2                unassigned      YES NVRAM  administratively down down    
Ethernet0/3                unassigned      YES NVRAM  administratively down down    
Loopback0                  10.20.42.32     YES NVRAM  up                    up      
R32#
R32#
R32#show ip eigrp neighbors
EIGRP-IPv4 VR(R32) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   10.20.42.9              Et0/0                    10 00:30:11    3   100  0  5
R32#
R32#
R32#show ip eigrp topology
EIGRP-IPv4 VR(R32) Topology Table for AS(100)/ID(10.20.42.32)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 10.20.42.32/32, 1 successors, FD is 163840
        via Connected, Loopback0
P 0.0.0.0/0, 1 successors, FD is 131153920
        via 10.20.42.9 (131153920/163840), Ethernet0/0
P 10.20.42.8/30, 1 successors, FD is 131072000
        via Connected, Ethernet0/0
```
