###  Финальный проект

### Тема: Отказоустойчивая сеть среднего бизнес-предприятия

### Цель:

1. Разработать действующую архитектуру сети среднего бизнес-предприятия, соcтоящего из основного офиса и удаленного филиала
2. Обеспечить достаточный уровень избыточности (redundancy) для обеспечения  надежности и безотказности сети
3. Использовать непроприетарные решения для обеспечения свободной взаимозаменяемости компонентов 

### Решение:


### Архитектура сети

Общий план сети предприятия представлен ниже:


![](/Labs/Lab15_FinalProject/pics/General_plan.jpg)

Основной офис распределен на 3 этажа, на которых располагаются соответствующие отделы. Ядро сетевой инфраструктуры представлено двумя роутерами, каждый из которых подключен к провайдеру и двух L3 свитчей.

Также имеется один удаленный филиал, у которого должен быть доступ к ресурсам основного офиса.  

Ниже представлен L3 план, где указана IP адресация в рамках сети:

![](/Labs/Lab15_FinalProject/pics/L3_plan.jpg)


Стенд проекта в среде виртуализации EVE:

![](/Labs/Lab15_FinalProject/pics/EVE.jpg)

Сетевое оборудование офиса смоделировано в среде EVE, где проверено функционирование системы, а также выполнены тесты отказоустойчивости системы.


#### Таблица адресации VLAN 

|VLAN|Сеть|Маска подстети|Шлюз по умолчанию|
|---:|:---:|:--:|:----:|
|VLAN10|172.16.1.0|/25|172.16.1.1|
|VLAN20|172.16.1.128|/25|172.16.1.129|
|VLAN30|172.16.2.0|/25|172.16.2.1|
|VLAN40|172.16.2.128|/25|172.16.2.129|
|VLAN50|172.16.3.0|/25|172.16.3.1|
|VLAN60|172.16.3.128|/25|172.16.3.129|

#### Таблица распределения VLAN 

|VLAN|Отдел|Этаж |
|---:|:---:|:--:|
|VLAN10|Продажи(Sales)|1|
|VLAN20|Кадры(HR)|1|
|VLAN30|Финанcы(Finance)|2|
|VLAN40|Отдел заказов(Order_Proc)|2|
|VLAN30|Финасны(Finance)|2|
|VLAN50|Юридический(Legal)|3|
|VLAN60|Сервер(Server)|3|

Все рабочие станции в основном офисе находятся в VLAN10-VLAN50 и получают IP адрес автоматически с DHCP с сервера  172.16.3.141 (VLAN60)

Все рабочие станции, в том числе и удаленного филиала имеют доступ к файловому серверу  172.16.3.201 (VLAN60)


Сеть удаленного офиса 192.168.1.0/24.

#### Выбор FHRP для уровня L3 свитчей         

В рамках каждой подсети организовано подключение рабочих станций через L2 access свитчи к двум L3 свитчам (Multilayer_L3_1 и Multilayer_L3_1), которые работают с связке для обеспечения избыточности в сети, используя протокол FHRP,также обеспечивают inter-vlan роутинг без захода на уровень Core роутеров (выполняют роль GW). 

В качестве конкретной реализации FHRP доступно несколько вариантов:

HSRP vs VRRP vs GLBP as FHRP(First Hop Redundancy Protocols)

**HSRP** - проприетарный протокол CISCO, который предоставляет механизм, предназначенный для поддержки бесперебойного переключения IP-трафика. Здесь, в HSRP, два или более маршрутизаторов создают иллюзию виртуального маршрутизатора. HSRP позволяет вам настроить два или более маршрутизатора как резервные маршрутизаторы и только один маршрутизатор как активный маршрутизатор одновременно. Активный маршрутизатор выполняет работу по пересылке трафика. Если он выходит из строя, резервный маршрутизатор компенсирует его, принимая на себя все обязанности активного маршрутизатора и пересылая трафик.

**VRRP** - это открытый стандартный протокол IEEE, который обеспечивает избыточность в сети. Очень похож на  HSRP. Он также устраняет единую точку отказа, присущую статической маршрутизируемой среде по умолчанию. Это протокол сетевого уровня с номером протокола 112. Количество маршрутизаторов в группе действует как виртуальный логический маршрутизатор, который действует как шлюз по умолчанию для всех локальных хостов. Если какой-либо из маршрутизаторов выходит из строя, другие члены группы могут взять на себя обязанности по пересылке трафика.

**GLBP** - это проприетарный протокол Cisco, который обеспечивает избыточность, как и другие, в дополнение к нему, он также обеспечивает балансировку нагрузки. Он может выполнять такие функции, как балансировка нагрузки на нескольких маршрутизаторах, используя один виртуальный IP-адрес и несколько виртуальных MAC-адресов.

Одно из главных отличий заключается в том, что VRRP является отраслевым стандартом, в то время как HSRP и GLBP являются проприетарный протоколами Cisco. VRRP и HSRP выделяют по одному L3 коммутатору или маршрутизатору, чтобы быть активным в группе, тогда как GLBP может настраивать схемы балансировки нагрузки.

В нашем проекте в качестве FHRP будет использоваться именно проткол VRRP главным образом из-за его открытости и непривязанности к конкретному вендору. Для обеспечения также функций балансировки трафика будем использовать доступные настройки/подходы протокола VRRP.


Рассмотрим настройку VRRP  на примере VLAN 10:

ML_L3_1:
`````
interface Vlan10
 description Sales_SVI
 ip address 172.16.1.1 255.255.255.128
 ip helper-address 172.16.3.141
 vrrp 10 ip 172.16.1.1
 vrrp 10 timers advertise 2
 vrrp 10 timers learn
`````

ML_L3_2:

`````
interface Vlan10
 description Sales_SVI
 ip address 172.16.1.2 255.255.255.128
 ip helper-address 172.16.3.141
 vrrp 10 ip 172.16.1.1
 vrrp 10 timers advertise 2
 vrrp 10 timers learn
`````


На обоих L3 свитчах создан SVI интерфейс выполняющего роль шлюза по умолчанию. На каждом из интерфейсов запущен процесс VRRP. 
В данном случае SVI первого L3 свита имеет адрес 172.16.1.1, второго 172.16.1.2. Адрес виртуального шлюза 172.16.1.1. Адрес виртуального шлюза совпадает в данном случае с первым L3 свитчем, что делает его т.н. IP owner, и данный свит будет выпонять функцию Master по умолчанию (т.е. доп. настроек приоритета не требуется). Время посылки между VRRP фреймами установлено в 2 сек. 

Также на каждом SVI настроен DHCP relay от DHCP сервера(172.16.3.141), который также резервируется. 

Проверим состояние VRRP в рабочем состоянии, когда оба L3 свитча включены:

````
ML_L3_1#sh vrrp
Vlan10 - Group 10
  State is Master
  Virtual IP address is 172.16.1.1
  Virtual MAC address is 0000.5e00.010a
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.1.1 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec
````

````
Vlan10 - Group 10
  State is Backup
  Virtual IP address is 172.16.1.1
  Virtual MAC address is 0000.5e00.010a
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.1.1, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 5.173 sec) Learning
````
 
 Как видим, что первый L3 свитч стоит в состоянии Master, а второй в режиме Slave.

 Проведем тест путем выключения первого L3 свитча с проверкой доступности файлового сервера со станции PC_Sales_1:

 ![](/Labs/Lab15_FinalProject/pics/VRRP_both_L3_OK.png)

 Симулируем полный отказ первого L3 свитча путем его выключения:


 ![](/Labs/Lab15_FinalProject/pics/VRRP_first_L3_Crash.png)

Видим, что после некоторого периода недоступности файловый сервер снова доступен. При этом видим, что второй свитч взял на себя функции GW для VLAN10 (перешел в состояние Master для интерфейса VLAN10) (где и находится рабочая станция PC_Sales_1).


Соответственно, на данном этапе обеспечена беспербойность сети, т.к. даже полный выход из строя одного из L3 свитчей не блокирует работу сети, а после недолгого простоя связность сети восстанавливается.

Что касается балансировки нагрузки, то как и было сказано выше, VRRP нативно не обладает функциями балансировки(в отличие от, например, GLBP).

Однако путем настройки групп и выделения ведущего/ведомого устройства можно добиться распределения нагрузки между устройствами. Это не является балансировкой в чистом виде, однако при грамотной настройке позволяет приблизится к равномерной нагрузке.


В нашем случае для балансировки будет использовать подход разделения ведущего/ведомого устройства для каждого VLAN в соотвествии с таблицей:


|VLAN|ML_L3_1|ML_L3_2|
|---:|:---:|:--:|
|VLAN10|Master|Backup|
|VLAN20|Master|Backup|
|VLAN30|Backup|Master|
|VLAN40|Backup|Master|
|VLAN50|Backup|Master|
|VLAN60|Backup|Master|

Что соотвествует выводу состояния VRRP на L3 свитчах:

````
ML_L3_1#sh vrrp
Vlan10 - Group 10
  State is Master
  Virtual IP address is 172.16.1.1
  Virtual MAC address is 0000.5e00.010a
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.1.1 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec

Vlan20 - Group 20
  State is Master
  Virtual IP address is 172.16.1.129
  Virtual MAC address is 0000.5e00.0114
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.1.129 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec

Vlan30 - Group 30
  State is Backup
  Virtual IP address is 172.16.2.1
  Virtual MAC address is 0000.5e00.011e
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.2.1, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 5.955 sec) Learning

Vlan40 - Group 40
  State is Backup
  Virtual IP address is 172.16.2.129
  Virtual MAC address is 0000.5e00.0128
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.2.129, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 6.277 sec) Learning

Vlan50 - Group 50
  State is Backup
  Virtual IP address is 172.16.3.1
  Virtual MAC address is 0000.5e00.0132
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.3.1, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 4.907 sec) Learning

Vlan60 - Group 60
  State is Backup
  Virtual IP address is 172.16.3.129
  Virtual MAC address is 0000.5e00.013c
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.3.129, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 5.340 sec) Learning


````

````
ML_L3_2>sh vrrp
Vlan10 - Group 10
  State is Backup
  Virtual IP address is 172.16.1.1
  Virtual MAC address is 0000.5e00.010a
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.1.1, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 5.310 sec) Learning

Vlan20 - Group 20
  State is Backup
  Virtual IP address is 172.16.1.129
  Virtual MAC address is 0000.5e00.0114
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 172.16.1.129, priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.609 sec (expires in 6.433 sec) Learning

Vlan30 - Group 30
  State is Master
  Virtual IP address is 172.16.2.1
  Virtual MAC address is 0000.5e00.011e
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.2.1 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec

Vlan40 - Group 40
  State is Master
  Virtual IP address is 172.16.2.129
  Virtual MAC address is 0000.5e00.0128
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.2.129 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec

Vlan50 - Group 50
  State is Master
  Virtual IP address is 172.16.3.1
  Virtual MAC address is 0000.5e00.0132
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.3.1 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec

Vlan60 - Group 60
  State is Master
  Virtual IP address is 172.16.3.129
  Virtual MAC address is 0000.5e00.013c
  Advertisement interval is 2.000 sec
  Preemption enabled
  Priority is 255
  Master Router is 172.16.3.129 (local), priority is 255
  Master Advertisement interval is 2.000 sec
  Master Down interval is 6.003 sec
````

### Проблема расщепления VRRP

При нарушении связности между L3 свитчами может возникнуть ситуация, когда оба свитча могут стать Master, что может приводить к нежелательным последствиям, таким как серьезным нарушение маршрутизации.

Показана ситуация, когда из-за обрыва линка до access switch оба GW в VLAN60 перешли в состояние Master:

![](/Labs/Lab15_FinalProject/pics/Vlan60_bothMasters.jpg)


Для устранения данной ситуации рекомендуется наличие прямого линка между L3 устройствами.

![](/Labs/Lab15_FinalProject/pics/PortChannel.jpg)

Теперь оба устройства L3 имеют связность вне зависимости от нижнего уровня. 

Настройки port-channel L3 свитчей.

````
interface Port-channel1
 switchport trunk allowed vlan 1,10,20,30,40,50,60,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk allowed vlan 1,10,20,30,40,50,60,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/3
 switchport trunk allowed vlan 1,10,20,30,40,50,60,100
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
````

#### Взаимодействие VRRP с STP.

Работа протокола VRRP никак не связана с работой протокола STP. Но, особенно при наличии L2-петель протокол STP может изменить топологию, что может привести к неоптимальному распределения трафика.

Например, для VLAN50 VRRP Master является L3_2, однако для STP Root Bridge является L3_1. Это приведет к топологии, что трафик от хостов VLAN50 пойдет до GW(L3_2) через L3_1. Это не приведет к потере работоспобности сети, однако нарушит оптимальную балансировку трафика.

![](/Labs/Lab15_FinalProject/pics/STP_VRRP_Master_Root.jpg)


![](/Labs/Lab15_FinalProject/pics/Access_sub_routing_NOT_OK.jpg)


Для устранения данного эффекта необходимо учитывать совестную работу STP и VRPP и обеспечить так, что VRRP Master также был  и STP Root Bridge.


Для синхронизации работы STP и VRRP необходимо проводить доп.настройку STP для оптимального выбора Root`a в соотвествии с логикой VRRP.

Проведем настройки выбора Root  в STP в соответствии с политикой VRRP для обоих L3 свитчей:

````
ML_L3_1(config)#spanning-tree vlan 10 root primary
ML_L3_1(config)#spanning-tree vlan 20 root primary

ML_L3_2(config)#spanning-tree vlan 30 root primary
ML_L3_2(config)#spanning-tree vlan 40 root primary
ML_L3_2(config)#spanning-tree vlan 50 root primary
ML_L3_2(config)#spanning-tree vlan 60 root primary
````

````
ML_L3_1(config)#spanning-tree vlan 30 root secondary
ML_L3_1(config)#spanning-tree vlan 40 root secondary
ML_L3_1(config)#spanning-tree vlan 50 root secondary
ML_L3_1(config)#spanning-tree vlan 60 root secondary

ML_L3_2(config)#spanning-tree vlan 10 root secondary
ML_L3_2(config)#spanning-tree vlan 20 root secondary
````

Также важно выполнение еще двух условий:
1. Использование семейства rapid STP для быстрой сходимости (необходимо, чтобы топология сформировалась как можно быстрее, до начала работы VRRP) c поддержкой STP для каждого VLAN (PVST,MSTP) 
2. Активирована опция preemt для возвращения Master в случае возобновления работы "упавшего" устройства (в VRRP включено по умолчанию)

В этом случае выбор root STP будет синхронизирован с VRRP и будет обеспечена оптимальная топология для балансировки трафика.

В случае отключении L3_2 Root STP и Master VRRP перемещается к L3_1:
![](/Labs/Lab15_FinalProject/pics/Access_sub_routing_R2_fails_VLAN50.jpg)


В случае возобновления работы L3_2 Root STP и Master VRRP перемещается обартно к L3_2:
![](/Labs/Lab15_FinalProject/pics/Access_sub_routing_R2_back_VLAN50.jpg)

#### Настройка MSTP

Протокол MSTP явялется предпочтительнымс точки зрения его непроприетарности.

Проведем настройку протокола MSTP:

Выделяем instance для каждого VLAN на L3_1, L3_2:


`````
ML_L3_1(config)#spanning-tree mst configuration
ML_L3_1(config)#name main_office
ML_L3_1(config)#revision 1
ML_L3_1(config)#instance 1 vlan 10
ML_L3_1(config)#instance 2 vlan 20
ML_L3_1(config)#instance 3 vlan 30
ML_L3_1(config)#instance 4 vlan 40
ML_L3_1(config)#instance 5 vlan 50
ML_L3_1(config)#instance 6 vlan 60
`````

Активация режима MSTP:
````
ML_L3_1(config)spanning-tree mode mst
````


Настройка приоритетов Root Bridge для L3_1,L3_2 в соотвествии с политикой VRRP:

````
ML_L3_1(config)#spanning-tree mst 1 root primary
ML_L3_1(config)#spanning-tree mst 2 root primary
ML_L3_1(config)#spanning-tree mst 3 root primary
ML_L3_1(config)#spanning-tree mst 4 root primary
ML_L3_1(config)#spanning-tree mst 5 root primary
ML_L3_1(config)#spanning-tree mst 5 root secondary
ML_L3_1(config)#spanning-tree mst 6 root secondary

````

````
ML_L3_2(config)#spanning-tree mst 1 root secondary
ML_L3_2(config)#spanning-tree mst 2 root secondary
ML_L3_2(config)#spanning-tree mst 3 root secondary
ML_L3_2(config)#spanning-tree mst 4 root secondary
ML_L3_2(config)#spanning-tree mst 5 root secondary
ML_L3_2(config)#spanning-tree mst 5 root primary
ML_L3_2(config)#spanning-tree mst 6 root primary
````

Пример состояния MSTP для 50 и 60 VLAN: 
![](/Labs/Lab15_FinalProject/pics/MSTP_VALN50_60.jpg)

Тест переключения Root Bridge в случае выхода из строz R2 для VLAN 50:

![](/Labs/Lab15_FinalProject/pics/MSTP_VALN50_Access_SW_R2_fails.jpg)
![](/Labs/Lab15_FinalProject/pics/MSTP_VALN50_Access_SW_R2_OK_back.jpg)



### Настройка провайдера ISP

Для обеспечения L3 связности между офисами необходимо настроить провайдера, имитриующего Интернет. 

Главный офис и филиал подключены к Интернет, через который посредством VPN ,будет осуществляться связность до внутренних ресурсов. Роль Интернета в данной работе будут выполнять два роутера, находящиеся в автономной системе AS300. Суммаризированный префикс выдачи IP клиентам 176.192.168./24, который будет нарезаться по маске /30 каждому клиенту. 

Для обеспечения передачи маршрутов и связности между роутерами настроен iBGP поверх Looback интерфейсов. Для связности Looback настроены статические маршруты по underlay сети 10.10.10.8/30 


![](/Labs/Lab15_FinalProject/pics/ISP1.jpg)


Ниже приведены настройки роутера ISP_R1 и ISP_R2:

````
!
hostname ISP_R1
!
ip cef
no ipv6 cef
!

!
interface Loopback0
 ip address 10.1.1.1 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.10.9 255.255.255.252
!
interface Ethernet0/1
 ip address 176.192.168.9 255.255.255.252
!
interface Ethernet0/2
 ip address 176.192.168.13 255.255.255.252
!

router bgp 300
 bgp log-neighbor-changes
 network 176.192.168.8 mask 255.255.255.252
 network 176.192.168.12 mask 255.255.255.252
 neighbor 10.2.2.2 remote-as 300
 neighbor 10.2.2.2 update-source Loopback0
!
ip route 10.2.2.2 255.255.255.255 10.10.10.10
!
end

````

````
!
hostname ISP_R2
!
ip cef
no ipv6 cef
!
interface Loopback0
 ip address 10.2.2.2 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.10.10 255.255.255.252
!
interface Ethernet0/1
 ip address 176.192.168.33 255.255.255.252
!
router bgp 300
 bgp log-neighbor-changes
 network 176.192.168.32 mask 255.255.255.252
 neighbor 10.1.1.1 remote-as 300
 neighbor 10.1.1.1 update-source Loopback0
!
ip route 10.1.1.1 255.255.255.255 10.10.10.9
!
end

````

### Настройка GRE, IPsec

После обеспечения L3 связности между офисами можно приступить к организации VPN туннелей между двумя роутерами главного офиса и роутером филиала.

Для организации VPN будем использовать GRE(Gerenic Routing Encapsulation) поверх IPsec с использованием pre-shared ключей.

Настройка включает в себя настройку двух фаз:
* Первая фаза включает себя настройку ISAKMP (Key Management Protocol)
* Вторая фаза настройка  IPSec 

Покажем настройку IPsec роутера Core_R1 головного офиса:

* Phase 1:

````
core_R1(config)#crypto isakmp policy 1
core_R1(config-isakmp)#encryption aes
core_R1(config-isakmp)#hash sha256
core_R1(config-isakmp)#authentication pre-share
core_R1(config-isakmp)#group 14
core_R1(config-isakmp)#lifetime 86400
!
core_R1(config)#crypto isakmp key secretkey address  0.0.0.0
````
Используем глобальную политику с приоритетом 1, метод шифрования AES, алгоритм хэширования SHA256, метод проверки подлинности PRE-SHARED KEY, 14 группа Диффи-Хеллмана, 86400 время жизни ключа сеанса до его смены.


* Phase 2:
````
core_R1(config)#crypto ipsec transform-set VTI esp-aes esp-sha-hmac
core_R1(cfg-crypto-trans)#mode transport
!
core_R1(config)#crypto ipsec profile IPSEC
core_R1(ipsec-profile)#set transform-set VTI
````

Создаем Transform-Set с именем VTI, режим transport более подходящий для GRE туннелей. Создаем IPSEC профайл и привязываем к нему созданный Transform-Set. 


На роутере создаем туннель c роутером филиала 176.192.168.34, c оверлейной сетью 101.0.1.0/24
и применяем к нему созданный профиль IPSEC:

````
interface Tunnel10
 ip address 101.0.1.1 255.255.255.0
 tunnel source 176.192.168.10
 tunnel destination 176.192.168.34
 tunnel path-mtu-discovery
 tunnel protection ipsec profile IPSEC

````

Далее на роутере филиала делаем аналогичные настройки для GRE, IPsec, GRE.

После настройка проверяем состояние GRE и IPsec:

* состояние туннеля GRE:
![](/Labs/Lab15_FinalProject/pics/GRE_R1.jpg)

* Состояние ISAKMP
![](/Labs/Lab15_FinalProject/pics/IPSEC_R1_isakmp.jpg)
* Состояние IPsec
![](/Labs/Lab15_FinalProject/pics/IPSEC_R1_crypto_ipsec.jpg)

Настройка туннеля между роутером R2 и R10 выполняется абсолютно аналогично.

В результате должно быть два туннеля: Core_R1 <-> R10 и Core_R2 <-> R10, поврех которых будет работать процесс OSPF.
 
### Настройка OSPF


Для обеспечения передачи маршрутов в рамках головного офиса, а также филиала запущен проткол динамической маршрутизации OSPF, принципиальная схема которого показана ниже:


![](/Labs/Lab15_FinalProject/pics/OSPF_diagram.jpg)

Backbone зона 0 (основная) развернута поверх двух GRE туннелей:
 Core_R10 <-> Core_R1
 Core_R10 <-> Core_R2

 а также два stub зоны: Area 1 для основого офиса и Area 3 для филиала.

  
  Для туннелей выбираем overlay сети 101.0.1.0/24 и 101.0.2.0/24 для первого и второго туннелей соответсвенно.

  Настройки GRE туннелей приведены ниже:

  ````
  hostname filiale_core_R10
  !
  interface Tunnel10
  ip address 101.0.1.2 255.255.255.0
  tunnel source 176.192.168.34
  tunnel destination 176.192.168.10
  !
   
  interface Tunnel20
  ip address 101.0.2.2 255.255.255.0
  tunnel source 176.192.168.34
  tunnel destination 176.192.168.14
  ````

  ````
  hostname core_R1
  !
  interface Tunnel10
  ip address 101.0.1.1 255.255.255.0
  tunnel source 176.192.168.10
  tunnel destination 176.192.168.34

  ````

  ````
  hostname core_R2
  !
  interface Tunnel20
  ip address 101.0.2.1 255.255.255.0
  tunnel source 176.192.168.14
  tunnel destination 176.192.168.34
  ````

  Соотвественно, поверх данных туннелей GRE строится Backbone Area 0. 
  Далее для всех роутеров добавлены соответствующие сети в зоны:
  
  ````
  hostname filiale_core_R10
  !
  router ospf 1
  router-id 1.1.1.10
  area 1 stub
  network 101.0.1.0 0.0.0.255 area 0
  network 101.0.2.0 0.0.0.255 area 0
  network 192.168.1.0 0.0.0.255 area 3
  ````

  ````
  hostname core_R1
  !
  ip ospf priority 255
  ip ospf priority 255
  !
  router ospf 1
  router-id 1.1.1.1
  area 1 stub
  network 101.0.1.0 0.0.0.255 area 0
  network 172.16.50.8 0.0.0.3 area 1
  network 172.16.50.16 0.0.0.3 area 1
  ````

  ````
  hostname core_R1
  !
  ip ospf priority 255
  ip ospf priority 255
  !
  router ospf 1
  router-id 1.1.1.2
  area 1 stub
  network 101.0.2.0 0.0.0.255 area 0
  network 172.16.50.12 0.0.0.3 area 1
  network 172.16.50.20 0.0.0.3 area 1
  network 172.16.100.12 0.0.0.0 area 1
  ````


  И настройки L3 свитчей в рамках процесса OSPF:

  ```` 
  hostname ML_L3_1
  !
  router ospf 1
  router-id 1.1.3.1
  area 1 stub
  passive-interface Vlan10
  passive-interface Vlan20
  passive-interface Vlan30
  passive-interface Vlan40
  passive-interface Vlan50
  passive-interface Vlan60
  network 172.16.0.0 0.0.3.255 area 1
  network 172.16.50.8 0.0.0.7 area 1
  ````
  ````
  hostname ML_L3_2
  !
  router ospf 1
  router-id 1.1.3.2
  area 1 stub
  passive-interface Vlan10
  passive-interface Vlan20
  passive-interface Vlan30
  passive-interface Vlan40
  passive-interface Vlan50
  passive-interface Vlan60
  network 172.16.0.0 0.0.3.255 area 1
  network 172.16.50.16 0.0.0.7 area 1
  ````

  На обоих свитчах все интерфейсы VlanX помечены как passive для того, чтобы исключить их из процесса OSPF, а лишь анонсировать сети наверх.

  Также в Core роутерах настроен  ospf priority 255 на соотвествующих интерфейсах для того, чтобы они гарантировано становились DR/BDR роутерами в рамках своей зоны 3.

  Проведем небольшое тестирование полученной кофигурации:

  Все устройства включены, у хоста филиала есть доступ к файловому серверу: 

  ![](/Labs/Lab15_FinalProject/pics/Filalle_ping_file_Server.jpg)

  Отключается первый ротуер. Временно пропадает сетевая связность, однако после периода времени связность восстанавливается, в таблице маршрутизации остаюся только префиксы через 2-ой туннель

  ![](/Labs/Lab15_FinalProject/pics/OSPF_R1_down.jpg)

  После включения первого роутера связь не прерываются, а в таблице маршуртизации появляются префксы, доступные через оба туннеля: 

  ![](/Labs/Lab15_FinalProject/pics/OSPF_R1_backagain.jpg)


Таким образом, применение протокола OSPF дало возможность во-первых упростить настройку самой маршрутизации и передачу префиксов в филиалы, а также обеспечть достаточный уровень избыточности сети таким образом, что выход из строя какого-либо устрйоства не ведет к полной потере связности.

### Настройки DHCP

Все рабочие станции в рамках сети должны получать IP адрес посредством DHCP сервера.

В основном офисе для этого предусмотрен выделенный сервер DHCP, находящийся в VLAN60 по адресу 172.16.3.141 
Настройки DHCP cервера приведены ниже:

````
hostname dhcp_server
!
ip dhcp excluded-address 172.16.1.1 172.16.1.9
ip dhcp excluded-address 172.16.1.129 172.16.1.138
ip dhcp excluded-address 172.16.2.1 172.16.2.9
ip dhcp excluded-address 172.16.2.129 172.16.2.138
ip dhcp excluded-address 172.16.3.1 172.16.3.9
!
ip dhcp pool VLAN_10
 network 172.16.1.0 255.255.255.128
 default-router 172.16.1.1
 dns-server 172.16.1.1
 lease 2
!
ip dhcp pool VLAN_20
 network 172.16.1.128 255.255.255.128
 default-router 172.16.1.129
 dns-server 172.16.1.129
 lease 2
!
ip dhcp pool VLAN_30
 network 172.16.2.0 255.255.255.128
 default-router 172.16.2.1
 dns-server 172.16.2.1
 lease 2
!
ip dhcp pool VLAN_40
 network 172.16.2.128 255.255.255.128
 default-router 172.16.2.128
 dns-server 172.16.2.128
 lease 2
!
ip dhcp pool VLAN_50
 network 172.16.3.0 255.255.255.128
 default-router 172.16.3.0
 dns-server 172.16.3.0
 lease 2
!
````

В офисе филиала роль DHCP выполняет непосредственно роутер Core_R10:

````
!
hostname filiale_core_R10
!
!
ip dhcp excluded-address 192.168.1.254
ip dhcp excluded-address 192.168.1.1 192.168.1.9
!
ip dhcp pool POOL-1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 192.168.1.1
 lease 2
!
````
