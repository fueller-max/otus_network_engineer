#### VPN. GRE. DmVPN

### Цель:

Настроить GRE между офисами Москва и С.-Петербург
Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

###  Задание:

1. Настроить GRE между офисами Москва и С.-Петербург.
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.

### Решение:

1. [Настроить GRE между офисами Москва и С.-Петербург](Readme.md#1-настроить-gre-между-офисами-москва-и-с-петербург)
2. [Настроить DMVMN между Москва и Чокурдах, Лабытнанги](Readme.md#2-настроить-dmvmn-между-москва-и-чокурдах-лабытнанги)



#### 1. Настроить GRE между офисами Москва и С.-Петербург

Общая информация:

GRE (Generic Routing Encapsulation) – протокол инкапсуляции, который широко применяется как сам по себе, так и в совокупности с IPSec для создания туннелей. 
Сам по себе GRE не обеспечивает никакой безопасности передаваемых данных – это site-to-site VPN туннель в чистом виде.

Протокол GRE инкапсулируется напрямую в IP, минуя TCP или UDP, в IP пакете есть специальное поле «Protocol type», в котором содержится число, обозначающее протокол, инкапсулированный в данный IP пакет, для GRE protocol type равен 47.

Настроим туннель GRE между пограничными ротутерами R15(Москва) и R18(Санкт-Петербург)

Для начала проверям IP связность между ротутерам R15 и R18:

````
R15#ping 176.192.168.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 176.192.168.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

````
````
R18#ping 175.192.168.6
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 175.192.168.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Связность посредством двух провайдеров между роутерами присутсвует. 

Настроим поверх этого туннель GRE, используя сеть GRE 10.1.1.0/24

```
R15(config)#interface tunnel 10
R15(config-if)#tunnel mode gre ip
R15(config-if)#ip address 10.1.1.1 255.255.255.0
R15(config-if)#tunnel source e0/2
R15(config-if)#tunnel destination 176.192.168.14

```

````R18(config)#interface tunnel 10
R18(config-if)#tunnel mode gre ip
R18(config-if)#ip address 10.1.1.3 255.255.255.0
R18(config-if)#tunnel source e0/2
R18(config-if)#tunnel destination 175.192.168.6
````

Проверяем состояние туннеля на R15:
````
R15#sh int tunnel 10
Tunnel10 is up, line protocol is up
  Hardware is Tunnel
  Internet address is 10.1.1.1/24
  MTU 17916 bytes, BW 100 Kbit/sec, DLY 50000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel linestate evaluation up
  Tunnel source 175.192.168.6 (Ethernet0/2), destination 176.192.168.14
````

Припингуем R18 по егo IP в тоннеле:  

````
R15#ping 10.1.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Сделаем также traceroute и убедимся, в туннеле есть непосредственная связь между хостами, хотя факитически трафик проходит через двух провайдеров:
````
R15#traceroute 10.1.1.3
Type escape sequence to abort.
Tracing the route to 10.1.1.3
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.1.3 [AS 301] 1 msec 1 msec *
````

На этм базовая настройка самого туннеля GRE завершена. 

Далее настроим статические маршртуы до сетей офисов в R15 и К18 в настроенный туннель, чтобы обеспечить связность между сетями офисов: 

````
R15#
ip route 172.16.0.0 255.255.0.0 Tunnel10
````

````
R18#
ip route 10.20.0.0 255.255.0.0 Tunnel10
````


Пингуем Loopback интерфейс пограничного ротера R18 c R15:
````
R15#ping 172.16.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Пингуем Loopback интерфейс пограничного ротера R15 c R18:
````
R18#ping 10.20.101.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.101.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Для теста также попробуем пропинговать VPC8 офиса Санкт-Петербурга с R15:

````
R15#ping 172.16.10.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.10.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

как видим трафик внтуренних сетей проходит через туннель GRE.

#### 2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги

Общая информация:

Dynamic Multipoint VPN (DMVPN) — виртуальная частная сеть с возможностью динамического создания туннелей между узлами.

В качесте hub-маршрутизатора будем использовать роутер R15 -Москва,
в качесте spoke роутеров будут выступать роутеры R28(Чокурдах) и R27(Лабытынаги).

Будем использовать Phase 2 для обеспечения работы Spoke-to-Spoke напрямую.

Для адресации туннеля выделяем сеть 10.10.10.0/24

Настроим mGRE + NHRP на R15:

````
R15(config)#int Tunnel20
R15(config-if)#ip address 10.10.10.1 255.255.255.0
R15(config-if)#ip mtu 1416
R15(config-if)#tunnel source e0/2
R15(config-if)#tunnel mode gre multipoint
R15(config-if)#tunnel key 999
R15(config-if)#ip nhrp authentication DMVPN
R15(config-if)#ip nhrp map multicast dynamic
R15(config-if)#ip nhrp network-id 1

````

* tunnel mode: by default the tunnel mode will be point-to-point GRE, we require a multipoint interface on the hub.
* tunnel source: the tunnel destinations will be dynamic but we  have to configure the source, our e0/2 interface.
* ip nhrp authentication: to authenticate  NHRP traffic, it’s optional but a good idea to enable. Use  “DMVPN”.
* ip nhrp map multicast dynamic:  this command tells the hub router where to forward multicast packets to. Since the IP addresses of the spoke routers are unknown, we use dynamic to automatically add their IP addresses to the multicast destination list when the spokes register themselves.
* ip nhrp network-id: when you use multiple DMVPN networks, you need the network ID to differentiate between the two networks. This value is only locally significant but for troubleshooting reasons it’s best to use the same value on all routers.


Далее настроим первый spoke роутер R28(Чокурдах):

````
R28(config-if)#ip address 10.10.10.10 255.255.255.0
R28(config-if)#ip nhrp authentication DMVPN
R28(config-if)#ip nhrp map 10.10.10.1 175.192.168.6
R28(config-if)#ip nhrp map multicast 175.192.168.6
R28(config-if)#ip nhrp network-id 1
R28(config-if)#ip nhrp nhs 10.10.10.1
R28(config-if)#tunnel source e0/0
R28(config-if)#tunnel mode gre multipoint
````

* ip nhrp map: we use this on the spoke to create a static mapping for the hub’s tunnel address (10.10.10.1) and the hub’s NBMA address (175.192.168.6). This will be stored in the NHRP cache of the spoke router.
* ip nhrp map multicast: here we specify which destinations should receive broadcast or multicast traffic through the tunnel interface. The confusing part is that you have to enter the NBMA address here. We need this command since routing protocols like RIP, EIGRP and OSPF require multicast.
* ip nhrp nhs: this is where we specify the NHRP server, our hub router.


По аналогии настроим второй spoke R27(Лабытынаги):

````
interface Tunnel20
 ip address 10.10.10.11 255.255.255.0
 ip nhrp authentication DMVPN
 ip nhrp map 10.10.10.1 175.192.168.6
 ip nhrp map multicast 175.192.168.6
 ip nhrp network-id 1
 ip nhrp nhs 10.10.10.1
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
````

Проверяем состояние туннеля dmvpn и nhrp на hub-роутере R15:

````
R15#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel20, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 176.192.168.22      10.10.10.10    UP 00:12:52     D
     1 176.192.168.30      10.10.10.11    UP 00:12:57     D

````

````
R15#sh ip nhrp
10.10.10.10/32 via 10.10.10.10
   Tunnel20 created 00:15:18, expire 01:55:53
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 176.192.168.22
10.10.10.11/32 via 10.10.10.11
   Tunnel20 created 00:15:23, expire 01:46:17
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 176.192.168.30

````

Проверяем состояние туннеля dmvpn и nhrp на spoke-роутере R28:

````
R28#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 175.192.168.6        10.10.10.1  NHRP 00:00:56     S

````
````
R28#sh ip nhrp
10.10.10.1/32 via 10.10.10.1
   Tunnel20 created 00:00:26, never expire
   Type: static, Flags:
   NBMA address: 175.192.168.6
````


Как видно, динамические туннели между hub-роутром с обоими spoke-роутерами поднялись.

Далее, для работы DMVPN настроим также EIGRP.


Для hub-router R15:

````
router eigrp 20
 network 10.10.10.0 0.0.0.255
````


Отключаем суммаризация маршрутов и подмену next-hop для обеспечения работы фазы 2 (непосредственная коммуникация между spoke ами):

````
interface Tunnel20
 no ip next-hop-self eigrp 20
 no ip split-horizon eigrp 20
````

R27, R28 с анонсированием  их loopback`ов:

````
R27#sh run | sec eigrp
router eigrp 20
 network 10.10.10.0 0.0.0.255
 network 10.3.1.1 0.0.0.0
````

````
R28#sh run | sec eigrp
router eigrp 20
 network 10.10.10.0 0.0.0.255
 network 192.168.100.1 0.0.0.0
````


Проверим состояние EIGRP на hub-роутере:

````
R15#sh ip eigrp neighbors
EIGRP-IPv4 Neighbors for AS(20)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.10.10.10             Tu20                     11 00:06:09    5  1374  0  1
0   10.10.10.11             Tu20                     12 00:07:20    1  2124  0  1
````

Работе команды trace  до loopback R28 c роутера R27
````
R27#trace 192.168.100.1
````

Можно наблюдать как появился отдельный  динамический маршрут между R27 и R28:

````
R27#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 175.192.168.6        10.10.10.1    UP 00:06:23     S
     1 176.192.168.22      10.10.10.10    UP 00:05:02     D
````


````
R28#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 175.192.168.6        10.10.10.1    UP 00:11:12     S
     1 176.192.168.30      10.10.10.11    UP 00:01:56     D

````

Небольшим недостатком явялется то, что первый запрос будет проходить через hub, а потом уже установится динамический туннель и трафик пойдет без участия hub роутера.

Для сетей малого и среднего размера - это не проблема, для сетей крупного масштаба можно использовать Phase 3 для устранения данного недостатка.

Актуальные конфигурации устройств находятся в папке configs.

