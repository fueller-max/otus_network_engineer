### PBR

### Цель:

Настроить политику маршрутизации в офисе Чокурдах
Распределить трафик между 2 линками

###  Задание:

1. Настроить политику маршрутизации для сети офиса.
2. Распределить трафик между двумя линками с провайдером.
3. Настроить отслеживание линка через технологию IP SLA.(только для IPv4)
4. Настроить для офиса Лабытнанги маршрут по-умолчанию.

### Решение:

На рисунке представлена топология и IP план офиса Чокурдах, Лабытынаги и Триада, выступющего в роли ISP.

![](/Labs/Lab08_PBR/pics/PBR_overview_plan.jpg)


При настройке политики маршрутизации в офисе Чокурдах будем следовать следующим соображениям:
1. В офисе имеется три VLAN:  VLAN10, VLAN20 и VLAN200 (Mangement VLAN )
2. Inter-Vlan коммуникация осуществялетсяна роутере R28(схема "Router on stick")
3. Базовым статическим маршрутом по умолчанию будет маршрут 176.192.172.1 - назовем его ISP2, вторым (Backup) 176.192.172.2 - назовем его ISP2. В базовом варианте статические маршруты отличаются заданной метрикой и работают в режиме переключения в случае полного отказа линка. Балансировка и проверка доступности хоста не предусмотрена.
4. Для настройки PBR (Policy Based Routing) будем исходить из следующих условий:
   - Трафик с VLAN 10 идет по линку 176.192.172.1 (ISP1)
   - Трафик с VLAN 20, 200 идет по линку 176.192.172.2 (ISP2)
   - Обеспечена работа SLA для контроля доступности ISP1,ISP2 и    переключение на один рабочий линк всего трафика в случае сбоя любого из линков.



 ### Конфигурация R28:

Настроим базовые статические маршурты с различной метрикой: 

````
 ip route 0.0.0.0 0.0.0.0 176.192.172.1
 ip route 0.0.0.0 0.0.0.0 176.192.173.1 10
`````

Приступим к настройке PBR + SLA:

Настроим SLA 1,2 для контроля доступности хостов ISP 1,2:

````
ip sla 1
 icmp-echo 176.192.172.1 source-interface Ethernet0/0
 threshold 2
 timeout 2000
 frequency 3
ip sla schedule 1 life forever start-time now
!
ip sla 2
 icmp-echo 176.192.173.1 source-interface Ethernet0/1
 threshold 2
 timeout 2000
 frequency 3
ip sla schedule 2 life forever start-time now

```` 
для контроля доступности используем ping-запрос c частотой раз в 3 сек, таймаутом 2 сек.


Создам два трека для контроля SLA 1,2:

````
track 1 ip sla 1 reachability
!
track 2 ip sla 2 reachability
````

Создадим ACL для фильтрации трафика на базе IP адресов:

````
access-list 1 permit 192.168.10.0 0.0.0.255
!
access-list 2 permit 192.168.200.65
access-list 2 permit 192.168.20.0 0.0.0.255
````

Создаем route-map ы для переключения шлюзов с использованием контроля доступности ISP хостов:

````
route-map tracking permit 100
 match ip address 1
 set ip next-hop verify-availability 176.192.172.1 10 track 1
 set ip next-hop verify-availability 176.192.173.1 20 track 2
!
route-map tracking permit 120
 match ip address 2
 set ip next-hop verify-availability 176.192.173.1 10 track 2
 set ip next-hop verify-availability 176.192.172.1 20 track 1

````

Добавляем политику маршрутизации PBR на соответствующие интерфейсы:

````
interface Ethernet0/2.10
 description Default Gateway for VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip policy route-map tracking
!
interface Ethernet0/2.20
 description Default Gateway for VLAN 20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip policy route-map tracking
!
interface Ethernet0/2.200
 encapsulation dot1Q 200
 ip address 192.168.200.65 255.255.255.252
 ip policy route-map tracking
!
````


На этом настройка PBR закончена, протестируем конгфиграцию:

1. Проверим состояние SLA 1,2:
![](/Labs/Lab08_PBR/pics/SLA_stat.jpg)

Оба линка до ISP1 и ISP2 активны.

2. Проверим состояние Route-Map:

![](/Labs/Lab08_PBR/pics/Route_map_basic.jpg)

Как видим треки, привязанные к SLA 1,2 в route-map находятся в up и активируют необходиме маршруты по умолчанию. Стоит заметить, что выполнение route-map прекращается при выполнении первого совпадения, поэтому порядок следования выражений важен.

3. Запустим ping с VPC30 до ISP1 и проверим статистику route-map 100:

![](/Labs/Lab08_PBR/pics/VPC30_ping%20ISP1.jpg)

 Статистика route-map:
 ![](/Labs/Lab08_PBR/pics/Route_map_VPC30_ping.jpg)

 Как видим, пакеты с VPC 30 обрабатываются route-map 100, как и запланировано.

 4. Запустим ping с VPC31 до ISP2 и проверим статистику route-map 120:

 ![](/Labs/Lab08_PBR/pics/VPC31_ping%20ISP2.jpg)

 ![](/Labs/Lab08_PBR/pics/Route_map_VPC31_ping.jpg)

 Как видим, пакеты с VPC 31 обрабатываются route-map 120, как и запланировано.

 5. Проверим, что трафик с VPC30 и VPC31 разделяется по линкам между ISP1, IPS2 используя Wireshark.

 Трафик до ISP1(порт e0/0 R28):

 ![](/Labs/Lab08_PBR/pics/Wireshark_ISP1.jpg)

 Трафик до ISP2(порт e0/1 R28):

  ![](/Labs/Lab08_PBR/pics/Wireshark_ISP2.jpg)

  Как видно, в обоих случаях имеем трафик до ISP1 только VPC30, а до ISP2 только VPC31, что говорит о том, что трафик распределяется между линками.


  6. Сымитируем обрыв канала до ISP2(выключив порт e0/1 на R28) и проверим реакцию системы:

  route-map определили недоступность ISP1 и переназначили адрес перехода:

  ![](/Labs/Lab08_PBR/pics/Route_IPS2_shut.jpg)

  Трафик до ISP1(порт e0/0 R28):

  ![](/Labs/Lab08_PBR/pics/Wireshark_ISP1_ISP2_down.jpg)

  Как видно, трафик от VPC30 и VPC31 идет теперь через линк до ISP1.

  На этом, базовую настройку и тестирование связки PBR + SLA в офисе  Чокурдах считаем законченной.


### Конфигурация R27:

Настроим маршрут по умолчанию, как указано в задании:

  ````
  ip route 0.0.0.0 0.0.0.0 176.192.174.1
  ````


  