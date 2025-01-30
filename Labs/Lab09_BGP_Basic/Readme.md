### BGP Basic

### Цель:
Настроить BGP между автономными системами
Организовать доступность между офисами Москва и С.-Петербург


###  Задание:

1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроить eBGP между провайдерами Киторн и Ламас.
3. Настроить eBGP между Ламас и Триада.
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.
5. Организовать IP доступность между пограничным роутерами офисами Москва и С.-Петербург.


###  Решение:

Фрагмент интересующей нас схемы с указанием номеров AS систем представлен ниже:
![](/Labs/Lab09_BGP_Basic/pics/IP_Plan_AS.jpg)

1. [Настройка eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас](Readme.md#1-настройка-ebgp-между-офисом-москва-и-двумя-провайдерами---киторн-и-ламас)

2. [Настройка eBGP между провайдерами Киторн и Ламас](Readme.md#2-настройка-ebgp-между-провайдерами-киторн-и-ламас)

3. [Настройка eBGP между Ламас и Триада](Readme.md#3-настроика-ebgp-между-ламас-и-триада)

4. [Настройка eBGP между офисом С.-Петербург и провайдером Триада](Readme.md#4-настройка-ebgp-между-офисом-с-петербург-и-провайдером-триада)

5. [Организация IP доступности между пограничным роутерами офисами Москва и С.-Петербург](Readme.md#5-организация-ip-доступности-между-пограничным-роутерами-офисами-москва-и-с-петербург)
#### 1. Настройка eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас

* Конфигурация BGP роутера R14 офис Москва:

````
R14#sh run | sec bgp
router bgp 1001
 bgp log-neighbor-changes
 neighbor 174.192.168.5 remote-as 101
````

* Конфигурация BGP роутера R15 офис Москва:

````
R15#sh run | sec bgp
router bgp 1001
 bgp log-neighbor-changes
 neighbor 175.192.168.5 remote-as 301
````

* Конфигурация BGP роутера R22 офис Китрон:

````
R22#sh run | sec bgp
router bgp 101
 bgp log-neighbor-changes
 network 174.192.168.0 mask 255.255.255.0
 neighbor 174.192.168.6 remote-as 1001
 neighbor 174.192.168.10 remote-as 301
````

* Конфигурация BGP роутера R21 офис Ламас:

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 175.192.168.0 mask 255.255.255.0
 neighbor 174.192.168.9 remote-as 101
 neighbor 175.192.168.6 remote-as 1001

````

Дополнительно на роутерах R22 и R21 прописываем пути для сетей:
 * 174.192.168.0/24 на роутере R22
 * 175.192.168.0/24 на роутере R21

до Null0 для того, чтобы эти сети рекламировались по eBGP.

````
R21#sh ip route
S        175.192.168.0/24 is directly connected, Null0
````

````
R22#sh ip route
S        174.192.168.0/24 is directly connected, Null0
````

#### Проверка передачи префиксов между AS на обозначенных роутерах:



* R14 офис Москва:
````
R14#sh ip bgp
BGP table version is 4, local router ID is 10.20.100.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  174.192.168.0/24 174.192.168.5            0             0 101 i
 *>  175.192.168.0/24 174.192.168.5                          0 101 301 i

````

* R15 офис Москва:
````
R15#sh ip bgp
BGP table version is 4, local router ID is 10.20.101.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  174.192.168.0/24 175.192.168.5                          0 301 101 i
 *>  175.192.168.0/24 175.192.168.5            0             0 301 i

````

* R22 офис Китрон:
````
R22#sh ip bgp
BGP table version is 4, local router ID is 10.3.3.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  174.192.168.0/24 0.0.0.0                  0         32768 i
 *>  175.192.168.0/24 174.192.168.10           0             0 301 i
````

* R21 офис Ламас:
````
R21#sh ip bgp
BGP table version is 4, local router ID is 10.3.2.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  174.192.168.0/24 174.192.168.9            0             0 101 i
 *>  175.192.168.0/24 0.0.0.0                  0         32768 i
````

Таким образом, соседство между пограничными роутерам офиса Москва и провайдерами Китрон и Ламас BGP установлено, а также осуществляется передача префксов между AS.


#### 2. Настройка eBGP между провайдерами Киторн и Ламас

Настройка eBGP между Киторн и Ламас выполнена выше в п.1

#### 3. Настроика eBGP между Ламас и Триада.

Для настройка eBGP между Ламас и Триада, в роутере R21(Ламас) добавим к существующей конфигруации соседство и AS систему с номером 520:

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 175.192.168.0 mask 255.255.255.0
 neighbor 174.192.168.9 remote-as 101
 neighbor 175.192.168.6 remote-as 1001
 neighbor 176.192.168.9 remote-as 520
````

В роутере R24(Триада) запустим BGP с номером AS520 и укажем соседство с роутером Ламас(AS301):
````
R24#sh run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.168.0 mask 255.255.255.0
 neighbor 176.192.168.10 remote-as 301
````

Также в роутере R24 задаем статический путь для сети 176.192.168/24 в Null0

````
R24#sh ip route
S        176.192.168.0/24 is directly connected, Null0
````



#### 4. Настройка eBGP между офисом С.-Петербург и провайдером Триада.

На пограничном роутере R18 запустим процесс BGP с номером AS 2042 и соседством с роутерами R24 и R26.

* Роутер R18 офис Санкт-Петербург:

````
R18#sh run | sec bgp
router bgp 2042
 bgp log-neighbor-changes
 neighbor 176.192.168.13 remote-as 520
 neighbor 176.192.168.17 remote-as 520
````

Дополним настройку R24(офис Триада):

````
R24#sh run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.168.0 mask 255.255.255.0
 neighbor 176.192.168.10 remote-as 301
 neighbor 176.192.168.14 remote-as 2042
````

Также запустим BGP на роутере R26(офис Триада):
````
R26#sh  run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.168.0 mask 255.255.255.0
 neighbor 176.192.168.18 remote-as 2042
````

с добавлением также маршрута 176.192.168/24 в Null0 

````
R26#sh ip route
S        176.192.168.0/24 is directly connected, Null0
````

#### 5. Организация IP доступности между пограничным роутерами офисами Москва и С.-Петербург.

В принципе, после приведенных настроек должна быть обеспечена доступность пограчничных роутеров между офисами Москва и Санкт-Петербург.

Выведем BGP маршруты на настроенных роутерах:

* R14 офис Москва:
 ````
 R14>sh ip route bgp

      174.192.0.0/16 is variably subnetted, 3 subnets, 3 masks
B        174.192.168.0/24 [20/0] via 174.192.168.5, 00:49:47
      175.192.0.0/24 is subnetted, 1 subnets
B        175.192.168.0 [20/0] via 174.192.168.5, 00:49:16
      176.192.0.0/24 is subnetted, 1 subnets
B        176.192.168.0 [20/0] via 174.192.168.5, 00:36:35
 ````

* R15 офис Москва:

````
R15#sh ip route bgp

      174.192.0.0/24 is subnetted, 1 subnets
B        174.192.168.0 [20/0] via 175.192.168.5, 00:50:35
      175.192.0.0/16 is variably subnetted, 3 subnets, 3 masks
B        175.192.168.0/24 [20/0] via 175.192.168.5, 00:50:35
      176.192.0.0/24 is subnetted, 1 subnets
B        176.192.168.0 [20/0] via 175.192.168.5, 00:37:37

````

* R22 (офис Китрон)
````
R22#sh ip route bgp

      175.192.0.0/24 is subnetted, 1 subnets
B        175.192.168.0 [20/0] via 174.192.168.10, 00:51:13
      176.192.0.0/16 is variably subnetted, 3 subnets, 3 masks
B        176.192.168.0/24 [20/0] via 174.192.168.10, 00:38:15

````

* R21(офис Ламас):
````
R21#sh ip route bgp

      174.192.0.0/16 is variably subnetted, 3 subnets, 3 masks
B        174.192.168.0/24 [20/0] via 174.192.168.9, 00:51:49
      176.192.0.0/16 is variably subnetted, 3 subnets, 3 masks
B        176.192.168.0/24 [20/0] via 176.192.168.9, 00:38:51
````

* R24 (офис Триада):
````
R24#sh ip route bgp

      174.192.0.0/24 is subnetted, 1 subnets
B        174.192.168.0 [20/0] via 176.192.168.10, 00:42:37
      175.192.0.0/24 is subnetted, 1 subnets
B        175.192.168.0 [20/0] via 176.192.168.10, 00:42:37

````

* R26 (офис Триада):
````
R26#sh ip route bgp

Gateway of last resort is not set

      176.192.0.0/16 is variably subnetted, 5 subnets, 2 masks
B        176.192.170.0/29 [20/0] via 176.192.171.2, 00:01:19

````

* R18 (офис Санкт-Петербург):
````
R18#sh ip route bgp

      174.192.0.0/24 is subnetted, 1 subnets
B        174.192.168.0 [20/0] via 176.192.168.13, 00:00:37
      175.192.0.0/24 is subnetted, 1 subnets
B        175.192.168.0 [20/0] via 176.192.168.13, 00:00:37
      176.192.0.0/16 is variably subnetted, 5 subnets, 3 masks
B        176.192.168.0/24 [20/0] via 176.192.168.13, 00:00:37
````


Проверям связность между пограничными роутерами офисов Москва и Санкт-Петербург:

Связность между R14(Москва) и R18(Петербург):
````
R14#ping 176.192.168.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.168.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

````

Связность между R15(Москва) и R18(Петербург):
````
R15#ping 176.192.168.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.168.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

````

Связность между R18(Петербург) и R14,R15(Москва):
````
R18#ping 174.192.168.6
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 174.192.168.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18#ping 175.192.168.6
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 175.192.168.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

````

Как видно, связность между пограничными роутерами обеспечена.

Актуальные конфигруации роутеров размещены в папке confgis.


