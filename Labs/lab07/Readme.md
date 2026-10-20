# iBGP

## Цель:

настроить iBGP в офисе Москва и в сети провайдера Триада для обеспечения полной IP-связности всех сетей.

## Задание:

1. Настроить iBGP в офисом Москва между маршрутизаторами R14 и R15.
2. Настроить iBGP в провайдере Триада, с использованием RR.
3. Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас.
4. Настроить офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
5. Все сети в лабораторной работе должны иметь IP связность.

<img width="1246" height="642" alt="Pasted image 20261011003951" src="https://github.com/user-attachments/assets/64e77991-1505-48e7-bc94-33efd31a4f08" />

## Настройка маршрутизаторов

### Москва - AS1001

#### R14

```
router bgp 1001
 neighbor 10.100.1.2 remote-as 1001
 neighbor 10.100.1.2 next-hop-self
```

#### R15

```
router bgp 1001
 neighbor 10.100.1.1 remote-as 1001
 neighbor 10.100.1.1 next-hop-self
 neighbor 10.100.254.6 route-map LAMAS-IN in
!
route-map LAMAS-IN permit 10
 set local-preference 200
exit
```

*Запросим повторную обработку маршрутов, в  случае если приоритет маршрутов в сторону Ламас не заработает автоматически.*

```
clear ip bgp 10.100.254.6 soft in
```

### Триада - AS520

#### R23 (RR)

```
router bgp 520
 neighbor 10.24.24.24 remote-as 520
 neighbor 10.24.24.24 update-source Loopback0
 neighbor 10.24.24.24 route-reflector-client
 
 neighbor 10.26.26.26 remote-as 520
 neighbor 10.26.26.26 update-source Loopback0
 neighbor 10.26.26.26 route-reflector-client
```

#### R24

```
router bgp 520
 neighbor 10.23.23.23 remote-as 520
 neighbor 10.23.23.23 update-source Loopback0
 neighbor 10.23.23.23 next-hop-self
```

#### R26

```
router bgp 520
 neighbor 10.23.23.23 remote-as 520
 neighbor 10.23.23.23 update-source Loopback0
 neighbor 10.23.23.23 next-hop-self
```

### Санкт-Петербург - AS2042

#### R18

```
router bgp 2042
 maximum-paths 2
```


## Диагностика

```
show ip bgp
show ip route bgp
show ip bgp summary
show ip route isis
```

### Москва - AS1001

#### R14

```
R14#show ip bgp
BGP table version is 32, local router ID is 10.100.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.20.42.0/30    10.100.1.2               0    200      0 301 520 2042 i
 *                    10.100.254.2                           0 101 520 2042 i
 *>i 10.20.42.4/30    10.100.1.2               0    200      0 301 520 2042 i
 *                    10.100.254.2                           0 101 520 2042 i
 *>i 10.52.200.4/30   10.100.1.2               0    200      0 301 520 2042 i
 *                    10.100.254.2                           0 101 520 2042 i
 *>i 10.52.250.12/30  10.100.1.2               0    200      0 301 520 2042 i
 *                    10.100.254.2                           0 101 520 2042 i
 *>  10.100.1.0/30    0.0.0.0                  0         32768 i
 *>  10.100.1.4/30    0.0.0.0                  0         32768 i
 *>  10.100.1.8/30    10.100.1.2              20         32768 i
 *>  10.100.1.24/30   0.0.0.0                  0         32768 i
 *>  10.100.1.28/30   10.100.1.2              20         32768 i
 *>  10.100.254.0/30  0.0.0.0                  0         32768 i

R14#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.100.254.2 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 17 subnets, 2 masks
B        10.20.42.0/30 [200/0] via 10.100.1.2, 02:03:08
B        10.20.42.4/30 [200/0] via 10.100.1.2, 02:03:08
B        10.52.200.4/30 [200/0] via 10.100.1.2, 02:03:08
B        10.52.250.12/30 [200/0] via 10.100.1.2, 02:03:08
B        10.100.254.4/30 [200/0] via 10.100.1.2, 4d17h

R14#ping 10.100.0.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.0.15, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

#### R15

```
R15#show ip bgp
BGP table version is 20, local router ID is 10.100.0.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    10.100.254.6                  200      0 301 520 2042 i
 *>  10.20.42.4/30    10.100.254.6                  200      0 301 520 2042 i
 *>  10.52.200.4/30   10.100.254.6                  200      0 301 520 2042 i
 *>  10.52.250.12/30  10.100.254.6                  200      0 301 520 2042 i
 r>i 10.100.1.0/30    10.100.1.1               0    100      0 i
 r>i 10.100.1.4/30    10.100.1.1               0    100      0 i
 r>i 10.100.1.8/30    10.100.1.1              20    100      0 i
 r>i 10.100.1.24/30   10.100.1.1               0    100      0 i
 r>i 10.100.1.28/30   10.100.1.1              20    100      0 i
 *>i 10.100.254.0/30  10.100.1.1               0    100      0 i
 *>  10.100.254.4/30  0.0.0.0                  0         32768 i

R15#show ip route bgp
*Oct 20 11:24:09.508: %SYS-5-CONFIG_I: Configured from console by console
R15#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.100.254.6 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 17 subnets, 2 masks
B        10.20.42.0/30 [20/0] via 10.100.254.6, 02:04:05
B        10.20.42.4/30 [20/0] via 10.100.254.6, 02:04:05
B        10.52.200.4/30 [20/0] via 10.100.254.6, 02:04:05
B        10.52.250.12/30 [20/0] via 10.100.254.6, 02:04:05
B        10.100.254.0/30 [200/0] via 10.100.1.1, 4d17h
```

### Триада - AS520

#### R23 (RR)

```
R23#show ip bgp summary
BGP router identifier 10.23.23.23, local AS number 520
BGP table version is 20, main routing table version 20
11 network entries using 1540 bytes of memory
22 path entries using 1760 bytes of memory
3/2 BGP path/bestpath attribute entries using 432 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3804 total bytes of memory
BGP activity 12/1 prefixes, 25/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.24.24.24     4          520    6612    6620       20    0    0 4d04h          11
10.26.26.26     4          520    6634    6634       20    0    0 4d04h           4
172.16.101.5    4          101    7496    7486       20    0    0 4d17h           7

R23#show ip bgp
BGP table version is 20, local router ID is 10.23.23.23
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.20.42.0/30    10.24.24.24              0    100      0 2042 i
 * i                  10.26.26.26              0    100      0 2042 i
 *>i 10.20.42.4/30    10.24.24.24              0    100      0 2042 i
 * i                  10.26.26.26              0    100      0 2042 i
 *>i 10.52.200.4/30   10.24.24.24              0    100      0 2042 i
 * i                  10.26.26.26              0    100      0 2042 i
 *>i 10.52.250.12/30  10.24.24.24              0    100      0 2042 i
 * i                  10.26.26.26              0    100      0 2042 i
 * i 10.100.1.0/30    10.24.24.24              0    100      0 301 1001 i
 *>                   172.16.101.5                           0 101 1001 i
 * i 10.100.1.4/30    10.24.24.24              0    100      0 301 1001 i
 *>                   172.16.101.5                           0 101 1001 i
 * i 10.100.1.8/30    10.24.24.24              0    100      0 301 1001 i
 *>                   172.16.101.5                           0 101 1001 i
          
R23#show ip route isis
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 21 subnets, 2 masks
i L2     10.24.24.24/32 [115/20] via 10.52.1.2, 4d17h, Ethernet0/2
i L2     10.25.25.25/32 [115/20] via 10.52.1.10, 4d17h, Ethernet0/1
i L2     10.26.26.26/32 [115/30] via 10.52.1.10, 4d17h, Ethernet0/1
                        [115/30] via 10.52.1.2, 4d17h, Ethernet0/2
i L2     10.52.1.4/30 [115/20] via 10.52.1.10, 4d17h, Ethernet0/1
i L2     10.52.1.12/30 [115/20] via 10.52.1.2, 4d17h, Ethernet0/2

R23#ping 10.24.24.24
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.24.24.24, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R23#ping 10.26.26.26
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.26.26.26, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
```

#### R24

```
R24#show ip bgp summary
BGP router identifier 10.24.24.24, local AS number 520
BGP table version is 16, main routing table version 16
11 network entries using 1540 bytes of memory
18 path entries using 1440 bytes of memory
3/2 BGP path/bestpath attribute entries using 432 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3484 total bytes of memory
BGP activity 12/1 prefixes, 24/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.23.23.23     4          520    6621    6613       16    0    0 4d04h           7
10.52.200.6     4         2042    7495    7485       16    0    0 4d17h           4
172.16.30.6     4          301    7488    7481       16    0    0 4d17h           7

R24#show ip bgp
BGP table version is 16, local router ID is 10.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    10.52.200.6              0             0 2042 i
 *>  10.20.42.4/30    10.52.200.6              0             0 2042 i
 r>  10.52.200.4/30   10.52.200.6              0             0 2042 i
 *>  10.52.250.12/30  10.52.200.6              0             0 2042 i
 * i 10.100.1.0/30    10.23.23.23              0    100      0 101 1001 i
 *>                   172.16.30.6                            0 301 1001 i
 * i 10.100.1.4/30    10.23.23.23              0    100      0 101 1001 i
 *>                   172.16.30.6                            0 301 1001 i
 * i 10.100.1.8/30    10.23.23.23              0    100      0 101 1001 i
 *>                   172.16.30.6                            0 301 1001 i
 * i 10.100.1.24/30   10.23.23.23              0    100      0 101 1001 i
 *>                   172.16.30.6                            0 301 1001 i
 * i 10.100.1.28/30   10.23.23.23              0    100      0 101 1001 i
 *>                   172.16.30.6                            0 301 1001 i
          
R24#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 22 subnets, 2 masks
B        10.20.42.0/30 [20/0] via 10.52.200.6, 4d17h
B        10.20.42.4/30 [20/0] via 10.52.200.6, 4d17h
B        10.52.250.12/30 [20/0] via 10.52.200.6, 4d17h
B        10.100.1.0/30 [20/0] via 172.16.30.6, 4d17h
B        10.100.1.4/30 [20/0] via 172.16.30.6, 4d17h
B        10.100.1.8/30 [20/0] via 172.16.30.6, 4d17h
B        10.100.1.24/30 [20/0] via 172.16.30.6, 4d17h
B        10.100.1.28/30 [20/0] via 172.16.30.6, 4d17h
B        10.100.254.0/30 [20/0] via 172.16.30.6, 4d17h

R24#ping 10.23.23.23
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.23.23, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/7/19 ms
```

#### R26

```
R26#show ip bgp summary
BGP router identifier 10.26.26.26, local AS number 520
BGP table version is 16, main routing table version 16
11 network entries using 1540 bytes of memory
15 path entries using 1200 bytes of memory
3/2 BGP path/bestpath attribute entries using 432 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3244 total bytes of memory
BGP activity 12/1 prefixes, 17/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.23.23.23     4          520    6645    6645       16    0    0 4d04h          11
10.52.250.14    4         2042    7492    7488       16    0    0 4d17h           4

R26#show ip bgp
BGP table version is 16, local router ID is 10.26.26.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.20.42.0/30    10.24.24.24              0    100      0 2042 i
 *>                   10.52.250.14             0             0 2042 i
 * i 10.20.42.4/30    10.24.24.24              0    100      0 2042 i
 *>                   10.52.250.14             0             0 2042 i
 * i 10.52.200.4/30   10.24.24.24              0    100      0 2042 i
 *>                   10.52.250.14             0             0 2042 i
 r i 10.52.250.12/30  10.24.24.24              0    100      0 2042 i
 r>                   10.52.250.14             0             0 2042 i
 *>i 10.100.1.0/30    10.23.23.23              0    100      0 101 1001 i
 *>i 10.100.1.4/30    10.23.23.23              0    100      0 101 1001 i
 *>i 10.100.1.8/30    10.23.23.23              0    100      0 101 1001 i
 *>i 10.100.1.24/30   10.23.23.23              0    100      0 101 1001 i
 *>i 10.100.1.28/30   10.23.23.23              0    100      0 101 1001 i
 *>i 10.100.254.0/30  10.23.23.23              0    100      0 101 1001 i

R26#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 24 subnets, 2 masks
B        10.20.42.0/30 [20/0] via 10.52.250.14, 4d17h
B        10.20.42.4/30 [20/0] via 10.52.250.14, 4d17h
B        10.52.200.4/30 [20/0] via 10.52.250.14, 4d17h
B        10.100.1.0/30 [200/0] via 10.23.23.23, 4d04h
B        10.100.1.4/30 [200/0] via 10.23.23.23, 4d04h
B        10.100.1.8/30 [200/0] via 10.23.23.23, 4d04h
B        10.100.1.24/30 [200/0] via 10.23.23.23, 4d04h
B        10.100.1.28/30 [200/0] via 10.23.23.23, 4d04h
B        10.100.254.0/30 [200/0] via 10.23.23.23, 4d04h

R26#ping 10.23.23.23
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.23.23, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
```

### Санкт-Петербург - AS2042

#### R18

```
R18#show ip bgp summary
BGP router identifier 10.20.42.18, local AS number 2042
BGP table version is 26, main routing table version 26
11 network entries using 1540 bytes of memory
18 path entries using 1440 bytes of memory
7 multipath network entries and 14 multipath paths
3/2 BGP path/bestpath attribute entries using 432 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3460 total bytes of memory
BGP activity 12/1 prefixes, 20/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.52.200.5     4          520    7630    7641       26    0    0 4d19h           7
10.52.250.13    4          520    7624    7627       26    0    0 4d19h           7

R18#show ip bgp
BGP table version is 26, local router ID is 10.20.42.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    0.0.0.0                  0         32768 i
 *>  10.20.42.4/30    0.0.0.0                  0         32768 i
 *>  10.52.200.4/30   0.0.0.0                  0         32768 i
 *>  10.52.250.12/30  0.0.0.0                  0         32768 i
 *m  10.100.1.0/30    10.52.250.13                           0 520 101 1001 i
 *>                   10.52.200.5                            0 520 301 1001 i
 *m  10.100.1.4/30    10.52.250.13                           0 520 101 1001 i
 *>                   10.52.200.5                            0 520 301 1001 i
 *m  10.100.1.8/30    10.52.250.13                           0 520 101 1001 i
 *>                   10.52.200.5                            0 520 301 1001 i
 *m  10.100.1.24/30   10.52.250.13                           0 520 101 1001 i
 *>                   10.52.200.5                            0 520 301 1001 i
 *m  10.100.1.28/30   10.52.250.13                           0 520 101 1001 i
 *>                   10.52.200.5                            0 520 301 1001 i

R18#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.52.200.5 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 20 subnets, 2 masks
B        10.100.1.0/30 [20/0] via 10.52.250.13, 00:01:37
                       [20/0] via 10.52.200.5, 00:01:37
B        10.100.1.4/30 [20/0] via 10.52.250.13, 00:01:37
                       [20/0] via 10.52.200.5, 00:01:37
B        10.100.1.8/30 [20/0] via 10.52.250.13, 00:01:37
                       [20/0] via 10.52.200.5, 00:01:37
B        10.100.1.24/30 [20/0] via 10.52.250.13, 00:01:37
                        [20/0] via 10.52.200.5, 00:01:37
B        10.100.1.28/30 [20/0] via 10.52.250.13, 00:01:37

R18#show ip route 10.100.254.0
Routing entry for 10.100.254.0/30
  Known via "bgp 2042", distance 20, metric 0
  Tag 520, type external
  Last update from 10.52.200.5 00:01:37 ago
  Routing Descriptor Blocks:
  * 10.52.250.13, from 10.52.250.13, 00:01:37 ago
      Route metric is 0, traffic share count is 1
      AS Hops 3
      Route tag 520
      MPLS label: none
    10.52.200.5, from 10.52.200.5, 00:01:37 ago
      Route metric is 0, traffic share count is 1
      AS Hops 3
      Route tag 520
      MPLS label: none

R18#show ip cef
Prefix               Next Hop             Interface
0.0.0.0/0            10.52.200.5          Ethernet0/2
0.0.0.0/8            drop
0.0.0.0/32           receive              
10.20.20.0/30        10.20.42.2           Ethernet0/1
                     10.20.42.6           Ethernet0/0
10.20.42.0/30        attached             Ethernet0/1
10.20.42.0/32        receive              Ethernet0/1
10.20.42.1/32        receive              Ethernet0/1
10.20.42.2/32        attached             Ethernet0/1
10.20.42.3/32        receive              Ethernet0/1
10.20.42.4/30        attached             Ethernet0/0
10.20.42.4/32        receive              Ethernet0/0
10.20.42.5/32        receive              Ethernet0/0
10.20.42.6/32        attached             Ethernet0/0
10.20.42.7/32        receive              Ethernet0/0
10.20.42.8/30        10.20.42.6           Ethernet0/0
10.20.42.16/32       10.20.42.6           Ethernet0/0
10.20.42.17/32       10.20.42.2           Ethernet0/1
10.20.42.18/32       receive              Loopback0
10.52.200.4/30       attached             Ethernet0/2
10.52.200.4/32       receive              Ethernet0/2
10.52.200.5/32       attached             Ethernet0/2
10.52.200.6/32       receive              Ethernet0/2
10.52.200.7/32       receive              Ethernet0/2
10.52.250.12/30      attached             Ethernet0/3
10.52.250.12/32      receive              Ethernet0/3
10.52.250.13/32      attached             Ethernet0/3
10.52.250.14/32      receive              Ethernet0/3
10.52.250.15/32      receive              Ethernet0/3
10.100.1.0/30        10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.1.4/30        10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.1.8/30        10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.1.24/30       10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.1.28/30       10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.254.0/30      10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
10.100.254.4/30      10.52.200.5          Ethernet0/2
                     10.52.250.13         Ethernet0/3
Prefix               Next Hop             Interface
127.0.0.0/8          drop
224.0.0.0/4          drop
224.0.0.0/24         receive              
240.0.0.0/4          drop
255.255.255.255/32   receive

R18#ping 10.52.200.5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.52.200.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R18#ping 10.52.250.13
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.52.250.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```
