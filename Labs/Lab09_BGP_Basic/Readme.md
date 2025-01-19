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
 network 176.192.160.0 mask 255.255.255.248
 neighbor 176.192.160.1 remote-as 101
````

* Конфигурация BGP роутера R15 офис Москва:

````
R15#router bgp 1001
 bgp log-neighbor-changes
 network 176.192.165.0 mask 255.255.255.248
 neighbor 176.192.165.1 remote-as 301
````

* Конфигурация BGP роутера R22 офис Китрон:

````
R22#sh run | sec bgp
router bgp 101
 bgp log-neighbor-changes
 network 10.3.3.1 mask 255.255.255.255
 network 176.192.150.0 mask 255.255.255.248
 network 176.192.160.0 mask 255.255.255.248
 network 176.192.168.0 mask 255.255.255.248
 neighbor 176.192.160.2 remote-as 1001
````

* Конфигурация BGP роутера R21 офис Ламас:

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 10.3.2.1 mask 255.255.255.255
 network 176.192.150.0 mask 255.255.255.248
 network 176.192.165.0 mask 255.255.255.248
 network 176.192.169.0 mask 255.255.255.248
 neighbor 176.192.165.2 remote-as 1001
````

#### Проверка состояний соседства BGP на обозначенных роутерах:

 * R14 офис Москва:
````
R14#show ip bgp neighbors | include BGP
BGP neighbor is 176.192.160.1,  remote AS 101, external link
  BGP version 4, remote router ID 10.3.3.1
  BGP state = Established, up for 01:38:03
  BGP table version 9, neighbor version 9/0
````

* R15 офис Москва:
````
R15#show ip bgp neighbors | include BGP
BGP neighbor is 176.192.165.1,  remote AS 301, external link
  BGP version 4, remote router ID 10.3.2.1
  BGP state = Established, up for 00:11:05
  BGP table version 10, neighbor version 10/0
````

* R22 офис Китрон:

````
R2#show ip bgp neighbors | include BGP
BGP neighbor is 176.192.160.2,  remote AS 1001, external link
  BGP version 4, remote router ID 10.20.100.1
  BGP state = Established, up for 01:41:03
  BGP table version 9, neighbor version 9/0
````

* R21 офис Ламас:
````
R21#show ip bgp neighbors | include BGP
BGP neighbor is 176.192.165.2,  remote AS 1001, external link
  BGP version 4, remote router ID 10.20.101.1
  BGP state = Established, up for 00:14:52
  BGP table version 10, neighbor version 10/0
````

* Проверка передачи префиксов между AS:

* R14 офис Москва:
````
R14#show ip bgp
BGP table version is 9, local router ID is 10.20.100.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.3.3.1/32      176.192.160.1            0             0 101 i
 *>  176.192.150.0/29 176.192.160.1            0             0 101 i
 r>  176.192.160.0/29 176.192.160.1            0             0 101 i
 *>  176.192.168.0/29 176.192.160.1            0             0 101 i
````

* R15 офис Москва:
````
R15#show ip bgp
BGP table version is 10, local router ID is 10.20.101.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.3.2.1/32      176.192.165.1            0             0 301 i
 *>  176.192.150.0/29 176.192.165.1            0             0 301 i
 *>  176.192.165.0/29 0.0.0.0                  0         32768 i
 *                    176.192.165.1            0             0 301 i
 *>  176.192.169.0/29 176.192.165.1            0             0 301 i
````

* R22 офис Китрон:
````
R22#show ip bgp
BGP table version is 9, local router ID is 10.3.3.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.3.3.1/32      0.0.0.0                  0         32768 i
 *>                   0.0.0.0                  0         32768 i
 *   176.192.160.0/29 176.192.160.2            0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *>  176.192.168.0/29 0.0.0.0                  0         32768 i
````

* R21 офис Ламас:
````
R21#show ip bgp
BGP table version is 10, local router ID is 10.3.2.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.3.2.1/32      0.0.0.0                  0         32768 i
 *>  176.192.150.0/29 0.0.0.0                  0         32768 i
 *   176.192.165.0/29 176.192.165.2            0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *>                   0.0.0.0                  0         32768 i
````

Таким образом, соседство между пограничными роутерам офиса Москва и провайдерами Китрон и Ламас BGP установлено, а также осуществляется передача префксов между AS.


#### 2. Настройка eBGP между провайдерами Киторн и Ламас

Для организации eBGP между Китрон(R22) и Ламас(21) добавим в уже существующие конфигурации IP адреса и номера AS соседства.

Конфигурации роутеров будут выгляеть следующим образом:

* R22 офис Китрон:
````
R22#sh run | sec bgp
router bgp 101
 bgp log-neighbor-changes
 network 10.3.3.1 mask 255.255.255.255
 network 176.192.150.0 mask 255.255.255.248
 network 176.192.160.0 mask 255.255.255.248
 network 176.192.168.0 mask 255.255.255.248
 neighbor 176.192.150.2 remote-as 301
 neighbor 176.192.160.2 remote-as 1001
````

* R21 офис Ламас:
```
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 10.3.2.1 mask 255.255.255.255
 network 176.192.150.0 mask 255.255.255.248
 network 176.192.165.0 mask 255.255.255.248
 network 176.192.169.0 mask 255.255.255.248
 neighbor 176.192.150.1 remote-as 101
 neighbor 176.192.165.2 remote-as 1001
```

Соотвественно, для роутера R22 добавили AS систему (301) Ламаса, а для R21 добавили AS систему (101) Китрона.

Для объективной краткости на данном этапе не будем выводить состояние соседства.

#### 3. Настроика eBGP между Ламас и Триада.

Для настройка eBGP между Ламас и Триада, в роутере R21(Ламас) добавим к существующей конфигруации соседство и AS систему с номером 520:

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 10.3.2.1 mask 255.255.255.255
 network 176.192.150.0 mask 255.255.255.248
 network 176.192.165.0 mask 255.255.255.248
 network 176.192.169.0 mask 255.255.255.248
 neighbor 176.192.150.1 remote-as 101
 neighbor 176.192.165.2 remote-as 1001
 neighbor 176.192.169.1 remote-as 520
````

В роутере R24(Триада) запустим BGP с номером AS520 и укажем соседство с роутером Ламас(AS301):
````
R24#sh run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.169.0 mask 255.255.255.248
 network 176.192.170.0 mask 255.255.255.248
 neighbor 176.192.169.2 remote-as 301
````

Также для объективной краткости на данном этапе не будем выводить состояние соседства.

#### 4. Настройка eBGP между офисом С.-Петербург и провайдером Триада.

На пограничном роутере R18 запустим процесс BGP с номером AS 2042 и соседством с роутерами R24 и R26.

* Роутер R18 офис Санкт-Петербург:
````
R18#sh run | sec bgp
router bgp 2042
 bgp log-neighbor-changes
 network 176.192.170.0 mask 255.255.255.248
 network 176.192.171.0 mask 255.255.255.248
 neighbor 176.192.170.1 remote-as 520
 neighbor 176.192.171.1 remote-as 520
````

Дополним настройку R24(офис Триада):
````
R24#sh run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.169.0 mask 255.255.255.248
 network 176.192.170.0 mask 255.255.255.248
 neighbor 176.192.169.2 remote-as 301
 neighbor 176.192.170.2 remote-as 2042
````

Также запустим BGP на роутере R26(офис Триада):
````
R26#sh run | sec bgp
router bgp 520
 bgp log-neighbor-changes
 network 176.192.171.0 mask 255.255.255.248
 network 176.192.172.0 mask 255.255.255.248
 neighbor 176.192.171.2 remote-as 2042
````

#### 5. Организация IP доступности между пограничным роутерами офисами Москва и С.-Петербург.

В принципе, после приведенных настроек должна быть обеспечена доступность пограчничных роутеров между офисами Москва и Санкт-Петербург.

Выведем BGP маршруты на настроенных роутерах:

* R14 офис Москва:
 ````
 R14>sh ip route bgp

Gateway of last resort is 176.192.160.1 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
B        10.3.2.1/32 [20/0] via 176.192.160.1, 01:48:03
B        10.3.3.1/32 [20/0] via 176.192.160.1, 02:18:53
      176.192.0.0/16 is variably subnetted, 7 subnets, 2 masks
B        176.192.150.0/29 [20/0] via 176.192.160.1, 02:19:23
B        176.192.165.0/29 [20/0] via 176.192.160.1, 01:49:09
B        176.192.168.0/29 [20/0] via 176.192.160.1, 02:19:23
B        176.192.169.0/29 [20/0] via 176.192.160.1, 01:48:38
B        176.192.170.0/29 [20/0] via 176.192.160.1, 01:33:23
 ````

* R15 офис Москва:

````
R15#sh ip route bgp

Gateway of last resort is 176.192.165.1 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 14 subnets, 2 masks
B        10.3.2.1/32 [20/0] via 176.192.165.1, 01:04:27
B        10.3.3.1/32 [20/0] via 176.192.165.1, 01:04:27
      176.192.0.0/16 is variably subnetted, 7 subnets, 2 masks
B        176.192.150.0/29 [20/0] via 176.192.165.1, 01:04:27
B        176.192.160.0/29 [20/0] via 176.192.165.1, 01:04:27
B        176.192.168.0/29 [20/0] via 176.192.165.1, 01:04:27
B        176.192.169.0/29 [20/0] via 176.192.165.1, 01:04:27
B        176.192.170.0/29 [20/0] via 176.192.165.1, 01:04:27

````

* R22 (офис Китрон)
````
R22#sh ip route bgp

Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 2 subnets
B        10.3.2.1 [20/0] via 176.192.150.2, 01:50:34
      176.192.0.0/16 is variably subnetted, 9 subnets, 2 masks
B        176.192.165.0/29 [20/0] via 176.192.150.2, 01:51:40
B        176.192.169.0/29 [20/0] via 176.192.150.2, 01:51:09
B        176.192.170.0/29 [20/0] via 176.192.150.2, 01:35:54

````

* R21(офис Ламас):
````
R21#sh ip route bgp
Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 2 subnets
B        10.3.3.1 [20/0] via 176.192.150.1, 01:55:25
      176.192.0.0/16 is variably subnetted, 9 subnets, 2 masks
B        176.192.160.0/29 [20/0] via 176.192.150.1, 01:55:25
B        176.192.168.0/29 [20/0] via 176.192.150.1, 01:55:25
B        176.192.170.0/29 [20/0] via 176.192.169.1, 01:37:47
````

* R24 (офис Триада):
````
R24#sh ip route bgp

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks
B        10.3.2.1/32 [20/0] via 176.192.169.2, 01:39:56
B        10.3.3.1/32 [20/0] via 176.192.169.2, 01:39:56
      176.192.0.0/16 is variably subnetted, 8 subnets, 2 masks
B        176.192.150.0/29 [20/0] via 176.192.169.2, 01:39:56
B        176.192.160.0/29 [20/0] via 176.192.169.2, 01:39:56
B        176.192.165.0/29 [20/0] via 176.192.169.2, 01:39:56
B        176.192.168.0/29 [20/0] via 176.192.169.2, 01:39:56
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

Gateway of last resort is 176.192.170.1 to network 0.0.0.0

      10.0.0.0/32 is subnetted, 2 subnets
B        10.3.2.1 [20/0] via 176.192.170.1, 01:40:03
B        10.3.3.1 [20/0] via 176.192.170.1, 01:40:03
      176.192.0.0/16 is variably subnetted, 10 subnets, 2 masks
B        176.192.150.0/29 [20/0] via 176.192.170.1, 01:40:03
B        176.192.160.0/29 [20/0] via 176.192.170.1, 01:40:03
B        176.192.165.0/29 [20/0] via 176.192.170.1, 01:40:03
B        176.192.168.0/29 [20/0] via 176.192.170.1, 01:40:03
B        176.192.169.0/29 [20/0] via 176.192.170.1, 01:40:03
B        176.192.172.0/29 [20/0] via 176.192.171.1, 01:32:38
````


Проверям связность между пограничными роутерами офисов Москва и Санкт-Петербург:

Связность между R14(Москва) и R18(Петербург):
````
R14#ping 176.192.170.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.170.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
````

Связность между R15(Москва) и R18(Петербург):
````
R15#ping 176.192.170.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.170.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Связность между R18(Петербург) и R14,R15(Москва):
````
R18#ping 176.192.160.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.160.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18#ping 176.192.165.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.165.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
````

Как видно, связность между пограничными роутерами обеспечена.

Актуальные конфигруации роутеров размещены в папке confgis.


