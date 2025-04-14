# Лабораторная работа №2. Построение Underlay. OSPF
Задачи:
1. Настроить OSPF в Underlay сети
2. Обеспечить доступность Loopback всех устройств
3. Зафиксировать в документации план работ, адресное пространство, схему сети, настройки
### Схема сети
Схема не изменена по сравнению с предыдущей работой. Добавлена информация о loopback адресах, а также стыковочные сети.
![](topology.jpg)
### Адресация сети
Для loopback в предыдущей работе была выделена сеть 10.40.0.128/25
|Address|Description|
|---|---|
|10.40.0.128/32| loopback otus-spine-sw01
|10.40.0.129/32| loopback otus-spine-sw02
|10.40.0.130/32| loopback otus-leaf-sw01
|10.40.0.131/32| loopback otus-leaf-sw02
|10.40.0.132/32| loopback otus-leaf-sw03
### Конфигурация
> Изначальная конфигурация взята из лабораторной работы №1

**Настройка loopback адресов**
Настроим loopback адреса на всем оборудовании.
Пример конфигурации с otus-leaf-sw01 (на остальном аналогично):
```
otus-leaf-sw01(config)# int loopback0
otus-leaf-sw01(config-if)# ip address 10.40.0.130/32
otus-leaf-sw01(config-if)# end
```
#### Настройка OSPF
**Глобальная настройка OSPF**
Глобальная настройка включает в себя включение feature ospf (Cisco Nexus), запуск процесса OSPF и назначения router-id. Router ID берём равным loopback адресу.
Пример конфигурации с otus-leaf-sw01 (на остальном аналогично):
```
otus-leaf-sw01(config)#feature ospf
otus-leaf-sw01(config)# router ospf Underlay
otus-leaf-sw01(config-router)# router-id 10.40.0.130
```
**Настройка OSPF на физ. интерфейсах**
Запускаем процесс OSPF на интерфейсах и назначаем тип point-to-point (так как у нас прямой линк между LEAF и SPINE, также эта команда позволяет избежать лишних сообщений (выбор DR и BDR))
Пример конфигурации с otus-leaf-sw01 (на остальном аналогично):
```
otus-leaf-sw01(config)# interface eth1/1-2
otus-leaf-sw01(config-if-range)# ip router ospf Underlay area 0
otus-leaf-sw01(config-if-range)# ip ospf network point-to-point 
```
**Настройка OSPF на loopback**
Включаем OSPF на loopback интерфейсах, чтобы адреса начали анонсироваться соседям
Пример конфигурации с otus-leaf-sw01 (на остальном аналогично):
```
otus-leaf-sw01(config)# int loopback0
otus-leaf-sw01(config-if)# ip router ospf Underlay area 0
```
### Проверка
**Проверка наличия OSPF соседей**
У каждого spine установилось три соседства (с каждым LEAF)
Пример списка OSPF соседей с otus-spine-sw01:
```
otus-spine-sw01# show ip ospf neighbors 
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.40.0.130       1 FULL/ -          00:09:39 10.40.0.1       Eth1/1 
 10.40.0.131       1 FULL/ -          00:09:38 10.40.0.5       Eth1/2 
 10.40.0.132       1 FULL/ -          00:09:34 10.40.0.9       Eth1/3 
```
Пример списка OSPF соседей с otus-spine-sw02:
```
otus-spine-sw02# show ip ospf neighbors 
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.40.0.130       1 FULL/ -          00:00:35 10.40.0.3       Eth1/1 
 10.40.0.131       1 FULL/ -          00:00:32 10.40.0.7       Eth1/2 
 10.40.0.132       1 FULL/ -          00:00:11 10.40.0.11      Eth1/3 
```
**Проверка наличия маршрутов до loopback**
В таблицах маршрутизации у каждого устройства появились маршруты до всех адресов loopback. 
- У каждого spine по одному лучшему маршруту до лупбека каждого leaf и 3 лучших маршрута до лупбека другого spine (так как все три маршрута идентичны, то между ними происходит балансировка)
- У каждого leaf по два лучших маршрута до лупбека других лифов (через оба spine) и по одному маршруту до лупбека spine

Пример таблицы маршрутизации с otus-spine-sw01 (на spine-sw02 аналогично):
```
otus-spine-sw01# show ip route ospf-Underlay 
...
10.40.0.129/32, ubest/mbest: 3/0
    *via 10.40.0.1, Eth1/1, [110/81], 00:08:30, ospf-Underlay, intra
    *via 10.40.0.5, Eth1/2, [110/81], 00:08:30, ospf-Underlay, intra
    *via 10.40.0.9, Eth1/3, [110/81], 00:08:30, ospf-Underlay, intra
10.40.0.130/32, ubest/mbest: 1/0
    *via 10.40.0.1, Eth1/1, [110/41], 00:12:44, ospf-Underlay, intra
10.40.0.131/32, ubest/mbest: 1/0
    *via 10.40.0.5, Eth1/2, [110/41], 00:09:34, ospf-Underlay, intra
10.40.0.132/32, ubest/mbest: 1/0
    *via 10.40.0.9, Eth1/3, [110/41], 00:09:11, ospf-Underlay, intra
```
Пример таблицы маршрутизации с otus-leaf-sw01 (на leaf-sw02/03 аналогично):
```
otus-leaf-sw01# show ip route ospf-Underlay
...
10.40.0.128/32, ubest/mbest: 1/0
    *via 10.40.0.0, Eth1/1, [110/41], 00:10:23, ospf-Underlay, intra
10.40.0.129/32, ubest/mbest: 1/0
    *via 10.40.0.2, Eth1/2, [110/41], 00:10:06, ospf-Underlay, intra
10.40.0.131/32, ubest/mbest: 2/0
    *via 10.40.0.0, Eth1/1, [110/81], 00:11:10, ospf-Underlay, intra
    *via 10.40.0.2, Eth1/2, [110/81], 00:11:10, ospf-Underlay, intra
10.40.0.132/32, ubest/mbest: 2/0
    *via 10.40.0.0, Eth1/1, [110/81], 00:10:52, ospf-Underlay, intra
    *via 10.40.0.2, Eth1/2, [110/81], 00:10:52, ospf-Underlay, intra
```
**Проверка наличия связности между loopback**
Проверка выполнена при помощи ping с указанием адреса loopback в качестве source. Проверка выполнена на всех устройствах, но ниже выводы с одного spine и с одного leaf.
Пример выполнения команды ping с otus-spine-sw01 (на spine-sw02 аналогично):
```
otus-spine-sw01# ping 10.40.0.129 source 10.40.0.128
PING 10.40.0.129 (10.40.0.129) from 10.40.0.128: 56 data bytes
64 bytes from 10.40.0.129: icmp_seq=0 ttl=253 time=333.217 ms
64 bytes from 10.40.0.129: icmp_seq=1 ttl=253 time=86.915 ms
64 bytes from 10.40.0.129: icmp_seq=2 ttl=253 time=87.09 ms
64 bytes from 10.40.0.129: icmp_seq=3 ttl=253 time=140.855 ms
64 bytes from 10.40.0.129: icmp_seq=4 ttl=253 time=233.726 ms

--- 10.40.0.129 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 86.915/176.36/333.217 ms
otus-spine-sw01# ping 10.40.0.130 source 10.40.0.128
PING 10.40.0.130 (10.40.0.130) from 10.40.0.128: 56 data bytes
64 bytes from 10.40.0.130: icmp_seq=0 ttl=254 time=228.551 ms
64 bytes from 10.40.0.130: icmp_seq=1 ttl=254 time=174.095 ms
64 bytes from 10.40.0.130: icmp_seq=2 ttl=254 time=467.301 ms
64 bytes from 10.40.0.130: icmp_seq=3 ttl=254 time=65.274 ms
64 bytes from 10.40.0.130: icmp_seq=4 ttl=254 time=153.611 ms

--- 10.40.0.130 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 65.274/217.766/467.301 ms
otus-spine-sw01# ping 10.40.0.131 source 10.40.0.128
PING 10.40.0.131 (10.40.0.131) from 10.40.0.128: 56 data bytes
64 bytes from 10.40.0.131: icmp_seq=0 ttl=254 time=127.399 ms
64 bytes from 10.40.0.131: icmp_seq=1 ttl=254 time=85.069 ms
64 bytes from 10.40.0.131: icmp_seq=2 ttl=254 time=59.245 ms
64 bytes from 10.40.0.131: icmp_seq=3 ttl=254 time=49.445 ms
64 bytes from 10.40.0.131: icmp_seq=4 ttl=254 time=196.34 ms

--- 10.40.0.131 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 49.445/103.499/196.34 ms
otus-spine-sw01# ping 10.40.0.132 source 10.40.0.128
PING 10.40.0.132 (10.40.0.132) from 10.40.0.128: 56 data bytes
64 bytes from 10.40.0.132: icmp_seq=0 ttl=254 time=351.085 ms
64 bytes from 10.40.0.132: icmp_seq=1 ttl=254 time=171.765 ms
64 bytes from 10.40.0.132: icmp_seq=2 ttl=254 time=461.338 ms
64 bytes from 10.40.0.132: icmp_seq=3 ttl=254 time=545.413 ms
64 bytes from 10.40.0.132: icmp_seq=4 ttl=254 time=71.953 ms

--- 10.40.0.132 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 71.953/320.31/545.413 ms
```
Пример выполнения команды ping с otus-leaf-sw01 (на leaf-sw02/03 аналогично):
```
otus-leaf-sw01# ping 10.40.0.128 source 10.40.0.130
PING 10.40.0.128 (10.40.0.128) from 10.40.0.130: 56 data bytes
64 bytes from 10.40.0.128: icmp_seq=0 ttl=254 time=658.767 ms
64 bytes from 10.40.0.128: icmp_seq=1 ttl=254 time=170.415 ms
64 bytes from 10.40.0.128: icmp_seq=2 ttl=254 time=189.111 ms
64 bytes from 10.40.0.128: icmp_seq=3 ttl=254 time=101.127 ms
64 bytes from 10.40.0.128: icmp_seq=4 ttl=254 time=81.322 ms

--- 10.40.0.128 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 81.322/240.148/658.767 ms
otus-leaf-sw01# ping 10.40.0.129 source 10.40.0.130
PING 10.40.0.129 (10.40.0.129) from 10.40.0.130: 56 data bytes
64 bytes from 10.40.0.129: icmp_seq=0 ttl=254 time=113.868 ms
64 bytes from 10.40.0.129: icmp_seq=1 ttl=254 time=41.358 ms
64 bytes from 10.40.0.129: icmp_seq=2 ttl=254 time=182.371 ms
64 bytes from 10.40.0.129: icmp_seq=3 ttl=254 time=56.998 ms
64 bytes from 10.40.0.129: icmp_seq=4 ttl=254 time=49.355 ms

--- 10.40.0.129 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 41.358/88.789/182.371 ms
otus-leaf-sw01# ping 10.40.0.131 source 10.40.0.130
PING 10.40.0.131 (10.40.0.131) from 10.40.0.130: 56 data bytes
64 bytes from 10.40.0.131: icmp_seq=0 ttl=253 time=1374.65 ms
64 bytes from 10.40.0.131: icmp_seq=1 ttl=253 time=2144.2 ms
64 bytes from 10.40.0.131: icmp_seq=2 ttl=253 time=1597.74 ms
64 bytes from 10.40.0.131: icmp_seq=3 ttl=253 time=954.897 ms
64 bytes from 10.40.0.131: icmp_seq=4 ttl=253 time=1638.43 ms

--- 10.40.0.131 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 954.897/1541.98/2144.2 ms
otus-leaf-sw01# ping 10.40.0.132 source 10.40.0.130
PING 10.40.0.132 (10.40.0.132) from 10.40.0.130: 56 data bytes
64 bytes from 10.40.0.132: icmp_seq=0 ttl=253 time=1980.3 ms
64 bytes from 10.40.0.132: icmp_seq=1 ttl=253 time=526.999 ms
64 bytes from 10.40.0.132: icmp_seq=2 ttl=253 time=174.644 ms
64 bytes from 10.40.0.132: icmp_seq=3 ttl=253 time=367.371 ms
64 bytes from 10.40.0.132: icmp_seq=4 ttl=253 time=726.299 ms

--- 10.40.0.132 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 174.644/755.122/1980.3 ms
```
### Итог
OSPF успешно настроен как Underlay протокол. Связность между loopback присутствует.