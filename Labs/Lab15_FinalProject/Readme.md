###  Финальный проект

### Тема: Отказоустойчивая сеть среднего бизнес-предприятия

### Цель:

1. Разработать действующую архитектуру сети среднего бизнес-предприятия, сотсоящего из основного офиса и удаленного филиала
2. Обеспечить достаточный уровень избыточности (redundancy) для обеспечения  надежности и безотказности сети
3. Использовать по возможности не-проприетарные решения для обеспечения взаимозамянемости компонентов 

### Решение:


### Архитектура сети

Общий план сети предприятия представлен ниже:

![](/Labs/Lab15_FinalProject/pics/General_plan.jpg)


L3 план:

![](/Labs/Lab15_FinalProject/pics/L3_plan.jpg)



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

### Настройка OSPF

![](/Labs/Lab15_FinalProject/pics/OSPF_diagram.jpg)
