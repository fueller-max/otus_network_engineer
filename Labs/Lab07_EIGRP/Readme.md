### EIGRP

### Цель:

Настроить EIGRP в С.-Петербург
Использовать named EIGRP

###  Задание:

1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.


### Решение:

На рисунке представлен план офиса с указанием IPv4 пространства.

![](/Labs/Lab07_EIGRP/pics/Overview.jpg)

Настройка EIGRP будет осуществляться на роутерах R16,R17,R18,R32 в рамках автономной системы AS2042.

Для выхода вовне у ротера R18 предусмотрено две сети 176.192.170.0 и  176.192.170.0. 
Для iner-VLAN роутинга будет использоваться свитч SW9 с функцией L3. Он также являеся шлюзом по умлочанию для хостов, находящихся в VLAN 10 и  VLAN 20.

<em>( К сожалению, возможности виртуализации не позволяются обеспечить работу протокола VRRP между SW9 и SW10. Использование VRRP было бы предпочтительнее в плане обеспечения устойчивости связности сети сети в случае выхода одного из свитчей из строя. )</em>


### Конфигруация ротуеров

Перед конфигурированием определимся с суммаризацией маршрутов.

Сети 172.16.8.0/29 и 172.16.9.0/29 (ротутер R17) объединим в общий маршрут 
172.16.8.0/21 (сети 172.16.8.0/24 - 172.16.15.0/24 )

Сети 172.16.16.0/29, 172.17.17.0/29, 172.17.18.0/29 (ротутер R16) объединим в общий маршрут 
172.16.16.0/21 (сети 172.16.8.0/24 - 172.16.23.0/24 )

Конфигруация R17:

![](/Labs/Lab07_EIGRP/pics/configs/R17.jpg)

Конфигруация R16:

![](/Labs/Lab07_EIGRP/pics/configs/R16.jpg)


На роутерах R17, R16 на интерфейсе e0/1(в сторону R18) используем суумаризацию по схеме, описанной выше 

Конфигруация R18:

![](/Labs/Lab07_EIGRP/pics/configs/R18.jpg)

На ротуере R18 на на интерфейсах e0/1 и e0/0 (в сторону R17 и R16) используем сумаризацию вида 
0.0.0.0/0 для пропогации маршрута по умлочанию.

Конфигруация R32:

![](/Labs/Lab07_EIGRP/pics/configs/R18.jpg)

Access-list для R32:

![](/Labs/Lab07_EIGRP/pics/configs/R32_access_list.jpg)

Для фильтрации всех маршрутов, приходящих на R32, кроме маршрута по умлочанию 
используем фильтрацию посредством access-list permit 0.0.0.0. 

### Проверка маршрутов на роутерах, полученных посредством EIGRP.

IP routes EIGRP R16:
![](/Labs/Lab07_EIGRP/pics/ip%20routes/R16_ip_eigrp.jpg)

IP routes EIGRP R17:
![](/Labs/Lab07_EIGRP/pics/ip%20routes/R17_ip_eigrp.jpg)

IP routes EIGRP R18:
![](/Labs/Lab07_EIGRP/pics/ip%20routes/R18_ip_eigrp.jpg)

IP routes EIGRP R32:
![](/Labs/Lab07_EIGRP/pics/ip%20routes/R32_ip_eigrp.jpg)

Как видим, маршруты совпадают с ожидаемыми:

- R16,R17 передабт на R18 суммаризированные маршруты + Loopback
- R16,R17,R32 получают маршрут по умочанию от R18
- R32 получает только маршрут по умлочанию

### Проверка сети

Проверим доступность Loopback интерфейсов всех роутеров c хоста VPC8:

![](/Labs/Lab07_EIGRP/pics/connectivity/VPC8_pingLoopbacks.jpg)

Проверим доступность Loopback интерфейсов всех роутеров c хоста VPC9:

![](/Labs/Lab07_EIGRP/pics/connectivity/VPC9_pingLoopbacks.jpg)

Проверим доступность VPC8 и VPC9 c роутера R18:

![](/Labs/Lab07_EIGRP/pics/connectivity/R18_ping_VPCs.jpg)

Также проверим доступность хоста VPC9 от  VPC8 (EIGRP тут участие не принимает, маршрутизация полностью через SW9 )

![](/Labs/Lab07_EIGRP/pics/connectivity/VPC8_ping_VPC9.jpg)
