## BGP. Основы

### Цель
Настроить BGP между автономными системами<br>
Организовать доступность между офисами Москва и С.-Петербург<br>

В этой  самостоятельной работе мы ожидаем, что вы самостоятельно:

- Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
- Настроите eBGP между провайдерами Киторн и Ламас.
- Настроите eBGP между Ламас и Триада.
- Настроите eBGP между офисом С.-Петербург и провайдером Триада.
- Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.


### Схема стенда
![img_16.png](img_16.png)


### Минимальная настройка

Анонс 100.0.0.0 с одной стороны Москвы:
```
router bgp 1001
 bgp router-id 101.0.0.14
 network 100.0.0.0 mask 255.255.255.0
 neighbor 101.0.0.22 remote-as 101
```

Прием анонсов с другой стороны Киторн:
```
router bgp 101
 bgp router-id 101.0.0.22
 neighbor 101.0.0.14 remote-as 1001
```

### Полезные команды с занятий

#### IPv4
```
show ip route
show ip route bgp
show ip bgp
show ip bgp summary
show ip bgp neighbors 30.0.0.15 advertised-routes
```


### Проверка BGP-связности между Москвой и Питером

Маршруты со стороны Питера:
```
R18#show ip bgp
BGP table version is 5, local router ID is 52.0.1.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  100.0.0.0/24     52.0.1.24                              0 520 301 101 1001 i
 *>  100.0.1.0/24     52.0.1.24                              0 520 301 1001 i
 *>  200.0.0.0        0.0.0.0                  0         32768 i
 *>  200.0.1.0        0.0.0.0                  0         32768 i
```

Со стороны Москвы:
```
R14#show ip bgp
BGP table version is 8, local router ID is 101.0.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  100.0.0.0/24     0.0.0.0                  0         32768 i
 *>  200.0.0.0        101.0.0.22                             0 101 301 520 2042 i
 *>  200.0.1.0        101.0.0.22                             0 101 301 520 2042 i
```


### Проверка доступности (ping) между Москвой и Питером

Проверим доступность между бордерами.  

```
R18#ping 100.0.1.15 source 200.0.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.0.1.15, timeout is 2 seconds:
Packet sent with a source address of 200.0.0.18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
```
R18#traceroute 100.0.1.15 source 200.0.0.18
Type escape sequence to abort.
Tracing the route to 100.0.1.15
VRF info: (vrf in name/id, vrf out name/id)
  1 52.0.1.24 0 msec 1 msec 0 msec
  2 52.0.0.21 0 msec 0 msec 0 msec
  3 30.0.0.15 1 msec 1 msec *
```



Для Москвы – аналогично

```
R14#ping 200.0.0.18 source 100.0.0.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.0.0.18, timeout is 2 seconds:
Packet sent with a source address of 100.0.0.14
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#
```

```
R15#ping 200.0.1.18 source 100.0.1.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.0.1.18, timeout is 2 seconds:
Packet sent with a source address of 100.0.1.15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

</details>
