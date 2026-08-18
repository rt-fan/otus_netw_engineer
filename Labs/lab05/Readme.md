# EIGRP. Настройка EIGRP named-mode для IPv4 и IPv6 в офисе С.-Петербург.

## Задание:
1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.
5. Настройка осуществляется одновременно для IPv4 и IPv6.

## Адресация IPv4

Используем /30 для линков между маршрутизаторами.

| Линк       | Сеть            | Адреса                                         |
| ---------- | --------------- | ---------------------------------------------- |
| R18 — R16  | 10.20.42.4/30   | R18 (0/0) 10.20.42.5, R16 (0/1) 10.20.42.6     |
| R18 — R17  | 10.20.42.0/30   | R18 (0/1) 10.20.42.1, R17 (0/1) 10.20.42.2     |
| R18 — R24  | 10.52.200.4/30  | R18 (0/2) 10.52.200.6, R24 (0/3) 10.52.200.5   |
| R18 — R26  | 10.52.250.12/30 | R18 (0/3) 10.52.250.14, R26 (0/3) 10.52.250.13 |
| R17 — SW9  |                 | R17 (0/0) -> SW9 (0/3)                         |
| R17 — SW10 |                 | R17 (0/2) -> SW10 (1/0)                        |
| R16 — R32  | 10.20.42.10/30  | R16 (0/3) 10.20.42.11, R32 (0/0) 10.20.42.12   |
| R16 — SW9  | /30             | R16 (0/0) -> SW10 (0/3)                        |
| R16 — SW10 | /30             | R16 (0/2) -> SW9 (1/0)                         |

Loopback'и:

| Router | Loopback       |
| ------ | -------------- |
| R16    | 10.20.42.16/32 |
| R17    | 10.20.42.17/32 |
| R18    | 10.20.42.18/32 |
| R32    | 10.20.42.32/32 |

## Настройка маршрутизаторов

#### R18

```
ipv6 unicast-routing

interface Ethernet0/0
 ip address 10.20.42.5 255.255.255.252
 ipv6 address 2001:DB8:20:42:4::1/64
 no shutdown

interface Ethernet0/1
 ip address 10.20.42.1 255.255.255.252
 ipv6 address 2001:DB8:20:42:0::1/64
 no shutdown

interface Ethernet0/2
 ip address 10.52.200.6 255.255.255.252
 ipv6 address 2001:DB8:52:200::2/64
 no shutdown

interface Ethernet0/3
 ip address 10.52.250.14 255.255.255.252
 ipv6 address 2001:DB8:52:250::2/64
 no shutdown

interface Loopback0
 ip address 10.20.42.18 255.255.255.255
 ipv6 address 2001:DB8:20:42::18/128

router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 100
  topology base

  network 10.20.42.0 0.0.0.255
  network 10.52.200.4 0.0.0.3
  network 10.52.250.12 0.0.0.3

  af-interface default
   no passive-interface
  exit-af-interface
 exit-address-family

 !
 address-family ipv6 unicast autonomous-system 100
  topology base

  af-interface default
   no shutdown
  exit-af-interface

  af-interface Ethernet0/0
  exit-af-interface
  af-interface Ethernet0/1
  exit-af-interface
  af-interface Ethernet0/2
  exit-af-interface
  af-interface Ethernet0/3
  exit-af-interface
  af-interface Loopback0
  exit-af-interface
 exit-address-family

interface Ethernet0/0
 ipv6 eigrp 100

interface Ethernet0/1
 ipv6 eigrp 100

interface Ethernet0/2
 ipv6 eigrp 100

interface Ethernet0/3
 ipv6 eigrp 100

interface Loopback0
 ipv6 eigrp 100
```

#### R16

```
ipv6 unicast-routing

interface Ethernet0/1
 ip address 10.20.42.6 255.255.255.252
 ipv6 address 2001:DB8:20:42:4::2/64

interface Ethernet0/3
 ip address 10.20.42.11 255.255.255.252
 ipv6 address 2001:DB8:20:42:10::1/64

interface Loopback0
 ip address 10.20.42.16 255.255.255.255
 ipv6 address 2001:DB8:20:42::16/128

router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 100
  topology base

  network 10.20.42.4 0.0.0.3
  network 10.20.42.10 0.0.0.3
  network 10.20.42.16 0.0.0.0

  af-interface default
   no passive-interface
  exit-af-interface

 exit-address-family

 !
 address-family ipv6 unicast autonomous-system 100
  topology base

  af-interface default
   no shutdown
  exit-af-interface
 exit-address-family

interface Ethernet0/1
 ip summary-address eigrp 100 10.20.42.16 255.255.255.240
 ipv6 summary-address eigrp 100 2001:DB8:20:42::/124
 ipv6 eigrp 100

interface Ethernet0/3
 ipv6 eigrp 100

interface Loopback0
 ipv6 eigrp 100
```

#### R17

```
ipv6 unicast-routing

interface Ethernet0/1
 ip address 10.20.42.2 255.255.255.252
 ipv6 address 2001:DB8:20:42:0::2/64

interface Loopback0
 ip address 10.20.42.17 255.255.255.255
 ipv6 address 2001:DB8:20:42::17/128

router eigrp SPB

 address-family ipv4 unicast autonomous-system 100
  topology base

  network 10.20.42.0 0.0.0.3
  network 10.20.42.17 0.0.0.0

  af-interface default
   no passive-interface
  exit-af-interface

 exit-address-family

 address-family ipv6 unicast autonomous-system 100
  topology base

  af-interface default
   no shutdown
  exit-af-interface

 exit-address-family

interface Ethernet0/1
 ip summary-address eigrp 100 10.20.42.16 255.255.255.240
 ipv6 summary-address eigrp 100 2001:DB8:20:42::/124
 ipv6 eigrp 100

interface Loopback0
 ipv6 eigrp 100
```

#### R32

```
ipv6 unicast-routing

interface Ethernet0/0
 ip address 10.20.42.12 255.255.255.252
 ipv6 address 2001:DB8:20:42:10::2/64

interface Loopback0
 ip address 10.20.42.32 255.255.255.255
 ipv6 address 2001:DB8:20:42::32/128

router eigrp SPB

 address-family ipv4 unicast autonomous-system 100
  topology base

  network 10.20.42.10 0.0.0.3

  af-interface default
   no passive-interface
  exit-af-interface

 exit-address-family

 address-family ipv6 unicast autonomous-system 100
  topology base

  af-interface default
   no shutdown
  exit-af-interface

 exit-address-family

interface Ethernet0/0
 ipv6 eigrp 100
```

#### Чтобы R32 видел только маршрут по умолчанию

На R16 создаем статический маршрут по умолчанию и распространяем его в EIGRP:

```
ip route 0.0.0.0 0.0.0.0 10.20.42.5

ipv6 route ::/0 2001:DB8:20:42:4::1

router eigrp SPB
 address-family ipv4 unicast autonomous-system 100
  redistribute static
 exit-address-family

 address-family ipv6 unicast autonomous-system 100
  redistribute static
 exit-address-family
```

Чтобы R32 не получал другие маршруты, на интерфейсе R16 в сторону R32 применяется входящий distribute-list с префикс-листом, разрешающим только маршрут по умолчанию:

```
ip prefix-list DEFAULT-ONLY seq 5 permit 0.0.0.0/0

router eigrp SPB
 address-family ipv4 unicast autonomous-system 100
  af-interface Ethernet0/3
   distribute-list prefix DEFAULT-ONLY out
  exit-af-interface
 exit-address-family
```

Для IPv6 аналогично:

```
ipv6 prefix-list DEFAULT-ONLY seq 5 permit ::/0

router eigrp SPB
 address-family ipv6 unicast autonomous-system 100
  af-interface Ethernet0/3
   distribute-list prefix-list DEFAULT-ONLY out
  exit-af-interface
 exit-address-family
```

## Проверка

На R32 после настройки должны остаться только маршруты по умолчанию, полученные по EIGRP:

```
show ip route eigrp
show ipv6 route eigrp
```

Ожидаемый результат:

- IPv4: только `D*EX 0.0.0.0/0`
- IPv6: только `D ::/0`


























