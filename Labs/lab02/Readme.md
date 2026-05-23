## Задание

Настроить политику маршрутизации в офисе Чокурдах и распределить трафик между двумя линками.

**Описание/Пошаговая инструкция выполнения домашнего задания:**

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите политику маршрутизации для сетей офиса.
2. Распределите трафик между двумя линками с провайдером.
3. Настроите отслеживание линка через технологию IP SLA (только для IPv4).
4. Настройте для офиса Лабытнанги маршрут по умолчанию.
5. План работы и изменения зафиксированы в документации .

## Решение:

Распределим трафик между двумя линками:
- VPC30 → через R25
- VPC31 → через R26

Если упадет линк до одного из двух маршрутизоторов, автоматически перенаправим трафик через работающий линк.

![[Pasted image 20260523120019.jpg]]

### Конфигурации устройств

##### Настройка для R25 (AS520 Триада)

```
conf t
ip route 10.3.10.0 255.255.255.0 10.52.250.6
```

##### Настройка для R26 (AS520 Триада)

```
conf t
ip route 10.3.10.0 255.255.255.0 10.52.250.10
```

##### Настройка для R27 (Лабытнанги)

```
conf t
ip route 0.0.0.0 0.0.0.0 10.52.250.1
```

##### Настройка для R28 (Чокурдах)

```
conf t

ip access-list standard VPC30  
  permit 10.3.10.30  
  
ip access-list standard VPC31  
  permit 10.3.10.31

ip sla 1  
  icmp-echo 10.52.250.5 source-interface Ethernet0/1  
  frequency 5

ip sla 2  
  icmp-echo 10.52.250.9 source-interface Ethernet0/0  
  frequency 5
  
ip sla schedule 1 life forever start-time now
ip sla schedule 2 life forever start-time now

track 1 ip sla 1 reachability  
track 2 ip sla 2 reachability

route-map PBR permit 10  
  match ip address VPC30  
  set ip next-hop verify-availability 10.52.250.5 1 track 1  
  set ip next-hop 10.52.250.9

route-map PBR permit 20  
  match ip address VPC31  
  set ip next-hop verify-availability 10.52.250.9 1 track 2  
  set ip next-hop 10.52.250.5

interface Ethernet0/2  
  ip policy route-map PBR

ip route 0.0.0.0 0.0.0.0 10.52.250.9 1  
ip route 0.0.0.0 0.0.0.0 10.52.250.5 10
```
