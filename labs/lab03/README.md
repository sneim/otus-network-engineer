## OSPF

### Цель:
Настроить OSPF офисе Москва.  
Разделить сеть на зоны.  
Настроить фильтрацию между зонами.  

### Описание/Пошаговая инструкция выполнения домашнего задания:

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.
5. Настройка для IPv6 повторяет логику IPv4.

### Схема стенда
![img_16.png](img_16.png)

### Выполнение

В Московском офисе у нас 3-х уровневая модель, поэтому:
- area 0 – будет имплементацией для Core layer 
- area 10 – будет реализован Distribution layer

Типичная конфигурация выглядит следующим образом (пример для R15):
```
router ospf 1
 router-id 15.15.15.15
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 no passive-interface Ethernet1/0
!
```

```
interface Ethernet1/0
 no shutdown
 ip address 10.0.5.15 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:1001:14:15::2/64
!
```

В примере выше неиспользуемые интерфейсы переведены в passive-режим (hello-пакеты не отсылаются) для уменьшения служебного трафика между маршрутизаторами.

После базовой настройки подключение к area 10 выглядит след образом:

```
R15#configure terminal
R15(config)#int e0/1
R15(config-if)#ip ospf 1 area 10
R15(config-if)#end
```
После аналогичного конфигурирования нескольких маршрутизаторов (R12, R13, R14) можно видеть, что соседство установлено:

```
R15#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
14.14.14.14       1   FULL/BDR        00:00:36    10.0.5.14       Ethernet1/0
12.12.12.12       1   FULL/DR         00:00:39    10.0.1.12       Ethernet0/1
13.13.13.13       1   FULL/DR         00:00:35    10.0.2.13       Ethernet0/0
R15#
```

Заметим, что в таблице маршрутизации, маршруты внутри зоны маркируются меткой `O`:

```
R15#show ip route ospf
...
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks
O        10.6.66.14/32 [110/11] via 10.0.5.14, 00:10:09, Ethernet1/0
```
Между зонами меткой `IA`:
```
R12#show ip route ospf

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.5.0/24 [110/20] via 10.0.2.14, 00:33:32, Ethernet0/2
                     [110/20] via 10.0.1.15, 00:04:18, Ethernet0/3
O IA     10.6.66.14/32 [110/11] via 10.0.2.14, 00:33:32, Ethernet0/2
```


### Анонсирование маршрутов по умолчанию

_Способ №1_

На R15 в настройках ospf делаем:
```
default-information originate
```

Задаем сам default:
```
R15(config)#ip route 0.0.0.0 0.0.0.0 30.0.0.21 name default_to_lamas
```

Проверяем:
```
R13#show ip route ospf
...
Gateway of last resort is 10.0.2.15 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.2.15, 00:01:58, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.5.0/24 [110/20] via 10.0.2.15, 02:59:04, Ethernet0/2
                     [110/20] via 10.0.1.14, 02:59:14, Ethernet0/3
O IA     10.6.66.14/32 [110/11] via 10.0.1.14, 02:59:14, Ethernet0/3
```

_Способ №2_

Принудительное распространение дефолта, даже если он не установлен:

```
R14(config-router)#default-information originate always
```

Видим, что теперь есть два маршрута по умолчанию:
```
R13#show ip route ospf

O*E2  0.0.0.0/0 [110/1] via 10.0.2.15, 00:54:31, Ethernet0/2
                [110/1] via 10.0.1.14, 00:54:41, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
*** 
```

По умолчанию стоит E2 и внутренние косты не учитываются.
Если нужно учитывать внутренние косты, то нужно включить E1.


### Выделение total stub (блокировка маршрутов из других зон) 

Нужно R19 выделить в area 101 и получать только маршрут по умолчанию.

Сначала объединим устройства в `standard area`:
```
ip ospf 1 area 101
```
Видим, что на R19 распространились все маршруты, фильтрации нет:
```
R19#show ip route

Gateway of last resort is 10.0.3.14 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.3.14, 00:00:06, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.1.0/24 [110/20] via 10.0.3.14, 00:00:06, Ethernet0/0
O IA     10.0.2.0/24 [110/20] via 10.0.3.14, 00:00:06, Ethernet0/0
C        10.0.3.0/24 is directly connected, Ethernet0/0
L        10.0.3.19/32 is directly connected, Ethernet0/0
O IA     10.0.5.0/24 [110/20] via 10.0.3.14, 00:00:06, Ethernet0/0
O IA     10.6.66.14/32 [110/11] via 10.0.3.14, 00:00:06, Ethernet0/0
R19#
```

Тоесть получаются все маршруты (пакеты со всеми типами LSA).

Поменяем на интерфейсах соседей (R14, R19) тип area на **total stub**, чтобы внутрь проходил только маршрут по умолчанию:
```
area 101 stub no-summary
```

Теперь R19 получает от R14 только маршрут по умолчанию:

```
R19#show ip route
Gateway of last resort is 10.0.3.14 to network 0.0.0.0
O*IA  0.0.0.0/0 [110/11] via 10.0.3.14, 00:00:27, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.3.0/24 is directly connected, Ethernet0/0
L        10.0.3.19/32 is directly connected, Ethernet0/0
```

Такая вот "фильтрация".
 
### Фильтрация на ABR/ASBR

Для теста фильтрации добавим на Loopback R19 IP-адреса (их и будем фильтровать):

```
R19(config)#int Loopback 0
R19(config-if)#ip address 100.0.0.19 255.255.255.255
R19(config-if)#ipv6 address 2001:1001:0:100::19/64
```

Смотрим на R20 (до фильтрации и включения OSPF):

<details>

<summary> почти пусто </summary>

```
R20#show ip route
Gateway of last resort is not set
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.3.0/24 is directly connected, Ethernet0/0
L        10.0.3.20/32 is directly connected, Ethernet0/0
```

```
R20#show ipv6 route
IPv6 Routing Table - default - 3 entries
C   2001:0:15:20::/64 [0/0]
     via Ethernet0/0, directly connected
L   2001:0:15:20::2/128 [0/0]
     via Ethernet0/0, receive
L   FF00::/8 [0/0]
     via Null0, receive
```

</details>

Включаем OSPF, проверяем появление маршрутов:

```
Gateway of last resort is 10.0.3.15 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.3.15, 00:08:38, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.1.0/24 [110/20] via 10.0.3.15, 00:08:38, Ethernet0/0
O IA     10.0.2.0/24 [110/20] via 10.0.3.15, 00:08:38, Ethernet0/0
C        10.0.3.0/24 is directly connected, Ethernet0/0
L        10.0.3.20/32 is directly connected, Ethernet0/0
O IA     10.0.5.0/24 [110/20] via 10.0.3.15, 00:08:38, Ethernet0/0
O IA     10.6.66.14/32 [110/21] via 10.0.3.15, 00:08:38, Ethernet0/0
      100.0.0.0/32 is subnetted, 1 subnets
O IA     100.0.0.19 [110/31] via 10.0.3.15, 00:00:08, Ethernet0/0
```

Добавляем фильтры на R15:

```
R15(config)#ip prefix-list DENY-AREA-101 seq 5 deny 100.0.0.19/32
R15(config)#ip prefix-list DENY-AREA-101 seq 100 permit 0.0.0.0/0 le 32

R15(config)#router ospf 1
R15(config-router)#area 102 filter-list prefix DENY-AREA-101 in
R15(config-router)#end
```


Проверяем, что `100.0.0.19` исчез из таблицы:
```
R20#show ip route
Gateway of last resort is 10.0.3.15 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.0.3.15, 00:24:16, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.1.0/24 [110/20] via 10.0.3.15, 00:24:16, Ethernet0/0
O IA     10.0.2.0/24 [110/20] via 10.0.3.15, 00:24:16, Ethernet0/0
C        10.0.3.0/24 is directly connected, Ethernet0/0
L        10.0.3.20/32 is directly connected, Ethernet0/0
O IA     10.0.5.0/24 [110/20] via 10.0.3.15, 00:24:16, Ethernet0/0
O IA     10.6.66.14/32 [110/21] via 10.0.3.15, 00:24:16, Ethernet0/0
```

В area 102 (R20) поступают все маршруты, кроме маршрутов до area 101 (R19).

Тоесть фильтрация работает.
