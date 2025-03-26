### IPSec over DmVPN

### Цель:

* Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
* Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги



###  Задание:

1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург.
2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.

Для IPSec использовать CA и сертификаты

### Решение:

1.[Настроить GRE поверх IPSec между офисами Москва и С.-Петербург](Readmemd#1-настроить-gre-поверх-ipsec-между-офисами-москва-и-с-петербург)
2.[Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги](Readme.md#2-настроить-dmvpn-поверх-ipsec-между-москва-и-чокурдах-лабытнанги)

сылка на акутальную схему:
https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Lab04.drawio&dark=auto#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Ffueller-max%2Fdrawio%2Fmain%2FLab04.drawio 


#### 1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург

Во избежание конфликтов отключаем существующий не секюрный туннель GRE между офисами Москва и Санкт-Петербург.


В качестве сервера сертификации будем использовать роутер R23 провайдера Триада.

Меням имя хоста на СА, запускаем http сервер:

````
R23(config)#hostname CA
CA(config)#ip domain name labtech.su
CA(config)#http server
````

Генерируем пару rsa ключей с меткой СА:
````
CA(config)#crypto key generate rsa label CA modulus 2048 exportable
The name for the keys will be: CA

% The key modulus size is 2048 bits
% Generating 2048 bit RSA keys, keys will be exportable...
[OK] (elapsed time was 6 seconds)

````
Проверяем сгенерированные ключи:

````
CA(config)#do show crypto key mypubkey all
% Key pair was generated at: 07:18:02 UTC Mar 25 2025
Key name: CA
Key type: RSA KEYS
 Storage Device: not specified
 Usage: General Purpose Key
 Key is exportable.
 Key Data:
  30820122 300D0609 2A864886 F70D0101 01050003 82010F00 3082010A 02820101
  00B9EEB8 C422CCAD 28DADC83 25B1C078 9B8001F1 C2E0440F D659BC3B 382DF0FF
  0B8BD922 479E1F32 FFF8398D 2A6FD837 7581A149 3B230BBD 63CBB5DF 1D69E67B
  3C422047 A2FFB87E 1F35FEB0 81403D9F 03121092 EA388E38 8612F586 374534F4
  2417849E 4175E058 B954FFD3 71056E79 48D208DE DFAB3569 BA0B5839 65A71DE9
  7831E3AB 0682E34C 17021EC0 888DD53A C1EFD870 20FA7960 14D5FD9D 301774F1
  6FD949E8 3FF465F5 82477F29 3F996701 A68A0557 14468612 AC7FCFB9 3F45D04D
  C329A82D 830F2580 98DD30A6 0D30C88E 64EC91F3 D0FC11F1 9D201460 2C67ACA3
  52E0CEF2 BE9C74AB C7F426F4 E00A4555 73BF42B2 1DDD6D6D D6D8F471 7143AA17
  87020301 0001
% Key pair was generated at: 07:18:02 UTC Mar 25 2025
Key name: CA.server
Key type: RSA KEYS
Temporary key
 Usage: Encryption Key
 Key is not exportable.
 Key Data:
  307C300D 06092A86 4886F70D 01010105 00036B00 30680261 00D910F9 7F3606A5
  8623318B 1517A1A2 D9CCE60E 59D81847 11D2A7A6 6178823C 89D86811 849BCAD3
  75D3E784 3ABA49BF 509C5144 989CAA3A 817F1A34 D9E3B2B6 149ACE27 9372CC91
  C9E0DBA1 35C2275D 1BC8E040 2A4E2307 F176D4BF 96102BDF 1B020301 0001
````


Проверим дефолтные настройки PKI сервера(имя сервера должно совпадать с меткой rsa ключа):
````
CA(config)#crypto pki server CA
CA(cs-server)#show
 serial-number 0x0
 database level minimum
 no database archive
 issuer-name CN=CA
 lifetime crl 6
 lifetime certificate 365
 lifetime ca-certificate 1095
 lifetime enrollment-request 168
````

Включаем сервер:

````
CA(cs-server)#no shut
%Some server settings cannot be changed after CA certificate generation.
% Please enter a passphrase to protect the private key
% or type Return to exit

Password:

Re-enter password:

% Certificate Server enabled.

````

Выводим итоговую информацию по PKI серверу. Режим granting пока оставляем ручной, т.е. подписание сертификатов будем производить отдельно в ручную.
````
CA#sh crypto pki server
Certificate Server CA:
    Status: enabled
    State: enabled
    Server's configuration is locked  (enter "shut" to unlock it)
    Issuer name: CN=CA
    CA cert fingerprint: 7C35442C A207F111 4E12481D C964A5DA
    Granting mode is: manual
    Last certificate issued serial number (hex): 1
    CA certificate expiration timer: 07:24:39 UTC Mar 24 2028
    CRL NextUpdate timer: 13:24:39 UTC Mar 25 2025
    Current primary storage dir: nvram:
    Database Level: Complete - all issued certs written as <serialnum>.cer
````

На этом этапе создан и запущен СA сервер, к которму могут обращаться хосты для получения сертификатов.

Далее приступает к настройке R15(Москва) и R18(Санкт-Петербург), которые будут обращаться к R23 для получения сертификатов.

Задаем имя домена, а также резолвим имя СА сервера и проверям его доступность:

````
R15(config)#ip domain name labtech.su

R15(config)#ip host CA 10.60.100.1

R15(config)#do ping CA
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.60.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
````

Генерируем RSA ключ:
````
R15(config)#crypto key generate rsa
The name for the keys will be: R15.labtech.su
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...
[OK] (elapsed time was 1 seconds)
````


Создаем trustpoint и указываем url нашего CA центра.
````
R15(config)#crypto pki trustpoint CA
R15(ca-trustpoint)#enrollment url http://CA:80
````

Запросим сертификат CA и примем его:

````
R15(config)#crypto pki authenticate CA
Certificate has the following attributes:
       Fingerprint MD5: 7C35442C A207F111 4E12481D C964A5DA
      Fingerprint SHA1: 98DFD831 EFC32847 0EBD752E 58E96586 5C32E2DA

% Do you accept this certificate? [yes/no]: yes
Trustpoint CA certificate accepted.
````

Пробуем запросить сертификат для R15 у CA:

````
R15(config)#crypto pki enroll CA
%
% Start certificate enrollment ..
% Create a challenge password. You will need to verbally provide this
   password to the CA Administrator in order to revoke your certificate.
   For security reasons your password will not be saved in the configuration.
   Please make a note of it.

Password:
Re-enter password:

% The subject name in the certificate will include: R15.labtech.su
% Include the router serial number in the subject name? [yes/no]: yes
% The serial number in the certificate will be: 67109104
% Include an IP address in the subject name? [no]: no
Request certificate from CA? [yes/no]: yes
% Certificate request sent to Certificate Authority
% The 'show crypto pki certificate verbose CA' commandwill show the fingerprint.

R15(config)#
*Mar 25 08:00:53.677: CRYPTO_PKI:  Certificate Request Fingerprint MD5: B9245ACE 80B9A951 BB629CF8 18321CB8
*Mar 25 08:00:53.677: CRYPTO_PKI:  Certificate Request Fingerprint SHA1: 996EF54F 17D34B8A D3683015 0C1EA0F3 AC5BAF1D
````


На CA сервере видим запрос от R15, требующий действия по подписанию сертификата:

````
CA#sh crypto pki server CA requests
Enrollment Request Database:

Subordinate CA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------

RA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------

Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
1      pending    B9245ACE80B9A951BB629CF818321CB8 serialNumber=67109104+hostname=R15.labtech.su
````

Подписываем сертификат и проверяем состояние запросов:

````
CA#crypto pki server CA grant all
CA#sh crypto pki server CA requests

Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
1      granted    B9245ACE80B9A951BB629CF818321CB8 serialNumber=67109104+hostname=R15.labtech.su
````
видим, что серитфикат был успешно подписан.

Проверяем полученный сертификат на R15

````
R15#sh crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 02
  Certificate Usage: General Purpose
  Issuer:
    cn=CA
  Subject:
    Name: R15.labtech.su
    Serial Number: 67109104
    serialNumber=67109104+hostname=R15.labtech.su
  Validity Date:
    start date: 08:04:20 UTC Mar 25 2025
    end   date: 08:04:20 UTC Mar 25 2026
  Associated Trustpoints: CA

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer:
    cn=CA
  Subject:
    cn=CA
  Validity Date:
    start date: 07:24:39 UTC Mar 25 2025
    end   date: 07:24:39 UTC Mar 24 2028
  Associated Trustpoints: CA
````

Видим, что сертификат успешно получем сроком на 3 года.

Настройки для получения сертифката для R18 абсолютно аналогичные и для снижения общего объема информации приводить здесь их не будем.

После получения сертификатов можно перейти непосредственно к настройке IPsec и GRE между роутерами R15 и R18.

Настраиваем  параметры первой фазу(ассиметричное шифрование ):
````
R15(config)#crypto isakmp enable
R15(config)#crypto isakmp policy 10
R15(config-isakmp)#encryption aes
R15(config-isakmp)#hash sha256
R15(config-isakmp)#group 16
R15(config-isakmp)#lifetime 3600
````
Настройки 2ой фазы(симметричное шифрование):

````
R15(config)#crypto ipsec transform-set VTI ?
  ah-md5-hmac      AH-HMAC-MD5 transform
  ah-sha-hmac      AH-HMAC-SHA transform
  ah-sha256-hmac   AH-HMAC-SHA256 transform
  ah-sha384-hmac   AH-HMAC-SHA384 transform
  ah-sha512-hmac   AH-HMAC-SHA512 transform
  comp-lzs         IP Compression using the LZS compression algorithm
  esp-3des         ESP transform using 3DES(EDE) cipher (168 bits)
  esp-aes          ESP transform using AES cipher
  esp-des          ESP transform using DES cipher (56 bits)
  esp-gcm          ESP transform using GCM cipher
  esp-gmac         ESP transform using GMAC cipher
  esp-md5-hmac     ESP transform using HMAC-MD5 auth
  esp-null         ESP transform w/o cipher
  esp-seal         ESP transform using SEAL cipher (160 bits)
  esp-sha-hmac     ESP transform using HMAC-SHA auth
  esp-sha256-hmac  ESP transform using HMAC-SHA256 auth
  esp-sha384-hmac  ESP transform using HMAC-SHA384 auth
  esp-sha512-hmac  ESP transform using HMAC-SHA512 auth

R15(config)#crypto ipsec transform-set VTI esp-aes esp-sha-hmac
R15(cfg-crypto-trans)#mode transport
````
Здесь используем режим transport для более оптимального использования с GRE. 

Далее создаем профиль для его последующей привязке к туннелю GRE. Внутри профия привязывает созданный transform-set:

````
R15(config)#crypto ipsec profile Test
R15(ipsec-profile)#set transform-set VTI
````

Далее создаем непосредственно тунель GRE c оверлейной сетью 10.10.10.0\24:

````
R15(config)#int tun 10
R15(config-if)#tunnel source 175.192.168.6
R15(config-if)#tunnel destination 176.192.168.14
R15(config-if)#ip address 10.10.10.15 255.255.255.0
R15(config-if)#tunnel protection ipsec profile Test
R15(config-if)#
*Mar 25 09:04:47.579: %CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON
````
Аналогичные настройки делаем на R18 и проверяем доступность роутеров в рамках оврелейной сети:

````
R15#ping 10.10.10.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/6 ms
````
````
R18(config)#do ping 10.10.10.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.15, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/6 ms
````

Туннель поднят, теперь можно прописать статические маршруты до сетей офиса через туннельную сеть и проверим, что трафик зашифрован.

````
R15(config)#ip route 172.16.0.0 mask 0.0.255.255 10.10.10.18
````
````
R18(config)#ip route 10.20.0.0 255.255.0.0 10.10.10.15
````

При пинге R18 c R15 видим прохождение зашифрованного трафика между ними:

![](/Labs/Lab14_IPSec/pics/ping_via_ipsec.jpg)




#### 2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги

Для DmVPN будем использовать тот же СА.

Запросим сертификты от СА для ротуеров R27, R28.
Процедура полностью аналогичня описанной выше, выведем здесь только полученные сертификаты от СA для R27, R28:

````
R27#sh crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 04
  Certificate Usage: General Purpose
  Issuer:
    cn=CA
  Subject:
    Name: R27.labtech.su
    Serial Number: 67109296
    serialNumber=67109296+hostname=R27.labtech.su
  Validity Date:
    start date: 17:38:21 UTC Mar 25 2025
    end   date: 17:38:21 UTC Mar 25 2026
  Associated Trustpoints: CA

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer:
    cn=CA
  Subject:
    cn=CA
  Validity Date:
    start date: 07:24:39 UTC Mar 25 2025
    end   date: 07:24:39 UTC Mar 24 2028
  Associated Trustpoints: CA

````

````
R28(config)#do sh crypto pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 05
  Certificate Usage: General Purpose
  Issuer:
    cn=CA
  Subject:
    Name: R28.labtech.su
    Serial Number: 67109312
    serialNumber=67109312+hostname=R28.labtech.su
  Validity Date:
    start date: 17:44:09 UTC Mar 25 2025
    end   date: 17:44:09 UTC Mar 25 2026
  Associated Trustpoints: CA

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer:
    cn=CA
  Subject:
    cn=CA
  Validity Date:
    start date: 07:24:39 UTC Mar 25 2025
    end   date: 07:24:39 UTC Mar 24 2028
  Associated Trustpoints: CA

````

Также по аналогии создадим isakmp политику, transform set и профиль, который подвяжем к туннелю DmVPN. 

````
crypto isakmp policy 10
 encr aes
 hash sha256
 group 16
 lifetime 3600


 crypto ipsec transform-set VTI esp-aes esp-sha-hmac
 mode transport

 crypto ipsec profile Test
 set transform-set VTI
````

При поднятии туннеля видим, что соединением проходит стадию IKE:
````
R28#sh dmvpn

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 175.192.168.6       10.10.11.15   IKE 00:01:01     S

````
Далее оба тунеля R15 <-> R27,  R15 <-> R28 переходят в состояние UP:

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
     1 176.192.168.30      10.10.11.27    UP 00:00:40     D
     1 176.192.168.22      10.10.11.28    UP 00:00:45     D

````

Далее попробуем пропинговать R28 с R27. Видим, что также образовался доп. динамический туннель между R27 и R28:

````
R27#ping 10.10.11.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.11.28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/8/18 ms
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
     1 175.192.168.6       10.10.11.15    UP 00:10:33     S
     1 176.192.168.22      10.10.11.28    UP 00:00:02     D

````

А трафик идет напрямую между R27 и R28 (Spoke-to-Spoke) в зашифрованном виде:

![](/Labs/Lab14_IPSec/pics/ping%20R27_R28.jpg)


Таким образом DmVPN тунели работают поверх IPsec.

Конфигурации всех устройств размещены в папке configs.