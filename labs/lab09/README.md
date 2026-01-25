
## BGP. Основы

### Цель
Настроить eBGP/iBGP IPv6 unicast для всех сегментов сети по аналогичной логике с настройкой eBGP/iBGP IPv4 unicast.<br>

В этой  самостоятельной работе мы ожидаем, что вы самостоятельно:

- Настроите eBGP IPv6 unicast между офисом Москва и двумя провайдерами - Киторн и Ламас.
- Настроите eBGP IPv6 unicast между провайдерами Киторн и Ламас.
- Настроите eBGP IPv6 unicast между Ламас и Триада.
- Настроите eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада.
Организуете IPv6 unicast связность между пограничными роутерами офисов Москва и С.-Петербург.
Настроите iBGP IPv6 unicast в офисе Москва между маршрутизаторами R14 и R15.
- Настроите iBGP IPv6 unicast в провайдере Триада, с использованием RR.



<br>

### Дефолт команды для настройки сессий


#### IPv6
```
show ipv6 route
show ipv6 route bgp
show bgp ipv6 unicast
show bgp ipv6 unicast summary
show bgp ipv6 unicast neighbors 2001:301:15:21::2 advertised-routes
```

### Настройка IPv6 сессий

Для настройки передачи IPv6 префиксов используется MP-BGP с указанием `address-family`.

Возможность установить MP-BGP сессию двумя способами:
- IPv6 ребро через которое идут IPv6 префиксы
- IPv4 ребро через которые идут IPv6 префиксы


Для указания специфичных настроек для IPv4 и IPv6 (например, ребер и сетей) используется 
 `address-family'. 

Пример конфигурации с использованием `address-family` для передачи IPv6 префиксов через IPv6 соседа (типичная конфигурация):

```
!
router bgp 1001
 bgp router-id 101.0.0.14
 bgp log-neighbor-changes
 neighbor 101.0.0.22 remote-as 101
 neighbor 2001:101:14:22::2 remote-as 101
  !
 address-family ipv4
   network 100.0.0.0 mask 255.255.255.0
   neighbor 101.0.0.21 activate
   no neighbor 2001:101:14:22::2 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:1001:0:100::14/128
  neighbor 2001:101:14:22::2 activate
 exit-address-family
!
```

При помощи `no neighbor` и `activate` задаем ребро для передачи префиксов. 



### Тест для IPv6

Не забываем source для того, чтобы подставить адрес Loopback-а (изветной сети). 

```
R18#ping 2001:1001:0:101::15 source 2001:2024:0:200::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1001:0:101::15, timeout is 2 seconds:
Packet sent with a source address of 2001:2024:0:200::18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/18 ms
R18#

R18#ping 2001:1001:0:101::15 source 2001:2024:0:201::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1001:0:101::15, timeout is 2 seconds:
Packet sent with a source address of 2001:2024:0:201::18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

```
R15#ping 2001:2024:0:201::18 source 2001:1001:0:101::15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:2024:0:201::18, timeout is 2 seconds:
Packet sent with a source address of 2001:1001:0:101::15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/10 ms
R15#

R15#ping 2001:2024:0:200::18 source 2001:1001:0:101::15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:2024:0:200::18, timeout is 2 seconds:
Packet sent with a source address of 2001:1001:0:101::15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


<summary> Табличка с адресацией под рукой </summary>

|Device|Port|IPv4      |IPv6                                    |VLAN|Link              |Comment           |
|------|----|----------|----------------------------------------|----|------------------|------------------|
|R14   |e0/0|10.0.2.14 |2001:0:12:14::2                         |-   |R14 e0/0 - e0/2 R12|Connectivity      |
|R14   |e0/1|10.0.1.14 |2001:0:13:14::2                         |-   |R14 e0/1 - e0/3 R13|Robustness        |
|R14   |e0/2|101.0.0.14|2001:1001:14:22::1                      |-   |R14 e0/2 - e0/0 R22|BGP: Provider link|
|R15   |e0/0|10.0.2.15 |2001:0:13:15::2                         |-   |R15 e0/0 - e0/2 R13|Connectivity      |
|R15   |e0/1|10.0.1.15 |2001:0:12:15::2                         |-   |R15 e0/1 - e0/3 R12|Robustness        |
|R15   |e0/2|30.0.0.15 |2001:1001:15:21::1                      |-   |R15 e0/2 - e0/0 R21|BGP: Provider link|
|R18   |e0/0|10.0.1.18 |2001:0:16:18::2                         |-   |R18 e0/0 – e0/1 R16|Connectivity      |
|R18   |e0/1|10.0.0.18 |2001:0:17:18::2                         |-   |R18 e0/1 – e0/1 R17|Connectivity      |
|R18   |e0/2|52.0.1.18 |2001:520:18:24::1                       |-   |R18 e0/2 – e0/3 R24|BGP: Provider link|
|R18   |e0/3|52.0.2.18 |2001:520:18:26::1                       |-   |R18 e0/3 – e0/3 R26|BGP: Provider link|
|R21        |e0/0|30.0.0.21 |2001:301:15:21::2                       |-   |R21 e0/0 – e0/2 R15|BGP: Customer link|
|R21        |e0/1|192.168.0.21|2001:0:21:22::1                         |-   |R21 e0/1 – e0/1 R22|BGP: Peering link |
|R21        |e0/2|52.0.0.21 |2001:520:21:24::1                       |-   |R21 e0/2 – e0/0 R24|BGP: Provider link|
|R22        |e0/0|101.0.0.22|2001:101:14:22::2                       |-   |R22 e0/0 – e0/2 R14|BGP: Customer link|
|R22        |e0/1|192.168.0.22|2001:0:21:22::2                         |-   |R22 e0/1 – e0/1 R21|BGP: Peering link |
|R22        |e0/2|52.0.0.22 |2001:520:22:23::1                       |-   |R22 e0/2 – e0/0 R23|BGP: Provider link|
|R23        |e0/0|52.0.0.23 |2001:520:22:23::2                       |-   |R23 e0/0 – e0/2 R22|BGP: Customer link|
|R23        |e0/1|10.0.1.23 |2001:0:23:25::1                         |-   |R23 e0/1 – e0/0 R25|Connectivity      |
|R23        |e0/2|10.0.0.23 |2001:0:23:24::1                         |-   |R23 e0/2 – e0/2 R24|Connectivity      |
|R24        |e0/0|52.0.0.24 |2001:520:21:24::2                       |-   |R24 e0/0 – e0/2 R21|BGP: Customer link|
|R24        |e0/1|10.0.3.24 |2001:0:24:26::1                         |-   |R24 e0/1 – e0/0 R26|Connectivity      |
|R24        |e0/2|10.0.0.24 |2001:0:23:24::2                         |-   |R24 e0/2 – e0/2 R23|Connectivity      |
|R24        |e0/3|52.0.1.24 |2001:520:18:24::2                       |-   |R24 e0/3 – e0/2 R18|BGP: Customer link|
|R25        |e0/0|10.0.1.25 |2001:0:23:25::2                         |-   |R25 e0/0 – e0/1 R23|Connectivity      |
|R25        |e0/1|52.0.5.25 |2001:520:25:27::1                       |-   |R25 e0/1 – e0/0 R27|BGP: Customer link|
|R25        |e0/2|10.0.2.25 |2001:0:25:26::1                         |-   |R25 e0/2 – e0/2 R26|Connectivity      |
|R25        |e0/3|52.0.4.25 |2001:520:25:28::1                       |-   |R25 e0/3 – e0/1 R28|BGP: Customer link|
|R26        |e0/0|10.0.3.26 |2001:0:24:26::2                         |-   |R26 e0/0 – e0/1 R24|Connectivity      |
|R26        |e0/1|52.0.3.26 |2001:520:26:28::1                       |-   |R26 e0/1 – e0/0 R28|BGP: Customer link|
|R26        |e0/2|10.0.2.26 |2001:0:25:26::2                         |-   |R26 e0/2 – e0/2 R25|Connectivity      |
|R26        |e0/3|52.0.2.26 |2001:520:18:26::2                       |-   |R26 e0/3 – e0/3 R18|BGP: Customer link|
