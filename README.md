# Репозиторий лабораторных работ курса "Дизайн сетей ЦОД" в otus.ru

Лабораторные работы:
- [Лаб. работа №1: Проектирование адресного пространства](labs/lab01/README.md)
- [Лаб. работа №2: Построение Underlay. OSPF](labs/lab02/README.md)
- [Лаб. работа №3: Построение Underlay. IS-IS](labs/lab03/README.md)
- [Лаб. работа №4: Построение Underlay. BGP](labs/lab04/README.md)
- [Лаб. работа №5: Построение VxLAN. L2VNI](labs/lab05/README.md)


### План адресации ###
Для всей площадки будет использоваться сеть **10.40.0.0/16**
|Network|||Description|
|---|---|---|---|
10.40.0.0/24|||EVPN Fabric|
||10.40.0.0/25||P2P for Spine-Leaf
|||10.40.0.0/31|otus-spine-sw01 <--> otus-leaf-sw01
|||10.40.0.2/31|otus-spine-sw02 <--> otus-leaf-sw01
|||10.40.0.4/31|otus-spine-sw01 <--> otus-leaf-sw02
|||10.40.0.6/31|otus-spine-sw02 <--> otus-leaf-sw02
|||10.40.0.8/31|otus-spine-sw01 <--> otus-leaf-sw03
|||10.40.0.10/31|otus-spine-sw02 <--> otus-leaf-sw03
||10.40.0.128/25||Loopback addresses for Spine, Leaf
|||10.40.0.128/32| loopback otus-spine-sw01
|||10.40.0.129/32| loopback otus-spine-sw02
|||10.40.0.130/32| loopback otus-leaf-sw01
|||10.40.0.131/32| loopback otus-leaf-sw02
|||10.40.0.132/32| loopback otus-leaf-sw03
10.40.100.0/24|||Client services LAB05|
