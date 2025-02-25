### Основные протоколы сети интернет


### Цель:
Настроить DHCP в офисе Москва
Настроить синхронизацию времени в офисе Москва
Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах


###  Задание:

1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
3. Настроить статический NAT для R20.
4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
5. Настроить статический NAT(PAT) для офиса Чокурдах.
6. Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и  R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.

### Решение:

1. [Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001](Readme.md#1-настроить-natpat-на-r14-и-r15-трансляция-должна-осуществляться-в-адрес-автономной-системы-as1001)
2. [Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042](Readme.md#2-настроить-natpat-на-r18-трансляция-должна-осуществляться-в-пул-из-5-адресов-автономной-системы-as2042)
3. [Настроить статический NAT для R20](Readme.md#3-настроить-статический-nat-для-r20)
4. [Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления](Readme.md#4-настроить-nat-так-чтобы-r19-был-доступен-с-любого-узла-для-удаленного-управления)
5. [Настроить статический NAT(PAT) для офиса Чокурдах](Readme.md#5-настроить-статический-natpat-для-офиса-чокурдах)
6. [Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и  R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP](Readme.md#6-настроить-для-ipv4-dhcp-сервер-в-офисе-москва-на-маршрутизаторах-r12-и--r13-vpc1-и-vpc7-должны-получать-сетевые-настройки-по-dhcp)
7. [Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13](Readme.md#7-настроить-ntp-сервер-на-r12-и-r13-все-устройства-в-офисе-москва-должны-синхронизировать-время-с-r12-и-r13)


Ссылка на актуальную схему сети: https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Lab04.drawio&dark=auto#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Ffueller-max%2Fdrawio%2Fmain%2FLab04.drawio

#### 1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.

Перед настройкой NAT отключаем анонсы сетей офисов Москва и Санкт-Петербруг в BGP.

Последовательность настройки PAT:

To configure PAT, the following commands are required:

* configure the router’s inside interface using the ip nat inside command.
* configure the router’s outside interface using the ip nat outside command.
* configure an access list that includes a list of the inside source  addresses that should be translated.
* enable PAT with the ip nat inside source list ACL_NUMBER interface TYPE overload global configuration command.


Проведем настройку PAT для R15:

 Выставляем настройку "ip nat outside" на внешнем интерфейсе (175.192.168.6 ), а на внутренних выставляем "ip nat inside" 

````
interface Ethernet0/0
 ip address 10.20.3.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 ip address 10.20.6.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/2
 ip address 175.192.168.6 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.20.4.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet1/0
 ip address 10.20.7.2 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
````

Создаем ACL для внутренней сети офиса:

````
access-list 10 permit 10.20.0.0 0.0.255.255
````

Запускаем PAT:

````
ip nat inside source list 10 interface Ethernet0/2 overload
````


Проверка работы PAT путем ping с VPC1 Loopback0 роутера R21 провайдера Ламас.

````
VPCS> ping 10.3.2.1

84 bytes from 10.3.2.1 icmp_seq=1 ttl=252 time=1.893 ms
84 bytes from 10.3.2.1 icmp_seq=2 ttl=252 time=1.814 ms
84 bytes from 10.3.2.1 icmp_seq=3 ttl=252 time=1.922 ms
84 bytes from 10.3.2.1 icmp_seq=4 ttl=252 time=1.707 ms
84 bytes from 10.3.2.1 icmp_seq=5 ttl=252 time=1.533 ms

````

Проверем таблицу трансляции на R15:
````
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 175.192.168.6:101 10.20.10.10:101    10.3.2.1:101       10.3.2.1:101
icmp 175.192.168.6:357 10.20.10.10:357    10.3.2.1:357       10.3.2.1:357
icmp 175.192.168.6:613 10.20.10.10:613    10.3.2.1:613       10.3.2.1:613
icmp 175.192.168.6:869 10.20.10.10:869    10.3.2.1:869       10.3.2.1:869
icmp 175.192.168.6:1125 10.20.10.10:1125  10.3.2.1:1125      10.3.2.1:1125
icmp 175.192.168.6:1381 10.20.10.10:1381  10.3.2.1:1381      10.3.2.1:1381
icmp 175.192.168.6:1637 10.20.10.10:1637  10.3.2.1:1637      10.3.2.1:1637
icmp 175.192.168.6:1893 10.20.10.10:1893  10.3.2.1:1893      10.3.2.1:1893
icmp 175.192.168.6:2149 10.20.10.10:2149  10.3.2.1:2149      10.3.2.1:2149
icmp 175.192.168.6:2405 10.20.10.10:2405  10.3.2.1:2405      10.3.2.1:2405
....
````

Как видно, PAT работает корректно и внутренний адрес 10.20.10.10 (VPC1) транслируется во внешний 175.192.168.6 (R15) и обратно по технологии PAT.


Аналогично для R14:

````
interface Ethernet0/0
 ip address 10.20.2.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 ip address 10.20.5.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/2
 ip address 174.192.168.6 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.20.1.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet1/0
 ip address 10.20.7.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
````

````
access-list 10 permit 10.20.0.0 0.0.255.255
````

````
ip nat inside source list 10 interface Ethernet0/2 overload
````


#### 2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.

В целом, настройка PAT для R18 аналгичен настройке R14, R15 из первого пункта.

````
interface Ethernet0/0
 ip address 172.16.2.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1
 ip address 172.16.1.1 255.255.255.248
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/2
 ip address 176.192.168.14 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 176.192.168.18 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
````

````
access-list 10 permit 172.16.0.0 0.0.255.255
access-list 11 permit 172.16.0.0 0.0.255.255
````
````
ip nat inside source list 10 interface Ethernet0/2 overload
ip nat inside source list 11 interface Ethernet0/3 overload
````


Проверка ping с VPC8 также Loopback R21(Ламас) и таблицы трансляции R18:

````
VPCS> ping 10.3.2.1

84 bytes from 10.3.2.1 icmp_seq=1 ttl=251 time=1.227 ms
84 bytes from 10.3.2.1 icmp_seq=2 ttl=251 time=1.285 ms
84 bytes from 10.3.2.1 icmp_seq=3 ttl=251 time=2.426 ms
84 bytes from 10.3.2.1 icmp_seq=4 ttl=251 time=1.214 ms
84 bytes from 10.3.2.1 icmp_seq=5 ttl=251 time=1.301 ms

````

````
R18(config)#do sh ip nat translation
Pro Inside global      Inside local       Outside local      Outside global
icmp 176.192.168.14:2674 172.16.10.10:2674 10.3.2.1:2674     10.3.2.1:2674
icmp 176.192.168.14:2930 172.16.10.10:2930 10.3.2.1:2930     10.3.2.1:2930
icmp 176.192.168.14:3186 172.16.10.10:3186 10.3.2.1:3186     10.3.2.1:3186
icmp 176.192.168.14:3442 172.16.10.10:3442 10.3.2.1:3442     10.3.2.1:3442
icmp 176.192.168.14:3698 172.16.10.10:3698 10.3.2.1:3698     10.3.2.1:3698
icmp 176.192.168.14:9074 172.16.10.10:9074 10.3.2.1:9074     10.3.2.1:9074
icmp 176.192.168.14:9330 172.16.10.10:9330 10.3.2.1:9330     10.3.2.1:9330
icmp 176.192.168.14:9586 172.16.10.10:9586 10.3.2.1:9586     10.3.2.1:9586
icmp 176.192.168.14:9842 172.16.10.10:9842 10.3.2.1:9842     10.3.2.1:9842
icmp 176.192.168.14:10098 172.16.10.10:10098 10.3.2.1:10098  10.3.2.1:10098
````

Надо отметить, что теперь из-за особенности работы PAT непсоредственный доступ 
между хостами офисов Москвы и Санкт-Петербруг теперь невозможен (т.е. нельзя, например,
пропинговать VPC8 с VPC1), т.к. обе сети теперь находятся в полностью
серой зоне за пограничными роутерами и соединения с внешними хостами теперь могут инициироваться
только изнутри сети офиса, а снаружи хостам офиса могут приходить только ответы на них.


#### 3. Настроить статический NAT для R20.

Для задачи статического NAT в офисе Москва на роутере R15 создадим выделим 
пул внешних адресов 176.192.200.2 - 176.192.200.10.

````
R15#

ip nat pool static_nat 176.192.200.2 176.192.200.10 netmask 255.255.255.0
````
Далее настраиваем статический NAT для адреса R20 трансляции его адреса интерфейса во внешний IP:

````
ip nat inside source static 10.20.4.2 176.192.200.2
````

Далее, на роутере провайдера Ламас R21 указываем статический маршрут к новой 
сети 176.192.200.0/24 офиса Москва для возможности доступа извне к хостам офиса Москва через 
подключение к роутеру R15.

````
R21(config)#ip route 176.192.200.0 255.255.255.0 175.192.168.6
````

Проверяем ping 176.192.200.2, который транслируется в 10.20.4.2 (R20) с роутера R21(Ламас):

````
R21#ping 176.192.200.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.200.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
````

Пинг успешен, также смотрим таблицу NAT роутера R15:

````
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 176.192.200.2      10.20.4.2          ---                ---
````
Видим, что запись о статический трансляции присутствует.

Также в роутере R21 сделаем анонс новой сети 176.192.200.0/24 в BGP, 
чтобы любое устройство имело возможность иметь доступ к устройствам 
за статическим NAT офиса Москва.

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
....
 network 176.192.200.0 mask 255.255.255.0
....
````

#### 4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.


В роутере R15 добавляем статическую запись для Loopabck0 интерфеса R19(10.20.102.1/32):

````
R15(config)#ip nat inside source static 10.20.102.1 176.192.200.4

````

Проверяем доступность R19 с  R18 (пограничный роутер Санкт-Петербург)

````
R18>ping 176.192.200.4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.200.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms

````

Видим, что связность работает и на любых хостах, где обеспечивается "глобальная" передача маршрта 
по BGP, связность до R19 должна быть.


#### 5. Настроить статический NAT(PAT) для офиса Чокурдах.

Настроим PAT в офисе.

Настраиваем интерфейсы на роутере R28 входы и выход NAT. Учитываем, что в данном случае присутсвуют sub-интерфейсы, также необходимо два ACL,
так имеется два порта подключению к провайдеру.

````
interface Ethernet0/0
 ip address 176.192.168.22 255.255.255.252
 ip nat outside


interface Ethernet0/1
 ip address 176.192.168.26 255.255.255.252
 ip nat outside


interface Ethernet0/2.10
 description Default Gateway for VLAN 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface Ethernet0/2.20
 description Default Gateway for VLAN 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside

interface Ethernet0/2.200
 encapsulation dot1Q 200
 ip address 192.168.200.65 255.255.255.252
 ip nat inside

````

Определяем два ACL для внутренней сети для трансляции NAT:

````
access-list 10 permit 192.168.0.0 0.0.255.255
access-list 11 permit 192.168.0.0 0.0.255.255
````


И запускам PAT:

````
ip nat inside source list 10 interface Ethernet0/0 overload
ip nat inside source list 11 interface Ethernet0/1 overload
````

Проверим Ping от VPC30 для Loopabck0 R19(офис Москва) 176.192.200.4 (доступ через статический NAT в офоисе Москва):

````
VPCS> ping 176.192.200.4

84 bytes from 176.192.200.4 icmp_seq=1 ttl=249 time=3.676 ms
84 bytes from 176.192.200.4 icmp_seq=2 ttl=249 time=3.263 ms
84 bytes from 176.192.200.4 icmp_seq=3 ttl=249 time=2.136 ms
84 bytes from 176.192.200.4 icmp_seq=4 ttl=249 time=2.006 ms
84 bytes from 176.192.200.4 icmp_seq=5 ttl=249 time=2.077 ms

````

Таблица трансляции R28:
````
R28#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 176.192.168.22:1311 192.168.10.10:1311 176.192.200.4:1311 176.192.200.4:1311
icmp 176.192.168.22:1567 192.168.10.10:1567 176.192.200.4:1567 176.192.200.4:1567
icmp 176.192.168.22:1823 192.168.10.10:1823 176.192.200.4:1823 176.192.200.4:1823
icmp 176.192.168.22:2079 192.168.10.10:2079 176.192.200.4:2079 176.192.200.4:2079
icmp 176.192.168.22:2335 192.168.10.10:2335 176.192.200.4:2335 176.192.200.4:2335

````


#### 6. Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и  R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.

Стоить отметить, что уровень L3 в офисе фактически начинается на SW4,5, которые являеются L3 устройствами и также делают InteValn роутинг и их SVI 
явялются Default Gateway для конечных хостов.
В принципе, DHCP серевера для Vlan можно было запустить и на них, однако осуществим запуск DHCP на следующем уровне - уровень R12,R13, а через SW4 осуществим трансляцию
DHCP.

Настраиваем POOL для VLAN10 на R12:

````
R12#sh run | sec dhcp
ip dhcp excluded-address 10.20.10.1 10.20.10.9
ip dhcp pool VLAN-10
 network 10.20.10.0 255.255.255.0
 default-router 10.20.8.1
````
В качестве default-router указываем интерефейс в сторону SW4.

На SW4 для SVI Vlan10 настриваем ретрансляцию DHCP путем добавления ip helper-address :

````
interface Vlan10
 ip address 10.20.10.1 255.255.255.0
 ip helper-address 10.20.8.1
````

Переводим VPC1 в режим DHCP и проверяем получение адреса по DHCP:

````
VPCS> ip dhcp
DDORA IP 10.20.10.11/24 GW 10.20.8.1
````


````
R12#show ip dhcp  binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.20.10.11         0100.5079.6668.01       Feb 25 2025 08:32 AM    Automatic
````

Как видим, VPC1 получил адрес по DHCP в рамках адресного простанства VLAN10.

Аналогично все настроим для VLAN20.

````
R12#sh run | sec dhcp
...
ip dhcp excluded-address 10.20.20.1 10.20.20.9
...
ip dhcp pool VLAN-20
 network 10.20.20.0 255.255.255.0
 default-router 10.20.8.1
```

````
interface Vlan20
 ip address 10.20.20.1 255.255.255.0
 ip helper-address 10.20.8.1
````

DHCP VPC2:
````
VPCS> ip dhcp
DDORA IP 10.20.20.11/24 GW 10.20.8.1
````

````
R12#show ip dhcp  binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.20.10.11         0100.5079.6668.01       Feb 25 2025 08:32 AM    Automatic
10.20.20.11         0100.5079.6668.07       Feb 25 2025 08:49 AM    Automatic
````

#### 7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.


Для настройки текущего времени и его сохранения используем следующие команды:

````
R12#sh clock
*09:00:49.390 UTC Mon Feb 24 2025
````

````
R12#clock set 12:02:00 24 February 2025
R12#
*Feb 24 12:02:00.000: %SYS-6-CLOCKUPDATE:
 System clock has been updated from 09:02:01 UTC Mon Feb 24 2025 to 12:02:00 UTC Mon Feb 24 2025, 
 configured from console by console.
````

````
R12#clock update-calendar
````


Запускаем NTP серверы на роутерах R12,R13:

````
R12(config)#ntp master 3
````

Сервер запускаем на Loopback0, чтобы не привязываться к конкретному физ.интерфейсу.

````
interface Loopback0
 ip address 10.20.103.1 255.255.255.255
 ntp broadcast
````

Сразу проверем статус NTP:

````
R12#sh ntp status
Clock is synchronized, stratum 3, reference is 127.127.1.1
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 42400 (1/100 of seconds), resolution is 4000
reference time is EB67DEE6.7C28F718 (06:17:10.485 UTC Tue Feb 25 2025)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 939.58 msec, peer dispersion is 938.58 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 16, last update was 3 sec ago.

````

````
R13(config)#ntp master 3
````

````
interface Loopback0
 ip address 10.20.104.1 255.255.255.255
 ntp broadcast
````

````
R13#sh ntp status
Clock is synchronized, stratum 3, reference is 127.127.1.1
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 2700 (1/100 of seconds), resolution is 4000
reference time is EB67DF1E.BF7CEFA0 (06:18:06.748 UTC Tue Feb 25 2025)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 3939.43 msec, peer dispersion is 3938.29 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 16, last update was 11 sec ago.

````

Далее, на всех устройствах офиса Москва синхронизация должна осуществляться по сереверам NTP на базе R12, R13.

Для примера укажем роутеры R14,R15:

````
R14(config)#ntp server 10.20.103.1
R14(config)#ntp server 10.20.104.1
````

````
R14#show ntp associations
  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.20.103.1     127.127.1.1      3    415    128    60  0.000   0.000 71.065
+~10.20.104.1     127.127.1.1      3     16    128     3  0.000   0.000  2.851
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
````

````
R14#sh ntp status
Clock is synchronized, stratum 4, reference is 10.20.103.1
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 91800 (1/100 of seconds), resolution is 4000
reference time is EB67E0E3.E41895E8 (06:25:39.891 UTC Tue Feb 25 2025)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 192.96 msec, peer dispersion is 189.45 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000000014 s/s
system poll interval is 128, last update was 12 sec ago.

````

Для теста отключим ротуер R12 и проверим реакцию системы:

Через некоторе время система перешла на синхронизацию от ротурера R13:

````
R14>sh ntp  associations

  address         ref clock       st   when   poll reach  delay  offset   disp
 ~10.20.103.1     .INIT.          16      -     64     0  0.000   0.000 15937.
*~10.20.104.1     127.127.1.1      3      1     64     1  0.000   0.000 3938.9
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

````

````
R14>sh ntp status
Clock is synchronized, stratum 4, reference is 10.20.104.1
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 7500 (1/100 of seconds), resolution is 4000
reference time is EB67E38B.D6041AE0 (06:36:59.836 UTC Tue Feb 25 2025)
clock offset is 0.0000 msec, root delay is 1.00 msec
root dispersion is 7942.46 msec, peer dispersion is 189.52 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 64, last update was 11 sec ago.

````


Все остальные устройства настраиваются аналогично.



