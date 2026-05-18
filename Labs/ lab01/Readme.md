# Архитектура сети

## Задание:

1. Разработать и задокументировать адресное пространство для лабораторного стенда;
2. Настроить ip-адреса на каждом активном порту;
3. Настроить каждый VPC в каждом офисе в своем VLAN;
4. Настроить VLAN/Loopbackup interface управления для сетевых устройств;
5. Настроить сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано;
6. Использовать IPv4. IPv6 по желанию.

## Решение:

### Адресация

Для маршрутизации и управления выделены следующие диапазоны:

- **AS 1001 (Москва):** `10.100.0.0/16`
- **AS 520 (Триада):** `10.52.0.0/16`
- **AS 2042 (С.-Петербург):** `10.20.42.0/24`
- **AS 101 (Киторн):** `172.16.101.0/24`
- **AS 301 (Ламас):** `172.16.30.0/24`
- **Лабытнанги:** `192.168.27.0/24`
- **Чокурдах:** `192.168.28.0/24`

<picture>
 <img src="image.png">
</picture>

### Таблица IP-адресации

#### Провайдеры: AS 101 (Киторн), AS 301 (Ламас) и AS 520 (Триада)

| Устройство | Интерфейс | IP Адрес      | Маска | Описание              |
| ---------- | --------- | ------------- | ----- | --------------------- |
| Ламас      |           |               |       |                       |
| **R21**    | Lo0       | 172.16.30.1   | /32   | Loopbackup int.       |
| **R21**    | e0/0      | 172.16.30.6   | /30   | R24 (AS 520)          |
| **R21**    | e0/1      | 172.16.101.10 | /30   | R22                   |
| **R21**    | e0/2      | 10.100.254.6  | /30   | R15 (AS 1001)         |
| Киторн     |           |               |       |                       |
| **R22**    | Lo0       | 172.16.101.1  | /32   | Loopbackup int.       |
| **R22**    | e0/0      | 10.100.254.2  | /30   | R14 (AS 1001)         |
| **R22**    | e0/1      | 172.16.101.9  | /30   | R21                   |
| **R22**    | e0/2      | 172.16.101.5  | /30   | R23 (AS 520)          |
| Триада     |           |               |       |                       |
| **R23**    | Lo0       | 10.52.0.23    | /32   | Loopbackup int.       |
| **R23**    | e0/0      | 172.16.101.6  | /30   | R22 (AS 101 - Киторн) |
| **R23**    | e0/1      | 10.52.1.9     | /30   | R25                   |
| **R23**    | e0/2      | 10.52.1.1     | /30   | R24                   |
| -          |           |               |       |                       |
| **R24**    | Lo0       | 10.52.0.24    | /32   | Loopbackup int.       |
| **R24**    | e0/0      | 172.16.30.5   | /30   | R21 (AS 301 - Ламас)  |
| **R24**    | e0/1      | 10.52.1.13    | /30   | R26                   |
| **R24**    | e0/2      | 10.52.1.2     | /30   | R23                   |
| **R24**    | e0/3      | 10.52.200.5   | /30   | R18 (AS 2042 - Питер) |
| -          |           |               |       |                       |
| **R25**    | Lo0       | 10.52.0.25    | /32   | Loopbackup int.       |
| **R25**    | e0/0      | 10.52.1.10    | /30   | R23                   |
| **R25**    | e0/1      | 10.52.250.1   | /30   | R27 (Лабытнанги)      |
| **R25**    | e0/2      | 10.52.1.5     | /30   | R26                   |
| **R25**    | e0/3      | 10.52.250.5   | /30   | R28 (Чокурдах)        |
| -          |           |               |       |                       |
| **R26**    | Lo0       | 10.52.0.26    | /32   | Loopbackup int.       |
| **R26**    | e0/0      | 10.52.1.14    | /30   | R24                   |
| **R26**    | e0/1      | 10.52.250.9   | /30   | R28 (Чокурдах)        |
| **R26**    | e0/2      | 10.52.1.6     | /30   | R25                   |
| **R26**    | e0/3      | 10.52.250.13  | /30   | R18 (AS 2042 - Питер) |

#### Лабытнанги

| Устройство | Интерфейс | IP Адрес     | Маска | Описание        |
| ---------- | --------- | ------------ | ----- | --------------- |
| **R27**    | Lo0       | 192.168.27.1 | /32   | Loopbackup int. |
| **R27**    | e0/0      | 10.52.250.2  | /30   | R25             |

#### AS1001 (Москва)

| Устройство | Интерфейс | IP Адрес     | Маска | Описание        |
| ---------- | --------- | ------------ | ----- | --------------- |
| **R14**    | Lo0       | 10.100.0.14  | /32   | Loopbackup int. |
| **R14**    | e0/0      | 10.100.1.5   | /30   | R12             |
| **R14**    | e0/1      | 10.100.1.1   | /30   | R13             |
| **R14**    | e0/2      | 10.100.254.1 | /30   | R22 (AS 101)    |
| **R14**    | e0/3      | 10.100.1.25  | /30   | R19             |
|            |           |              |       |                 |
| **R15**    | Lo0       | 10.100.0.15  | /32   | Loopbackup int. |
| **R15**    | e0/0      | 10.100.1.9   | /30   | R13             |
| **R15**    | e0/1      | 10.100.1.21  | /30   | R12             |
| **R15**    | e0/2      | 10.100.254.5 | /30   | R21 (AS 301)    |
| **R15**    | e0/3      | 10.100.1.29  | /30   | R20             |
|            |           |              |       |                 |
| **R12**    | Lo0       | 10.100.0.12  | /32   | Loopbackup int. |
| **R12**    | e0/2      | 10.100.1.6   | /30   | R14             |
| **R12**    | e0/3      | 10.100.1.22  | /30   | R15             |
| **R12**    | e0/0.10   | 10.100.10.1  | /24   |                 |
| **R12**    | e0/0.20   | 10.100.20.2  | /24   |                 |
| **R12**    | e0/0.99   | 10.100.99.12 | /24   |                 |
|            |           |              |       |                 |
| **R13**    | Lo0       | 10.100.0.13  | /32   | Loopbackup int. |
| **R13**    | e0/2      | 10.100.1.10  | /30   | R15             |
| **R13**    | e0/3      | 10.100.1.2   | /30   | R14             |
| **R13**    | e0/0.10   | 10.100.10.2  | /24   |                 |
| **R13**    | e0/0.20   | 10.100.20.1  | /24   |                 |
| **R13**    | e0/0.99   | 10.100.99.13 | /24   |                 |
|            |           |              |       |                 |
| **R19**    | Lo0       | 10.100.0.19  | /32   | Loopbackup int. |
| **R19**    | e0/0      | 10.100.1.26  | /30   | R14             |
|            |           |              |       |                 |
| **R20**    | Lo0       | 10.100.0.20  | /32   | Loopbackup int. |
| **R20**    | e0/0      | 10.100.1.30  | /30   | R15             |
|            |           |              |       |                 |
| **SW2**    | vlan 99   | 10.100.99.2  | /24   |                 |
| **SW3**    | vlan 99   | 10.100.99.3  | /24   |                 |
| **SW4**    | vlan 99   | 10.100.99.4  | /24   |                 |
| **SW4**    | e0/2      |              |       | Po1             |
| **SW4**    | e0/3      |              |       | Po1             |
| **SW5**    | vlan 99   | 10.100.99.5  | /24   |                 |
| **SW5**    | e0/2      |              |       | Po1             |
| **SW5**    | e0/3      |              |       | Po1             |
|            |           |              |       |                 |
| **VPC1**   | eth0      | 10.100.10.10 | /24   | SW3->R12        |
| **VPC7**   | eth0      | 10.100.20.10 | /24   | SW2->R13        |
   
#### AS 2042 (С.-Петербург)

| Устройство | Интерфейс | IP Адрес     | Маска | Описание        |
| :--------- | :-------- | :----------- | :---- | :-------------- |
| **R18**    | Lo0       | 10.20.42.18  | /32   | Loopbackup int. |
| **R18**    | e0/2      | 10.52.200.6  | /30   | R24 (AS 520)    |
| **R18**    | e0/3      | 10.52.250.14 | /30   | R26 (AS 520)    |
| **R18**    | e0/1      | 10.20.42.1   | /30   | R17             |
| **R18**    | e0/0      | 10.20.42.5   | /30   | R16             |
|            |           |              |       |                 |
| **R17**    | Lo0       | 10.20.42.17  | /32   | Loopbackup int. |
| **R17**    | e0/1      | 10.20.42.2   | /30   | R18             |
| **R17**    | e0/0.10   | 10.20.10.1   | /24   |                 |
| **R17**    | e0/0.20   | 10.20.20.2   | /24   |                 |
| **R17**    | e0/0.99   | 10.20.99.17  | /24   |                 |
|            |           |              |       |                 |
| **R16**    | Lo0       | 10.20.42.16  | /32   | Loopbackup int. |
| **R16**    | e0/1      | 10.20.42.6   | /30   | R18             |
| **R16**    | e0/3      | 10.20.42.9   | /30   | R32             |
| **R16**    | e0/2.10   | 10.20.10.2   | /24   |                 |
| **R16**    | e0/2.20   | 10.20.20.1   | /24   |                 |
| **R16**    | e0/2.99   | 10.20.99.16  | /24   |                 |
|            |           |              |       |                 |
| **R32**    | Lo0       | 10.20.42.32  | /32   | Loopbackup int. |
| **R32**    | e0/0      | 10.20.42.10  | /30   | R16             |
|            |           |              |       |                 |
| **SW9**    | Vlan99    | 10.20.99.9   | /24   |                 |
| **SW9**    | e0/0      |              |       | Po1             |
| **SW9**    | e0/1      |              |       | Po1             |
|            |           |              |       |                 |
| **SW10**   | Vlan99    | 10.20.99.10  | /24   |                 |
| **SW10**   | e0/0      |              |       | Po1             |
| **SW10**   | e0/1      |              |       | Po1             |
|            |           |              |       |                 |
| **VPC8**   | eth0      | 10.20.10.10  | /24   | SW9             |
| **VPC10**  | eth0      | 10.20.20.10  | /24   | SW10            |
|            |           |              |       |                 |

#### Чокурдах

| Устройство | Интерфейс | IP Адрес      | Маска | Описание      |
| ---------- | --------- | ------------- | ----- | ------------- |
| **R28**    | Lo0       | 192.168.28.1  | /32   | Loopback int. |
| **R28**    | e0/0      | 10.52.250.10  | /30   | R26           |
| **R28**    | e0/1      | 10.52.250.6   | /30   | R25           |
| **R28**    | e0/2.99   | 192.168.99.28 | /24   |               |
| **R28**    | e0/2.30   | 10.3.30.1     | /24   |               |
| **R28**    | e0/2.31   | 10.3.31.1     | /24   |               |
|            |           |               |       |               |
| **SW29**   | Vlan99    | 192.168.99.29 | /24   |               |
| **SW29**   | e0/0      |               |       |               |
| **SW29**   | e0/1      |               |       |               |
| **SW29**   | e0/2      |               |       | R28           |
|            |           |               |       |               |
| **VPC30**  | eth0      | 10.3.30.30    | /24   | SW29          |
| **VPC31**  | eth0      | 10.3.31.31    | /24   | SW29          |

### Конфигурации устройств

#### Провайдеры: AS 101 - Киторн, AS 301 - Ламас и AS 520 - Триада

##### Настройка для R21

```
hostname R21
no ip domain-lookup
!
interface Loopback0
 ip address 172.16.30.1 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.254.6 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 172.16.101.10 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 172.16.30.6 255.255.255.252
 no shutdown
!
ip route 10.100.0.0 255.255.0.0 10.100.254.5
ip route 172.16.101.0 255.255.255.0 172.16.101.9
ip route 10.52.0.0 255.255.0.0 172.16.30.5
!
end
write
```

##### Настройка для R22

```
hostname R22
no ip domain-lookup
!
interface Loopback0
 ip address 172.16.101.1 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.254.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 172.16.101.9 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 172.16.101.5 255.255.255.252
 no shutdown
!
ip route 10.100.0.0 255.255.0.0 10.100.254.1
ip route 172.16.30.0 255.255.255.0 172.16.101.10
ip route 10.52.0.0 255.255.0.0 172.16.101.6
!
end
write
```


##### Настройка для R23

```
hostname R23
no ip domain-lookup
!
interface Loopback0
 ip address 10.52.0.23 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.101.6 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.52.1.9 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.52.1.1 255.255.255.252
 no shutdown
!
ip route 10.52.0.24 255.255.255.255 10.52.1.2
ip route 10.52.0.25 255.255.255.255 10.52.1.10
ip route 10.52.0.26 255.255.255.255 10.52.1.2
ip route 10.52.250.0 255.255.255.252 10.52.1.10
ip route 10.52.250.8 255.255.255.252 10.52.1.2
ip route 10.20.42.0 255.255.255.0 10.52.1.2
ip route 10.100.0.0 255.255.0.0 10.52.1.2
!
end
write
```

##### Настройка для R24

```
hostname R24
no ip domain-lookup
!
interface Loopback0
 ip address 10.52.0.24 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.30.5 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.52.1.13 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.52.1.2 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.52.200.5 255.255.255.252
 no shutdown
!
ip route 10.52.0.23 255.255.255.255 10.52.1.1
ip route 10.52.0.25 255.255.255.255 10.52.1.14
ip route 10.52.0.26 255.255.255.255 10.52.1.14
ip route 10.52.250.0 255.255.255.252 10.52.1.14
ip route 10.52.250.8 255.255.255.252 10.52.1.14
ip route 172.16.101.4 255.255.255.252 10.52.1.1
ip route 10.20.42.0 255.255.255.0 10.52.200.6
!
end
write
```

##### Настройка для R25

```
hostname R25
no ip domain-lookup
!
interface Loopback0
 ip address 10.52.0.25 255.255.255.255
!
interface Ethernet0/0
 ip address 10.52.1.10 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.52.250.1 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.52.1.5 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.52.250.5 255.255.255.252
 no shutdown
!
ip route 10.52.0.23 255.255.255.255 10.52.1.9
ip route 10.52.0.24 255.255.255.255 10.52.1.6
ip route 10.52.0.26 255.255.255.255 10.52.1.6
ip route 172.16.101.4 255.255.255.252 10.52.1.9
ip route 172.16.30.4 255.255.255.252 10.52.1.6
ip route 10.20.42.0 255.255.255.0 10.52.1.6
!
end
write
```

##### Настройка для R26

```
hostname R26
no ip domain-lookup
!
interface Loopback0
 ip address 10.52.0.26 255.255.255.255
!
interface Ethernet0/0
 ip address 10.52.1.14 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.52.250.9 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.52.1.6 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.52.250.13 255.255.255.252
 no shutdown
!
ip route 10.52.0.23 255.255.255.255 10.52.1.13
ip route 10.52.0.24 255.255.255.255 10.52.1.13
ip route 10.52.0.25 255.255.255.255 10.52.1.5
ip route 172.16.101.4 255.255.255.252 10.52.1.13
ip route 172.16.30.4 255.255.255.252 10.52.1.13
ip route 10.20.42.0 255.255.255.0 10.52.250.14
!
end
write
```

#### AS1001 - Москва

##### Настройка для R14

```
hostname R14
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.14 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.1.5 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.100.1.1 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.100.254.1 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.100.1.25 255.255.255.252
 no shutdown
!
ip route 10.100.0.19 255.255.255.255 10.100.1.26
ip route 10.100.0.13 255.255.255.255 10.100.1.2
ip route 10.100.0.12 255.255.255.255 10.100.1.6
ip route 172.16.0.0 255.255.0.0 10.100.254.2
ip route 0.0.0.0 0.0.0.0 10.100.254.2
!
end
write
```

##### Настройка для R15

```
hostname R15
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.15 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.1.9 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.100.1.20 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.100.254.5 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.100.1.29 255.255.255.252
 no shutdown
!
ip route 10.100.0.13 255.255.255.255 10.100.1.10
ip route 10.100.0.20 255.255.255.255 10.100.1.30
ip route 10.100.0.12 255.255.255.255 10.100.1.22
ip route 172.16.0.0 255.255.0.0 10.100.254.6
ip route 0.0.0.0 0.0.0.0 10.100.254.6
!
end
write
```

##### Настройка для R19

```
hostname R19
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.19 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.1.26 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.100.1.25
!
end
write
```

##### Настройка для R20

```
hostname R20
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.20 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.100.1.30 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.100.1.29
!
end
write
```

##### Настройка для R12

```
hostname R12
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.12 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 no ip address
 no shutdown
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.100.10.1 255.255.255.0
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.100.20.2 255.255.255.0
!
interface Ethernet0/0.99
 encapsulation dot1Q 99
 ip address 10.100.99.12 255.255.255.0
!
interface Ethernet0/1
 no shutdown
!
interface Ethernet0/2
 ip address 10.100.1.6 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.100.1.22 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.100.1.5
ip route 10.100.0.13 255.255.255.255 10.100.99.13
!
end
write
```

##### Настройка для R13

```
hostname R13
no ip domain-lookup
!
interface Loopback0
 ip address 10.100.0.13 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 no ip address
 no shutdown
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.100.10.2 255.255.255.0
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.100.20.1 255.255.255.0
!
interface Ethernet0/0.99
 encapsulation dot1Q 99
 ip address 10.100.99.13 255.255.255.0
!
interface Ethernet0/1
 no shutdown
!
interface Ethernet0/2
 ip address 10.100.1.10 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 10.100.1.2 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.100.1.9
ip route 10.100.0.12 255.255.255.255 10.100.99.12
!
end
write
```

##### Настройка для SW4

```
hostname SW4
no ip domain-lookup
!
vlan 10,20,99
!
interface range Ethernet0/2-3
 description SW5
 channel-group 1 mode active
 channel-protocol lacp
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary
!
spanning-tree portfast default
!
interface range Ethernet0/0-1
 description ACCESS
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
interface range Ethernet1/0-1
 description UPLINK
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
interface Vlan99
 ip address 10.100.99.4 255.255.255.0
 no shutdown
!
ip default-gateway 10.100.99.12
!
end
write
```

##### Настройка для SW5

```
hostname SW5
no ip domain-lookup
!
vlan 10,20,99
!
interface range Ethernet0/2-3
 description SW4
 channel-group 1 mode passive
 channel-protocol lacp
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
spanning-tree vlan 10 root secondary
spanning-tree vlan 20 root primary
!
interface range Ethernet0/0-1
 description ACCESS
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
interface range Ethernet1/0-1
 description UPLINK
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
interface Vlan99
 ip address 10.100.99.5 255.255.255.0
 no shutdown
!
ip default-gateway 10.100.99.13
!
end
write
```

##### Настройка SW3

```
hostname SW3
no ip domain-lookup
!
vlan 10,99
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,99
 no shutdown
!
interface Ethernet0/2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
!
interface Vlan99
 ip address 10.100.99.3 255.255.255.0
 no shutdown
!
ip default-gateway 10.100.99.12
!
end
write
```

##### Настройка SW2

```
hostname SW2
no ip domain-lookup
!
vlan 20,99
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,99
 no shutdown
!
interface Ethernet0/2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
!
interface Vlan99
 ip address 10.100.99.2 255.255.255.0
 no shutdown
!
ip default-gateway 10.100.99.13
end
write
```

##### VPC1

```
set pcname VPC1
ip 10.100.10.10 255.255.255.0 10.100.10.1
save
```

##### VPC7

```
set pcname VPC1
ip 10.100.20.10 255.255.255.0 10.100.20.1
save
```

#### AS 2042 - С.-Петербург

##### Настройка для R18

```
hostname R18
no ip domain-lookup
!
interface Loopback0
 ip address 10.20.42.18 255.255.255.255
 no shutdown
!
interface Ethernet0/0  
description R16
ip address 10.20.42.5 255.255.255.252  
no shutdown  
!
interface Ethernet0/1  
description R17
ip address 10.20.42.1 255.255.255.252  
no shutdown  
!
interface Ethernet0/2  
description R24
ip address 10.52.200.6 255.255.255.252  
no shutdown  
!
interface Ethernet0/3  
description R26 
ip address 10.52.250.14 255.255.255.252  
no shutdown  
!
ip route 10.20.42.16 255.255.255.255 10.20.42.6  
ip route 10.20.42.17 255.255.255.255 10.20.42.2  
ip route 10.20.42.32 255.255.255.255 10.20.42.6
ip route 10.20.42.10 255.255.255.0 10.20.42.2  
ip route 10.20.42.20 255.255.255.0 10.20.42.2  
ip route 10.20.42.99 255.255.255.0 10.20.42.2  
ip route 10.52.0.0 255.255.0.0 10.52.200.5  
ip route 172.16.0.0 255.255.0.0 10.52.200.5
!
end
write
```

##### Настройка для R17

```
hostname R17
no ip domain-lookup
!
interface Loopback0
 ip address 10.20.42.17 255.255.255.255
!
interface Ethernet0/1
 description R18
 ip address 10.20.42.2 255.255.255.252
 no shutdown
!
interface Ethernet0/0
 no shutdown
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.20.10.1 255.255.255.0
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.20.20.2 255.255.255.0
!
interface Ethernet0/0.99
 encapsulation dot1Q 99 native
 ip address 10.20.99.17 255.255.255.0
!
ip route 10.20.42.16 255.255.255.255 10.20.42.1
ip route 10.20.42.18 255.255.255.255 10.20.42.1
ip route 10.20.42.32 255.255.255.255 10.20.42.1
ip route 0.0.0.0 0.0.0.0 10.20.42.1
!
end
write
```

##### Настройка для R16

```
hostname R16
no ip domain-lookup
!
interface Loopback0
ip address 10.20.42.16 255.255.255.255
!
interface Ethernet0/1
description R18
ip address 10.20.42.6 255.255.255.252
no shutdown
!
interface Ethernet0/3
description R32
ip address 10.20.42.9 255.255.255.252
no shutdown
!
interface Ethernet0/2
no shutdown
!
interface Ethernet0/2.10
encapsulation dot1Q 10
ip address 10.20.10.2 255.255.255.0
!
interface Ethernet0/2.20
encapsulation dot1Q 20
ip address 10.20.20.1 255.255.255.0
!
interface Ethernet0/2.99
encapsulation dot1Q 99 native
ip address 10.20.99.16 255.255.255.0
!
ip route 10.20.42.17 255.255.255.255 10.20.42.5
ip route 10.20.42.18 255.255.255.255 10.20.42.5
ip route 10.20.42.32 255.255.255.255 10.20.42.10
ip route 0.0.0.0 0.0.0.0 10.20.42.5
!
end
write
```


##### Настройка для R32

```
hostname R32
no ip domain-lookup
!
interface Loopback0  
ip address 10.20.42.32 255.255.255.255  
!
interface Ethernet0/0  
description R16  
ip address 10.20.42.10 255.255.255.252  
no shutdown  
! 
ip route 10.20.42.16 255.255.255.255 10.20.42.9  
ip route 10.20.42.17 255.255.255.255 10.20.42.9  
ip route 10.20.42.18 255.255.255.255 10.20.42.9  
ip route 10.20.42.10 255.255.255.0 10.20.42.9  
ip route 10.20.42.20 255.255.255.0 10.20.42.9  
ip route 10.20.42.99 255.255.255.0 10.20.42.9  
ip route 0.0.0.0 0.0.0.0 10.20.42.9
!
end
write
```

##### Настройка для SW9

```
hostname SW9
no ip domain-lookup
!
vlan 10,20,30
!
interface Vlan99
 ip address 10.20.99.9 255.255.255.0
 no shutdown
!
ip default-gateway 10.20.99.17
!
interface range Ethernet 0/0-1  
description SW10  
switchport trunk encapsulation dot1q  
switchport mode trunk  
switchport trunk native vlan 99  
switchport trunk allowed vlan 10,20,99  
channel-group 1 mode active  
no shutdown
!
interface Port-channel1  
description SW10  
switchport trunk encapsulation dot1q  
switchport mode trunk  
switchport trunk native vlan 99  
switchport trunk allowed vlan 10,20,99
!  
interface Ethernet0/2  
description VPC8  
switchport mode access  
switchport access vlan 10  
spanning-tree portfast  
no shutdown
!
interface Ethernet0/3  
description R17  
switchport trunk encapsulation dot1q  
switchport mode trunk  
switchport trunk native vlan 99  
switchport trunk allowed vlan 10,20,99  
no shutdown
!
interface Ethernet1/0  
description R16  
switchport trunk encapsulation dot1q  
switchport mode trunk  
switchport trunk native vlan 99  
switchport trunk allowed vlan 10,20,99  
no shutdown
!
end
write
```

##### Настройка для SW10

```
hostname SW10
no ip domain-lookup
!
vlan 10,20,99
!
interface Vlan99
 ip address 10.20.99.10 255.255.255.0
 no shutdown
!
ip default-gateway 10.20.99.16
!
interface range Ethernet 0/0-1
 description SW9
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 channel-group 1 mode passive
 no shutdown
!
interface Port-channel1
 description SW9
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
!
interface Ethernet0/2
 description VPC10
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
!
interface Ethernet0/3
 description R16
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
interface Ethernet1/0
 description R17
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
!
end
write
```

##### Настройка для VPC8

```
set pcname VPC8
ip 10.20.10.10 255.255.255.0 10.20.10.1
save
```

##### Настройка для VPC10

```
set pcname VPC8
ip 10.20.20.10 255.255.255.0 10.20.20.1
save
```


#### Чокурдах

##### Настройка для R28

```
hostname R28
no ip domain-lookup
!
interface Loopback0
 ip address 192.168.28.1 255.255.255.255
 no shutdown
!
interface Ethernet0/0
 ip address 10.52.250.10 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.52.250.6 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 no ip address
 no shutdown
!
interface Ethernet0/2.99
 encapsulation dot1Q 99
 ip address 192.168.99.28 255.255.255.0
!
interface Ethernet0/2.30
 encapsulation dot1Q 30
 ip address 10.3.30.1 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 10.52.250.9
ip route 10.3.30.0 255.255.255.0 10.52.250.10
ip route 10.3.31.0 255.255.255.0 10.52.250.10
!
end
write memory
```

##### Настройка для SW29

```
hostname SW29
no ip domain-lookup
!
vlan 30,99
!
interface Vlan99
 ip address 192.168.99.29 255.255.255.0
 no shutdown
!
interface Ethernet0/2
 description R28
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30,99
 no shutdown
!
interface Ethernet0/0
 description VPC30
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
interface Ethernet0/1
 description VPC31
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
ip default-gateway 192.168.99.28
!
end
write memory
```

##### Настройка для VPC30

```
set pcname VPC30
ip 10.3.30.30 255.255.255.0 10.3.30.1
save
```

##### Настройка для VPC31

```
set pcname VPC31
ip 10.3.30.31 255.255.255.0 10.3.30.1
save
```
