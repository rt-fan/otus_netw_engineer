# BGP. Фильтрация

## Цель:

настроить фильтрацию в офисах Москва и С.-Петербург.

## Задание:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
5. Все сети в лабораторной работе должны иметь IP связность.

<img width="1246" height="642" alt="Pasted image 20261011003951" src="https://github.com/user-attachments/assets/64e77991-1505-48e7-bc94-33efd31a4f08" />

## Настройка маршрутизаторов

### Москва - AS1001

#### R14

```
ip as-path access-list 10 permit ^$
!
router bgp 1001
 neighbor 10.100.254.2 filter-list 10 out
!
ip prefix-list KITORNE-IN seq 10 permit 0.0.0.0/0
!
router bgp 1001
 neighbor 10.100.254.2 prefix-list KITORNE-IN in
```

#### R15

```
ip as-path access-list 10 permit ^$
!
router bgp 1001
 neighbor 10.100.254.6 filter-list 10 out
!
ip prefix-list LAMAS-IN seq 10 permit 0.0.0.0/0
ip prefix-list LAMAS-IN seq 20 permit 10.20.42.0/30
ip prefix-list LAMAS-IN seq 30 permit 10.20.42.4/30
!
router bgp 1001
 neighbor 10.100.254.6 prefix-list LAMAS-IN in
```

### Ламас - AS301

#### R21

```
ip prefix-list LAMAS-OUT seq 10 permit 0.0.0.0/0
ip prefix-list LAMAS-OUT seq 20 permit 10.20.42.0/30
ip prefix-list LAMAS-OUT seq 30 permit 10.20.42.4/30
!
router bgp 301
 neighbor 10.100.254.5 prefix-list LAMAS-OUT out
 neighbor 10.100.254.5 default-originate
```

### Киторн - AS101

#### R22

```
ip prefix-list KITORNE-OUT seq 10 permit 0.0.0.0/0
!
router bgp 101
 neighbor 10.100.254.1 prefix-list KITORNE-OUT out
 neighbor 10.100.254.1 default-originate
```

### Санкт-Петербург - AS2042

#### R18

```
ip prefix-list SPB-OUT seq 10 permit 10.20.42.0/30
ip prefix-list SPB-OUT seq 20 permit 10.20.42.4/30
!
router bgp 2042
 neighbor 10.52.200.5 prefix-list SPB-OUT out
 neighbor 10.52.250.13 prefix-list SPB-OUT out
```

## Диагностика

```
show ip bgp summary
```

### Москва - AS1001

#### R14

```
show ip bgp summary
show ip bgp neighbors 10.100.254.2 received-routes
show ip bgp


R14#show ip bgp summary
BGP router identifier 10.100.0.14, local AS number 1001
BGP table version is 37, main routing table version 37
10 network entries using 1400 bytes of memory
11 path entries using 880 bytes of memory
7/5 BGP path/bestpath attribute entries using 1008 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
6 BGP filter-list cache entries using 96 bytes of memory
BGP using 3480 total bytes of memory
BGP activity 13/3 prefixes, 20/9 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.1.2      4         1001    7740    7746       37    0    0 4d21h           4
10.100.254.2    4          101    7753    7743       37    0    0 4d21h           1


R14#show ip bgp neighbors 10.100.254.2 received-routes
% Inbound soft reconfiguration not enabled on 10.100.254.2


R14#show ip bgp
BGP table version is 37, local router ID is 10.100.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r   0.0.0.0          10.100.254.2                           0 101 i
 r>i                  10.100.1.2               0    200      0 301 i
 *>i 10.20.42.0/30    10.100.1.2               0    200      0 301 520 2042 i
 *>i 10.20.42.4/30    10.100.1.2               0    200      0 301 520 2042 i
 *>  10.100.1.0/30    0.0.0.0                  0         32768 i
 *>  10.100.1.4/30    0.0.0.0                  0         32768 i
 *>  10.100.1.8/30    10.100.1.2              20         32768 i
 *>  10.100.1.24/30   0.0.0.0                  0         32768 i
 *>  10.100.1.28/30   10.100.1.2              20         32768 i
 *>  10.100.254.0/30  0.0.0.0                  0         32768 i
 *>i 10.100.254.4/30  10.100.1.2               0    100      0 i
```

#### R15

```
show ip bgp summary
show ip bgp
show ip route 10.20.42.0
show ip route 10.20.42.4
show ip route 0.0.0.0


R15#show ip bgp summary
BGP router identifier 10.100.0.15, local AS number 1001
BGP table version is 27, main routing table version 27
10 network entries using 1400 bytes of memory
10 path entries using 800 bytes of memory
8/5 BGP path/bestpath attribute entries using 1152 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
6 BGP filter-list cache entries using 96 bytes of memory
BGP using 3520 total bytes of memory
BGP activity 13/3 prefixes, 19/9 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.1.1      4         1001    7748    7742       27    0    0 4d21h           6
10.100.254.6    4          301    7745    7753       27    0    0 4d21h           3


R15#show ip bgp
BGP table version is 27, local router ID is 10.100.0.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          10.100.254.6                  200      0 301 i
 *>  10.20.42.0/30    10.100.254.6                  200      0 301 520 2042 i
 *>  10.20.42.4/30    10.100.254.6                  200      0 301 520 2042 i
 r>i 10.100.1.0/30    10.100.1.1               0    100      0 i
 r>i 10.100.1.4/30    10.100.1.1               0    100      0 i
 r>i 10.100.1.8/30    10.100.1.1              20    100      0 i
 r>i 10.100.1.24/30   10.100.1.1               0    100      0 i
 r>i 10.100.1.28/30   10.100.1.1              20    100      0 i
 *>i 10.100.254.0/30  10.100.1.1               0    100      0 i
 *>  10.100.254.4/30  0.0.0.0                  0         32768 i


R15#show ip route 10.20.42.0
Routing entry for 10.20.42.0/30
  Known via "bgp 1001", distance 20, metric 0
  Tag 301, type external
  Last update from 10.100.254.6 05:46:13 ago
  Routing Descriptor Blocks:
  * 10.100.254.6, from 10.100.254.6, 05:46:13 ago
      Route metric is 0, traffic share count is 1
      AS Hops 3
      Route tag 301
      MPLS label: none


R15#show ip route 10.20.42.4
Routing entry for 10.20.42.4/30
  Known via "bgp 1001", distance 20, metric 0
  Tag 301, type external
  Last update from 10.100.254.6 05:46:13 ago
  Routing Descriptor Blocks:
  * 10.100.254.6, from 10.100.254.6, 05:46:13 ago
      Route metric is 0, traffic share count is 1
      AS Hops 3
      Route tag 301
      MPLS label: none


R15#show ip route 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 10.100.254.6
      Route metric is 0, traffic share count is 1
```

### Ламас - AS301

#### R21

```
show ip bgp neighbors 10.100.254.5 advertised-routes


R21#show ip bgp neighbors 10.100.254.5 advertised-routes
BGP table version is 21, local router ID is 172.16.30.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

Originating default network 0.0.0.0

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    172.16.30.5                            0 520 2042 i
 *>  10.20.42.4/30    172.16.30.5                            0 520 2042 i

Total number of prefixes 2


R21#show ip bgp summary
BGP router identifier 172.16.30.1, local AS number 301
BGP table version is 21, main routing table version 21
10 network entries using 1400 bytes of memory
19 path entries using 1520 bytes of memory
6/3 BGP path/bestpath attribute entries using 864 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3880 total bytes of memory
BGP activity 13/3 prefixes, 26/7 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.254.5    4         1001    7758    7750       21    0    0 4d21h           7
172.16.30.5     4          520    7738    7746       21    0    0 4d21h           2
172.16.101.9    4          101    7750    7760       21    0    0 4d21h           9
```

### Киторн - AS101
#### R22

```
show ip bgp neighbors 10.100.254.1 advertised-routes


R22#show ip bgp neighbors 10.100.254.1 advertised-routes
BGP table version is 24, local router ID is 172.16.101.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

Originating default network 0.0.0.0

     Network          Next Hop            Metric LocPrf Weight Path

Total number of prefixes 0


R22#show ip bgp summary
BGP router identifier 172.16.101.1, local AS number 101
BGP table version is 24, main routing table version 24
10 network entries using 1400 bytes of memory
19 path entries using 1520 bytes of memory
7/4 BGP path/bestpath attribute entries using 1008 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4024 total bytes of memory
BGP activity 13/3 prefixes, 33/14 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.254.1    4         1001    7750    7759       24    0    0 4d21h           7
172.16.101.6    4          520    7743    7752       24    0    0 4d21h           2
172.16.101.10   4          301    7759    7750       24    0    0 4d21h           9
```

### Санкт-Петербург - AS2042

#### R18

```
show ip bgp neighbors 10.52.200.5 advertised-routes
show ip bgp neighbors 10.52.250.13 advertised-routes
ping 10.100.254.1 source 10.20.42.1
ping 10.100.254.5 source 10.20.42.1


R18#show ip bgp neighbors 10.52.200.5 advertised-routes
BGP table version is 26, local router ID is 10.20.42.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    0.0.0.0                  0         32768 i
 *>  10.20.42.4/30    0.0.0.0                  0         32768 i

Total number of prefixes 2 


R18#show ip bgp neighbors 10.52.250.13 advertised-routes
BGP table version is 26, local router ID is 10.20.42.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.42.0/30    0.0.0.0                  0         32768 i
 *>  10.20.42.4/30    0.0.0.0                  0         32768 i

Total number of prefixes 2 


R18#ping 10.100.254.1 source 10.20.42.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.254.1, timeout is 2 seconds:
Packet sent with a source address of 10.20.42.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/9/27 ms
R18#
R18#ping 10.100.254.5 source 10.20.42.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.254.5, timeout is 2 seconds:
Packet sent with a source address of 10.20.42.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/6/15 ms


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
10.52.200.5     4          520    7742    7754       26    0    0 4d21h           7
10.52.250.13    4          520    7735    7742       26    0    0 4d20h           7
```
