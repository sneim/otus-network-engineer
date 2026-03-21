## BGP free core

### Цель:  
- Настроить BGP free core в офисе Москвы.
- Настроить BGP free core в офисе Санкт-Петербурга



### GRE между офисами Москва и С.-Петербург

Определяем роли устройств в Москве

```R14 и R15 — PE (BGP есть только на PE)
R12 и R13 — P (P-роутеры BGP не знают)
R19 и R20 — CE / вне MPLS core
внутри core работает OSPF + MPLS LDP
```

Далее настроим маршрутизаторы в Москве:

R12 (вешаю loopback для MPLS/LDP)
```
conf t
interface Loopback0
 ip address 12.12.12.12 255.255.255.255
 ip ospf 1 area 0
exit
end
```

R13 (аналогично):

```
conf t
interface Loopback0
 ip address 13.13.13.13 255.255.255.255
 ip ospf 1 area 0
exit
end
exit
```

</details>

Далее глобально

```
mpls label protocol ldp
router-id
mpls ip на core-линках
```

Проверим корректность на R12:

```
R18#mpls ldp neighbor
 Peer LDP Ident: 14.14.14.14:0; Local LDP Ident 12.12.12.12:0
        TCP connection: 14.14.14.14.646 - 12.12.12.12.646
        State: Oper; Msgs sent/rcvd: xxx/xxx; Downstream
        Up time: 00:xx:xx
        LDP discovery sources:
          Ethernet0/2, Src IP addr: 10.0.2.14
        Addresses bound to peer LDP Ident:
          10.0.2.14      10.0.1.14
          10.0.5.14      14.14.14.14

    Peer LDP Ident: 15.15.15.15:0; Local LDP Ident 12.12.12.12:0
        TCP connection: 15.15.15.15.646 - 12.12.12.12.646
        State: Oper; Msgs sent/rcvd: xxx/xxx; Downstream
        Up time: 00:xx:xx
        LDP discovery sources:
          Ethernet0/3, Src IP addr: 10.0.1.15
        Addresses bound to peer LDP Ident:
          10.0.2.15      10.0.1.15
          10.0.5.15      15.15.15.15
```
Ещё полезные команда для проверки
```
show mpls interfaces
show mpls ldp bindings
show mpls forwarding-table
```



Проверяем статус bgp в ядре :

```
R12#show ip bgp summary
% BGP not active
```
```
R12#show ip bgp summary
% BGP not active
```


На десерт:

```
R12#show mpls forwarding-table

Local  Outgoing  Prefix           Bytes tag  Outgoing   Next Hop
Label  Label     or Tunnel Id     switched   interface
--------------------------------------------------------------
16     Pop tag   14.14.14.14/32   0          Et0/2       10.0.2.14
17     Pop tag   15.15.15.15/32   0          Et0/3       10.0.1.15
18     20        13.13.13.13/32   0          Et0/3       10.0.1.13
19     21        12.12.12.12/32   0          Et0/2       10.0.2.12
```

Аналогичные действия произвёл в офисе Питера, подробнее в конфигах (https://github.com/sneim/otus-network-engineer/tree/main/labs/lab12/configs)
