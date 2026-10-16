# BGP. Основы

## Цель:

настроить BGP между автономными системами и организовать доступность между офисами Москва и С.-Петербург.

## Задание:

1. Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроите eBGP между провайдерами Киторн и Ламас.
3. Настроите eBGP между Ламас и Триада.
4. Настроите eBGP между офисом С.-Петербург и провайдером Триада.
5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.
6. План работы и изменения зафиксированы в документации.

<img width="1246" height="642" alt="Pasted image 20261011003951" src="https://github.com/user-attachments/assets/64e77991-1505-48e7-bc94-33efd31a4f08" />

## Адресация BGP-сосоедств

| Роутер | AS   | BGP-сосед | AS соседа | IP соседа     |
| ------ | ---- | --------- | --------- | ------------- |
| R14    | 1001 | R22       | 101       | 10.100.254.2  |
| R15    | 1001 | R21       | 301       | 10.100.254.6  |
| R22    | 101  | R14       | 1001      | 10.100.254.1  |
| R22    | 101  | R21       | 301       | 172.16.101.10 |
| R22    | 101  | R23       | 520       | 172.16.101.6  |
| R21    | 301  | R15       | 1001      | 10.100.254.5  |
| R21    | 301  | R22       | 101       | 172.16.101.9  |
| R21    | 301  | R24       | 520       | 172.16.30.5   |
| R23    | 520  | R22       | 101       | 172.16.101.5  |
| R24    | 520  | R21       | 301       | 172.16.30.6   |
| R24    | 520  | R18       | 2042      | 10.52.200.6   |
| R26    | 520  | R18       | 2042      | 10.52.250.14  |
| R18    | 2042 | R24       | 520       | 10.52.200.5   |
| R18    | 2042 | R26       | 520       | 10.52.250.13  |


## Настройка маршрутизаторов

### Москва - AS1001

#### R14

```
router bgp 1001
 bgp log-neighbor-changes
!
 neighbor 10.100.254.2 remote-as 101
!
 network 10.100.1.0 mask 255.255.255.252
 network 10.100.1.4 mask 255.255.255.252
 network 10.100.1.24 mask 255.255.255.252
 network 10.100.254.0 mask 255.255.255.252
```

#### R15

```
router bgp 1001
 bgp log-neighbor-changes
!
 neighbor 10.100.254.6 remote-as 301
!
 network 10.100.1.8 mask 255.255.255.252
 network 10.100.1.12 mask 255.255.255.252
 network 10.100.1.28 mask 255.255.255.252
 network 10.100.254.4 mask 255.255.255.252
```

### Киторн - AS101

#### R22

```
router bgp 101
 bgp log-neighbor-changes
!
 neighbor 10.100.254.1 remote-as 1001
 neighbor 172.16.101.10 remote-as 301
 neighbor 172.16.101.6 remote-as 520
```

### Ламас - AS301

#### R21

```
router bgp 301
 bgp log-neighbor-changes
!
 neighbor 10.100.254.5 remote-as 1001
 neighbor 172.16.101.9 remote-as 101
 neighbor 172.16.30.5 remote-as 520
```

### Триада - AS520

#### R23

```
router bgp 520
 bgp log-neighbor-changes
!
 neighbor 172.16.101.5 remote-as 101
!
 neighbor 10.24.24.24 remote-as 520
 neighbor 10.24.24.24 update-source Loopback0
 neighbor 10.24.24.24 route-reflector-client
 neighbor 10.24.24.24 next-hop-self
!
 neighbor 10.26.26.26 remote-as 520
 neighbor 10.26.26.26 update-source Loopback0
 neighbor 10.26.26.26 route-reflector-client
 neighbor 10.26.26.26 next-hop-self
```

#### R24

```
router bgp 520
 bgp log-neighbor-changes
!
 neighbor 172.16.30.6 remote-as 301
 neighbor 10.52.200.6 remote-as 2042
!
 neighbor 10.23.23.23 remote-as 520
 neighbor 10.23.23.23 update-source Loopback0
 neighbor 10.23.23.23 next-hop-self
```

#### R26

```
router bgp 520
 bgp log-neighbor-changes
!
 no neighbor 10.52.0.23
 neighbor 10.52.250.14 remote-as 2042
!
 neighbor 10.23.23.23 remote-as 520
 neighbor 10.23.23.23 update-source Loopback0
 neighbor 10.23.23.23 next-hop-self
```

### Санкт-Петербург - AS2042

#### R18

```
router bgp 2042
 bgp log-neighbor-changes
!
 network 10.20.42.0 mask 255.255.255.252
 network 10.20.42.4 mask 255.255.255.252
 network 10.52.200.4 mask 255.255.255.252
 network 10.52.250.12 mask 255.255.255.252
!
 network 10.20.42.18 mask 255.255.255.255
!
 neighbor 10.52.200.5 remote-as 520
 neighbor 10.52.250.13 remote-as 520
```

## Диагностика

```
show ip bgp summary
show ip bgp
show ip route bgp
```

#### R14

```
R14#show ip bgp summary
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.1.2      4         1001    1365    1367       21    0    0 20:36:30        5
10.100.254.2    4          101    1369    1366       21    0    0 20:36:27        4

R14#show ip bgp
     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.20.42.0/30    10.100.1.2               0    100      0 301 520 2042 i
 *>                   10.100.254.2                           0 101 520 2042 i
 * i 10.20.42.4/30    10.100.1.2               0    100      0 301 520 2042 i
 *>                   10.100.254.2                           0 101 520 2042 i
 * i 10.52.200.4/30   10.100.1.2               0    100      0 301 520 2042 i
 *>                   10.100.254.2                           0 101 520 2042 i
 * i 10.52.250.12/30  10.100.1.2               0    100      0 301 520 2042 i
 *>                   10.100.254.2                           0 101 520 2042 i
 *>  10.100.1.0/30    0.0.0.0                  0         32768 i
 *>  10.100.1.4/30    0.0.0.0                  0         32768 i
 *>  10.100.1.8/30    10.100.1.6              30         32768 i
 *>  10.100.1.12/30   10.100.1.6              20         32768 i
 *>  10.100.1.24/30   0.0.0.0                  0         32768 i
 *>  10.100.1.28/30   10.100.1.2              20         32768 i
          
R14#show ip route bgp
Gateway of last resort is 10.100.254.2 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 20 subnets, 2 masks
B        10.20.42.0/30 [20/0] via 10.100.254.2, 07:36:50
B        10.20.42.4/30 [20/0] via 10.100.254.2, 07:36:50
B        10.52.200.4/30 [20/0] via 10.100.254.2, 07:36:50
B        10.52.250.12/30 [20/0] via 10.100.254.2, 07:36:50
B        10.100.254.4/30 [200/0] via 10.100.1.2, 20:35:34

R14#ping 10.52.200.6 source 10.100.254.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.52.200.6, timeout is 2 seconds:
Packet sent with a source address of 10.100.254.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/11/27 ms
```

#### R15

```

```

#### R22

```

```

#### R23

```

```

#### R24

```

```

#### R26

```

```

#### R18

```

```
