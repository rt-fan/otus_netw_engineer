# BGP для маршрутизации IPv6 unicast

## Цель:

настроить eBGP/iBGP IPv6 unicast для всех сегментов сети по аналогичной логике с настройкой eBGP/iBGP IPv4 unicast.

## Задание:

1. Настроите eBGP IPv6 unicast между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроите eBGP IPv6 unicast между провайдерами Киторн и Ламас.
3. Настроите eBGP IPv6 unicast между Ламас и Триада.
4. Настроите eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада.
5. Организуете IPv6 unicast связность между пограничными роутерами офисов Москва и С.-Петербург.
6. Настроите iBGP IPv6 unicast в офисе Москва между маршрутизаторами R14 и R15.
7. Настроите iBGP IPv6 unicast в провайдере Триада, с использованием RR.
8. Все сети IPv6 в лабораторной работе должны иметь связность между собой.

<img width="1246" height="642" alt="Pasted image 20261011003951" src="https://github.com/user-attachments/assets/64e77991-1505-48e7-bc94-33efd31a4f08" />

## Настройка маршрутизаторов

### Москва - AS1001

#### R14

```
interface Ethernet0/2
 ipv6 address 2001:DB8:FFFF:14::2/64
 no shutdown
!
router bgp 1001
 address-family ipv6
  neighbor 10.100.1.2 activate
  neighbor 2001:DB8:FFFF:14::1 remote-as 101
  neighbor 2001:DB8:FFFF:14::1 activate
 exit-address-family
```

##### Диагностика

```
R14#show bgp ipv6 unicast summary
BGP router identifier 10.100.0.14, local AS number 1001
BGP table version is 12, main routing table version 12
5 network entries using 820 bytes of memory
8 path entries using 832 bytes of memory
7/4 BGP path/bestpath attribute entries using 1008 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2828 total bytes of memory
BGP activity 23/8 prefixes, 43/24 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.1.2      4         1001     109     104       12    0    0 01:22:56        4
2001:DB8:FFFF:14::1
                4          101      98      96       12    0    0 01:21:52        3
R14#
R14#show bgp ipv6 unicast
BGP table version is 12, local router ID is 10.100.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2001:DB8:14::14/128
                       ::                       0         32768 i
 * i 2001:DB8:15::15/128
                       ::FFFF:10.100.1.2
                                                0    100      0 i
 * i 2001:DB8:23::23/128
                       ::FFFF:10.100.1.2
                                                0    100      0 301 520 i
 *>                   2001:DB8:FFFF:14::1
                                                              0 101 520 i
 * i 2001:DB8:24::24/128
                       ::FFFF:10.100.1.2
                                                0    100      0 301 520 i
 *>                   2001:DB8:FFFF:14::1
          
R14#show ipv6 route bgp
IPv6 Routing Table - default - 17 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:23::23/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
B   2001:DB8:24::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6000, Ethernet0/2
```

#### R15

```
interface Ethernet0/2
 ipv6 address 2001:DB8:254:15::2/64
 no shutdown
!
router bgp 1001
 address-family ipv6
  neighbor 10.100.1.1 activate
  neighbor 2001:DB8:254:15::1 remote-as 301
  neighbor 2001:DB8:254:15::1 activate
 exit-address-family
```

##### Диагностика

```
R15#show bgp ipv6 unicast summary
BGP router identifier 10.100.0.15, local AS number 1001
BGP table version is 10, main routing table version 10
5 network entries using 820 bytes of memory
8 path entries using 832 bytes of memory
7/3 BGP path/bestpath attribute entries using 1008 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2804 total bytes of memory
BGP activity 18/3 prefixes, 47/29 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.1.1      4         1001     103     109       10    0    0 01:22:26        4
2001:DB8:254:15::1
                4          301      99      96       10    0    0 01:21:34        3
R15#
R15#show bgp ipv6 unicast
BGP table version is 10, local router ID is 10.100.0.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 2001:DB8:14::14/128
                       ::FFFF:10.100.1.1
                                                0    100      0 i
 *>  2001:DB8:15::15/128
                       ::                       0         32768 i
 * i 2001:DB8:23::23/128
                       2001:DB8:FFFF:14::1
                                                0    100      0 101 520 i
 *>                   2001:DB8:254:15::1
                                                              0 301 520 i
 * i 2001:DB8:24::24/128
                       2001:DB8:FFFF:14::1
                                                0    100      0 101 301 520 i
 *>                   2001:DB8:254:15::1
          
R15#show ipv6 route bgp
IPv6 Routing Table - default - 17 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:23::23/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   2001:DB8:24::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2
```

### Ламас - AS301

#### R21

```
ipv6 unicast-routing
!
interface Ethernet0/0
 ipv6 address 2001:DB8:254:15::1/64
 no shutdown
!
interface Ethernet0/1
 ipv6 address 2001:DB8:101:21::1/64
 no shutdown
!
interface Ethernet0/2
 ipv6 address 2001:DB8:301:24::1/64
 no shutdown
!
router bgp 301
 address-family ipv6
  neighbor 2001:DB8:254:15::2 remote-as 1001
  neighbor 2001:DB8:254:15::2 activate

  neighbor 2001:DB8:101:21::2 remote-as 101
  neighbor 2001:DB8:101:21::2 activate

  neighbor 2001:DB8:301:24::2 remote-as 520
  neighbor 2001:DB8:301:24::2 activate
 exit-address-family
```

##### Диагностика

```
R21#show bgp ipv6 unicast summary
BGP router identifier 172.16.30.1, local AS number 301
BGP table version is 10, main routing table version 10
5 network entries using 820 bytes of memory
7 path entries using 728 bytes of memory
7/5 BGP path/bestpath attribute entries using 1008 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2700 total bytes of memory
BGP activity 20/3 prefixes, 86/56 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:101:21::2
                4          101      98      95       10    0    0 01:20:51        3
2001:DB8:254:15::2
                4         1001      96      98       10    0    0 01:21:01        1
2001:DB8:301:24::2
                4          520      93      97       10    0    0 01:20:24        3
R21#
R21#show bgp ipv6 unicast
BGP table version is 10, local router ID is 172.16.30.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2001:DB8:14::14/128
                       2001:DB8:101:21::2
                                                              0 101 1001 i
 *>  2001:DB8:15::15/128
                       2001:DB8:254:15::2
                                                0             0 1001 i
 *>  2001:DB8:23::23/128
                       2001:DB8:301:24::2
                                                              0 520 i
 *                    2001:DB8:101:21::2
                                                              0 101 520 i
 *>  2001:DB8:24::24/128
                       2001:DB8:301:24::2
                                                0             0 520 i
          
R21#show ipv6 route bgp
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:14::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6010, Ethernet0/1
B   2001:DB8:15::15/128 [20/0]
     via FE80::A8BB:CCFF:FE00:F020, Ethernet0/0
B   2001:DB8:23::23/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8000, Ethernet0/2
B   2001:DB8:24::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8000, Ethernet0/2
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8000, Ethernet0/2
```

### Киторн - AS101

#### R22

```
ipv6 unicast-routing
!
interface Ethernet0/0
 ipv6 address 2001:DB8:FFFF:14::1/64
 no shutdown
!
interface Ethernet0/1
 ipv6 address 2001:DB8:101:21::2/64
 no shutdown
!
interface Ethernet0/2
 ipv6 address 2001:DB8:520:23::2/64
 no shutdown
!
router bgp 101
 address-family ipv6
  neighbor 2001:DB8:FFFF:14::2 remote-as 1001
  neighbor 2001:DB8:FFFF:14::2 activate

  neighbor 2001:DB8:101:21::1 remote-as 301
  neighbor 2001:DB8:101:21::1 activate
  
  neighbor 2001:DB8:520:23::3 remote-as 520
  neighbor 2001:DB8:520:23::3 activate
 exit-address-family
```

##### Диагностика

```
R22#show bgp ipv6 unicast summary
BGP router identifier 172.16.101.1, local AS number 101
BGP table version is 9, main routing table version 9
5 network entries using 820 bytes of memory
7 path entries using 728 bytes of memory
6/5 BGP path/bestpath attribute entries using 864 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2556 total bytes of memory
BGP activity 25/8 prefixes, 83/53 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:101:21::1
                4          301      95      97        9    0    0 01:20:08        4
2001:DB8:520:23::3
                4          520      39      47        9    0    0 00:31:49        2
2001:DB8:FFFF:14::2
                4         1001      94      96        9    0    0 01:20:07        1
R22#show bgp ipv6 unicast
BGP table version is 9, local router ID is 172.16.101.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2001:DB8:14::14/128
                       2001:DB8:FFFF:14::2
                                                0             0 1001 i
 *>  2001:DB8:15::15/128
                       2001:DB8:101:21::1
                                                              0 301 1001 i
 *   2001:DB8:23::23/128
                       2001:DB8:101:21::1
                                                              0 301 520 i
 *>                   2001:DB8:520:23::3
                                                0             0 520 i
 *>  2001:DB8:24::24/128
                       2001:DB8:101:21::1
                                                              0 301 520 i
          
R22#show ipv6 route bgp
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:14::14/128 [20/0]
     via FE80::A8BB:CCFF:FE00:E020, Ethernet0/0
B   2001:DB8:15::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5010, Ethernet0/1
B   2001:DB8:23::23/128 [20/0]
     via FE80::A8BB:CCFF:FE01:7000, Ethernet0/2
B   2001:DB8:24::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5010, Ethernet0/1
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:7000, Ethernet0/2
```

### Триада - AS520

#### R23

```
interface Ethernet0/0
 ipv6 address 2001:DB8:520:23::3/64
 no shutdown
!
ipv6 prefix-list R23-LOOPBACK seq 5 permit 2001:DB8:23::23/128

route-map R23-LOOPBACK-NH permit 10
 match ipv6 address prefix-list R23-LOOPBACK
 set ipv6 next-hop 2001:DB8:23::23

route-map R23-LOOPBACK-NH permit 20

router bgp 520
 neighbor 2001:DB8:520:23::2 remote-as 101
 address-family ipv6
  neighbor 10.24.24.24 route-map R23-LOOPBACK-NH out
  neighbor 10.26.26.26 route-map R23-LOOPBACK-NH out
  network 2001:DB8:23::23/128
  neighbor 10.24.24.24 activate
  neighbor 10.24.24.24 route-reflector-client
  neighbor 10.24.24.24 next-hop-self
  neighbor 10.26.26.26 activate
  neighbor 10.26.26.26 route-reflector-client
  neighbor 10.26.26.26 next-hop-self
  neighbor 2001:DB8:520:23::2 activate
 exit-address-family
```

##### Диагностика

```
R23#show bgp ipv6 unicast summary
BGP router identifier 10.23.23.23, local AS number 520
BGP table version is 5, main routing table version 5
6 network entries using 984 bytes of memory
9 path entries using 936 bytes of memory
7/4 BGP path/bestpath attribute entries using 1008 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3048 total bytes of memory
BGP activity 26/9 prefixes, 116/85 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.24.24.24     4          520      51      52        5    0    0 00:28:03        4
10.26.26.26     4          520      40      68        5    0    0 00:28:03        2
2001:DB8:520:23::2
                4          101      42      35        5    0    0 00:28:03        2


R23#show bgp ipv6 unicast
BGP table version is 5, local router ID is 10.23.23.23
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2001:DB8:14::14/128
                       2001:DB8:520:23::2
                                                              0 101 1001 i
 * i                  2001:DB8:301:24::1
                                                0    100      0 301 101 1001 i
 *>  2001:DB8:15::15/128
                       2001:DB8:520:23::2
                                                              0 101 301 1001 i
 * i                  2001:DB8:301:24::1
                                                0    100      0 301 1001 i
 *>  2001:DB8:23::23/128
                       ::                       0         32768 i
 * i 2001:DB8:24::24/128
                       ::FFFF:10.24.24.24


R23#show ipv6 route bgp
IPv6 Routing Table - default - 18 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:14::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6020, Ethernet0/0
B   2001:DB8:15::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:6020, Ethernet0/0
B   2001:DB8:100::18/128 [200/0]
     via 2001:DB8:52:200::2
```

#### R24

```
interface Ethernet0/0
 ipv6 address 2001:DB8:301:24::2/64
 no shutdown
!
interface Ethernet0/3
 ipv6 address 2001:DB8:52:200::1/64
 no shutdown
!
router bgp 520
 neighbor 2001:DB8:52:200::2 remote-as 2042
 neighbor 2001:DB8:301:24::1 remote-as 301
 address-family ipv6
  network 2001:DB8:24::24/128
  neighbor 10.23.23.23 activate
  neighbor 2001:DB8:52:200::2 activate
  neighbor 2001:DB8:301:24::1 activate
 exit-address-family
```

##### Диагностика

```
R24#show bgp ipv6 unicast summary
BGP router identifier 10.24.24.24, local AS number 520
BGP table version is 6, main routing table version 6
5 network entries using 820 bytes of memory
7 path entries using 728 bytes of memory
7/5 BGP path/bestpath attribute entries using 1008 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2676 total bytes of memory
BGP activity 20/4 prefixes, 79/54 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.23.23.23     4          520      58      57        6    0    0 00:34:14        3
2001:DB8:52:200::2
                4         2042      94      94        6    0    0 01:20:42        1
2001:DB8:301:24::1
                4          301      99      95        6    0    0 01:22:06        2
R24#
R24#show bgp ipv6 unicast
BGP table version is 6, local router ID is 10.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 2001:DB8:14::14/128
                       ::FFFF:10.23.23.23
                                                0    100      0 101 1001 i
 *>                   2001:DB8:301:24::1
                                                              0 301 101 1001 i
 * i 2001:DB8:15::15/128
                       ::FFFF:10.23.23.23
                                                0    100      0 101 301 1001 i
 *>                   2001:DB8:301:24::1
                                                              0 301 1001 i
 r>i 2001:DB8:23::23/128
                       2001:DB8:23::23          0    100      0 i
 *>  2001:DB8:24::24/128
                       ::                       0         32768 i
          
R24#show ipv6 route bgp
IPv6 Routing Table - default - 19 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:14::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5020, Ethernet0/0
B   2001:DB8:15::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5020, Ethernet0/0
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:2020, Ethernet0/3
```

#### R26

```
interface Ethernet0/3
 ipv6 address 2001:DB8:52:250::1/64
 no shutdown
!
router bgp 520
 neighbor 2001:DB8:52:250::2 remote-as 2042
 address-family ipv6
  network 2001:DB8:26::26/128
  neighbor 10.23.23.23 activate
  neighbor 2001:DB8:52:250::2 activate
 exit-address-family
```

##### Диагностика

```
R26#show bgp ipv6 unicast summary
BGP router identifier 10.26.26.26, local AS number 520
BGP table version is 4, main routing table version 4
5 network entries using 820 bytes of memory
6 path entries using 624 bytes of memory
6/3 BGP path/bestpath attribute entries using 864 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2404 total bytes of memory
BGP activity 26/10 prefixes, 79/58 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.23.23.23     4          520      75      48        4    0    0 00:34:57        4
2001:DB8:52:250::2
                4         2042      95      94        4    0    0 01:21:27        1
R26#
R26#show bgp ipv6 unicast
BGP table version is 4, local router ID is 10.26.26.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 2001:DB8:14::14/128
                       ::FFFF:10.23.23.23
                                                0    100      0 101 1001 i
 * i 2001:DB8:15::15/128
                       ::FFFF:10.23.23.23
                                                0    100      0 101 301 1001 i
 r>i 2001:DB8:23::23/128
                       2001:DB8:23::23          0    100      0 i
 *>  2001:DB8:26::26/128
                       ::                       0         32768 i
 * i 2001:DB8:100::18/128
                       2001:DB8:52:200::2
                                                0    100      0 2042 i
 *>                   2001:DB8:52:250::2
          
R26#show ipv6 route bgp
IPv6 Routing Table - default - 15 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:100::18/128 [20/0]
     via FE80::A8BB:CCFF:FE01:2030, Ethernet0/3
```

### Санкт-Петербург - AS2042

#### R18

```
interface Ethernet0/2
 ipv6 address 2001:DB8:52:200::2/64
 no shutdown
!
interface Ethernet0/3
 ipv6 address 2001:DB8:52:250::2/64
 no shutdown
!
router bgp 2042
 address-family ipv6
  neighbor 2001:DB8:52:200::1 remote-as 520
  neighbor 2001:DB8:52:200::1 activate

  neighbor 2001:DB8:52:250::1 remote-as 520
  neighbor 2001:DB8:52:250::1 activate
 exit-address-family
```

##### Диагностика

```
R18#show bgp ipv6 unicast summary
BGP router identifier 10.20.42.18, local AS number 2042
BGP table version is 7, main routing table version 7
6 network entries using 984 bytes of memory
7 path entries using 728 bytes of memory
5/5 BGP path/bestpath attribute entries using 720 bytes of memory
4 BGP AS-PATH entries using 112 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2544 total bytes of memory
BGP activity 17/0 prefixes, 32/7 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:52:200::1
                4          520      96      95        7    0    0 01:21:49        4
2001:DB8:52:250::1
                4          520      95      95        7    0    0 01:21:52        2
R18#
R18#show bgp ipv6 unicast
BGP table version is 7, local router ID is 10.20.42.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  2001:DB8:14::14/128
                       2001:DB8:52:200::1
                                                              0 520 301 101 1001 i
 *>  2001:DB8:15::15/128
                       2001:DB8:52:200::1
                                                              0 520 301 1001 i
 *   2001:DB8:23::23/128
                       2001:DB8:52:250::1
                                                              0 520 i
 *>                   2001:DB8:52:200::1
                                                              0 520 i
 *>  2001:DB8:24::24/128
                       2001:DB8:52:200::1
          
R18#show ipv6 route bgp
IPv6 Routing Table - default - 25 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:14::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   2001:DB8:15::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   2001:DB8:23::23/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   2001:DB8:24::24/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   2001:DB8:26::26/128 [20/0]
     via FE80::A8BB:CCFF:FE01:A030, Ethernet0/3
```
