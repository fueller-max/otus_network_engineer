### BGP Фильтрация 

### Цель:

Настроить фильтрацию для офисе Москва
Настроить фильтрацию для офисе С.-Петербург

###  Задание:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.

#### Общая информация:
-По умолчанию, BGP передает все префиксы BGP соседям. Это означает, что при наличии двух и более линков к провайдерам зона может стать транзитной.

Методы борьбы с транзитным трафиком:

* Filter-list with AS PATH access-list.
* No-Export Community.
* Prefix-list Filtering
* Distribute-list Filtering

### Решение:

1. [Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)](Readme.md#1-настроить-фильтрацию-в-офисе-москва-так-чтобы-не-появилось-транзитного-трафикаas-path)
2. [Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)](Readme.md#2--настроить-фильтрацию-в-офисе-с-петербург-так-чтобы-не-появилось-транзитного-трафикаprefix-list)

3. [Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию](Readme.md#3-настроить-провайдера-киторн-так-чтобы-в-офис-москва-отдавался-только-маршрут-по-умолчанию)

4. [Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург](Readme.md#4-настроить-провайдера-ламас-так-чтобы-в-офис-москва-отдавался-только-маршрут-по-умолчанию-и-префикс-офиса-с-петербург)

#### 1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).

Начнем с проверки маршрутов, которые получает роутер R22(провайдер Китрон) от роутера R14(офис Москва):

````
R22#show ip bgp neighbors 174.192.168.6 routes

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.0.0/16     174.192.168.6            0             0 1001 i
 *   172.16.100.1/32  174.192.168.6                          0 1001 301 520 2042 i
 *   175.192.168.0/24 174.192.168.6                          0 1001 301 i
 *   176.16.0.0       174.192.168.6                          0 1001 301 520 2042 i
 *   176.192.168.0/24 174.192.168.6                          0 1001 301 520 i
 r   176.192.168.4/30 174.192.168.6                          0 1001 301 520 i

Total number of prefixes 6
````

Как видно, роутер получает 6 префиксов, один из которых 10.20.0.0/16 относитеся к офису и должен передаваться, а остальные "транзитные" и их передавать не нужно.

Настроим фильтрацию на роутере R14 c использованием AS-PATH access-list.

````
R14(config)#ip as-path access-list 1 permit ^$

R14(config-router)#neighbor 174.192.168.5 filter-list 1 out
````

Проверяем теперь маршруты, которые получает роутер R22:

````
R22#show ip bgp neighbors 174.192.168.6 routes

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.0.0/16     174.192.168.6            0             0 1001 i

Total number of prefixes 1
````

Как видно, что теперь от офиса Москва передается только  внутренний префикс, а все внешние не передаются.

Суть метода зключается в использовании access-list в котором используется регулярное выражение ^$, что означает любую пустую строку. Соотвественно, роут будет передан только если AS-PATH отсутствует, т.к. только локальные префиксы.

Проделаем такую же процедуру в связке ротуреов R15(Москва) и R21(провайдер Ламас).

````
R15(config)#ip as-path access-list 1 permit ^$
R15(config-router)#neighbor 175.192.168.5 filter-list 1 out
````

````
R21#sh ip bgp neighbors 175.192.168.6 routes

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.0.0/16     175.192.168.6            0             0 1001 i

Total number of prefixes 1
````

Таким образом, через офис Москва будет исключен транзитный трафик.


#### 2.  Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).

Используем настройку prefix-list таким образом, чтобы фильтрацию проходил только префикс самого офиса и ничего более.

````
R18(config)#ip prefix-list NO-TRANSIT seq 5 permit 172.16.0.0/16
````

Назначаем на оба соседства R24/R25:
````
R18(config)#router bgp 2042
R18(config-router)# neighbor 176.192.168.13 prefix-list NO-TRANSIT out
R18(config-router)#neighbor 176.192.168.17 prefix-list NO-TRANSIT out

````

Проверяем маршруты, получаемые от роутера Санкт-Петербруг:
````
R24#sh ip bgp neighbors 176.192.168.14 routes

     Network          Next Hop            Metric LocPrf Weight Path
*>  172.16.0.0       176.192.168.14           0             0 2042 i

Total number of prefixes 1
````

````
R26#sh ip bgp neighbor 176.192.168.18 routes

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.0.0       176.192.168.18           0             0 2042 i

Total number of prefixes 1
````

Таким образом, через офис Санкт-Петербург будет исключен транзитный трафик.


#### 3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.


Проверим какие маршруты  сейчас отправляет Китрон в офис Москва(R14):

`````
R22#show ip  bgp neighbors 174.192.168.6 advertised-routes
BGP table version is 13, local router ID is 10.3.3.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.20.0.0/16     174.192.168.6            0             0 1001 i
 *>  174.192.168.0/24 0.0.0.0                  0         32768 i
 *>  175.192.168.0/24 174.192.168.10           0             0 301 i
 *>  176.16.0.0       176.192.168.5                          0 520 2042 i
 *>  176.192.168.0/24 176.192.168.5                          0 520 i
 r>  176.192.168.4/30 176.192.168.5            0             0 520 i
 *>  176.192.168.16/30
                       176.192.168.5                          0 520 i
 *>  176.192.168.20/30
                       176.192.168.5                          0 520 i
 *>  176.192.168.24/30
                       176.192.168.5                          0 520 i
 *>  176.192.168.28/30
                       176.192.168.5                          0 520 i

Total number of prefixes 10
`````



Теперь настроим передачу маршрута по умолчанию для R14:

````
R22(config-router)#neighbor 174.192.168.6 default-originate
````

Проверяем отдачу маршрута по умолчанию на R14: 
````
R22#show ip  bgp neighbors 174.192.168.6 advertised-routes

Originating default network 0.0.0.0

   ....

Total number of prefixes 9

````

Проверим таблицу маршрутизации на R14 и убедимся, что маршрут по умолчанию приходит с R22:

````
R14#sh ip route

Gateway of last resort is 174.192.168.5 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 174.192.168.5, 00:01:20
      10.0.0.0/8 is variably subnetted, 14 subnets, 3 masks

.....

````

Как видим, маршрут по умолчанию приходит и устанавливается в таблицу маршрутизации R14.

Теперь сделаем отсечение всех остальных маршрутов на R22 путем введения prefix-list и назначения его на соседство R14:

````
R22(config)#

ip prefix-list Only-Default permit 0.0.0.0/0

neighbor 174.192.168.6 prefix-list Only-Default out

````

Проверяем отдачу маршрутов R22 на R14:

````
R22#show ip  bgp neighbors 174.192.168.6 advertised-routes

Originating default network 0.0.0.0

     Network          Next Hop            Metric LocPrf Weight Path

Total number of prefixes 0
````

Как видно, не передаются никакие префиксы, кроме дефолтного.

Проверим полученные маршруты на R14:

````
R14#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          174.192.168.5                          0 101 i
 *>  10.20.0.0/16     0.0.0.0                  0         32768 i
 * i                  10.20.101.1              0    200      0 i
 r>i 174.192.168.0/24 10.20.101.1              0    200      0 301 101 i
 r>i 175.192.168.0/24 10.20.101.1              0    200      0 301 i
 r>i 176.16.0.0       10.20.101.1              0    200      0 301 520 2042 i
 r>i 176.192.168.0/24 10.20.101.1              0    200      0 301 520 i
 r>i 176.192.168.4/30 10.20.101.1              0    200      0 301 520 i
 r>i 176.192.168.16/30
                       10.20.101.1              0    200      0 301 520 i
 r>i 176.192.168.20/30
                       10.20.101.1              0    200      0 301 520 i
 r>i 176.192.168.24/30
                       10.20.101.1              0    200      0 301 520 i
 r>i 176.192.168.28/30
                       10.20.101.1              0    200      0 301 520 i

````

````
R14#sh ip route

Gateway of last resort is 174.192.168.5 to network 0.0.0.0
....

````

Как видно, в таблицу BGP R14 от Китрона(AS101) приходит только дефолтный маршрут, который потом устанавливается в основную таблицу маршрутизации.


#### 4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург

Подход будет аналогичным п.3 с разницей лишь настройки prefix-list c добавлением сети 172.16.0.0/16 Санкт-Петербурга

Приведем настройки:

Оргинация дефолтного маршрута для R15:

````
R21#sh run | sec bgp
router bgp 301
 bgp log-neighbor-changes
 network 175.192.168.0 mask 255.255.255.0
 neighbor 174.192.168.9 remote-as 101
 neighbor 175.192.168.6 remote-as 1001
 neighbor 175.192.168.6 default-originate
 neighbor 176.192.168.9 remote-as 520
````

Префикс лист для Москвского офиса: 
````
R21(config)#ip prefix-list Moscow-Office seq 5 permit 0.0.0.0/0
R21(config)#ip prefix-list Moscow-Office seq 10 permit 172.16.0.0/16
````

````
router bgp 301

neighbor 175.192.168.6 prefix-list Moscow-Office out
````

Проверка отдачи маршрутов:

````
R21#sh ip bgp neighbor 175.192.168.6 advertised-routes

Originating default network 0.0.0.0

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.0.0       176.192.168.9                          0 520 2042 i

Total number of prefixes 1

````

Проверка принимаемых маршрутов на R15:

````
R15#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          175.192.168.5                          0 301 i
 * i 10.20.0.0/16     10.20.100.1              0    150      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.0.0       175.192.168.5                          0 301 520 2042 i
````

Соотвественно, приходит маршрут по умолчанию и префикс офиса Санкт-Петербург, как и требовалось.



