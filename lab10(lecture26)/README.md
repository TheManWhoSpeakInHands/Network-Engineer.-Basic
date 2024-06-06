# Лабораторная работа. Настройка протокола OSPFv2 для одной области

# Задание
### [1. Создать сеть согласно топологии и произвести базовую настройку оборудования.](#1)
### [2. Настройка и проверка базовой работы протокола OSPFv2 для одной области.](#2)
### [3. Реализация различных оптимизаций на каждом маршрутизаторе.](#3)
### [4.	Проверка оптимизации OSPFv2.](#4)
## Таблица адресации

|Устройство | Интерфейс | IP-адрес          | Маска подсети   |
|-----------|-----------|-------------------|-----------------|
|R1         |G0/0/1     |10.53.0.1          | 255.255.255.0   |
|           |Loopback 1 |172.16.1.1         | 255.255.255.0   |
|R2         |G0/0/1     |10.53.0.2          | 255.255.255.0   |
|           |Loopback 1 |192.168.1.1        | 255.255.255.0   |

## Цели

Часть 1. Настройка основного сетевого устройства
Часть 2. Настройка сетей VLAN
Часть 3: Настройки безопасности коммутатора.

### Решение


### <a name="1"> 1. Создать сеть согласно топологии и произвести базовую настройку оборудования.</a>  

##### Топология Сети

<image src="./Conf.PNG" alt="Configuration.">  

##### Настройка и проверка основных параметров маршрутизаторров

```
enable
configure terminal
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
hostname R1
copy running-config startup-config 
```
Аналогично для второго маршрутизатора
##### Настройка и проверка основных параметров коммутаторов

```
enable
configure terminal
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
hostname S1
copy running-config startup-config 
```
Аналогично для второго коммутатора
### <a name="2"> 2. Настройка и проверка базовой работы протокола OSPFv2 для одной области.</a>  

##### Настройка адреса интерфейса и базового OSPFv2 на каждом маршрутизаторе

Настроим адреса интерфейсов на каждом маршрутизаторе, как показано в таблице адресации выше.

     R1(config)#int g0/0/1
     R1(config-if)#ip address 10.53.0.1 255.255.255.0
     R1(config-if)#no shutdown 

     R1(config)#interface loopback 1
     R1(config-if)#ip address 172.16.1.1 255.255.255.0
     R1(config-if)#no shutdown 

     R2(config)#in g0/0/1
     R2(config-if)#ip address 10.53.0.2 255.255.255.0
     R2(config-if)#no shutdown 
     %LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
     %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

     R2(config-if)#int loopback 1
     %LINK-5-CHANGED: Interface Loopback1, changed state to up
     %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up

     R2(config-if)#ip address 192.168.1.1 255.255.255.0
     R2(config-if)#no shutdown 

Перейдём в режим конфигурации маршрутизатора OSPF, используя идентификатор процесса 56.

     R1(config)#router ospf 56

Настроим статический идентификатор маршрутизатора для каждого маршрутизатора (1.1.1.1 для R1, 2.2.2.2 для R2).

     R1(config-router)#router-id 1.1.1.1
     R2(config-router)#router-id 2.2.2.2

Настроим инструкцию сети для сети между R1 и R2, поместив ее в область 0.

     R2(config-router)#network 10.53.0.0 0.0.0.255 area 0

Только на R2 добавим конфигурацию, необходимую для объявления сети Loopback 1 в область OSPF 0.

     R2(config-router)#network 192.168.1.1 0.0.0.0 area 0

Убедимся, что OSPFv2 работает между маршрутизаторами. Выполним  команду **show ip ospf neighbor**, чтобы убедиться, что R1 и R2 сформировали смежность.

     R1#sh ip ospf neighbor
          Neighbor ID     Pri   State           Dead Time   Address         Interface
          2.2.2.2           1   FULL/DR         00:00:35    10.53.0.2       GigabitEthernet0/0/1

     R2#show ip ospf neighbor 
          Neighbor ID     Pri   State           Dead Time   Address         Interface
          1.1.1.1           1   FULL/BDR        00:00:31    10.53.0.1       GigabitEthernet0/0/1

 <image src="./r2-neighbor.PNG" alt="Проверка смежности.">  

На R1 выполним команду **show ip route ospf**, и убедился, что сеть **R2** **Loopback1** присутствует в таблице маршрутизации. Обратил внимание, что поведение OSPF по умолчанию заключается в объявлении интерфейса обратной связи в качестве маршрута узла с использованием 32-битной маски.

     R1#show ip route ospf
          192.168.1.0/32 is subnetted, 1 subnets
     O       192.168.1.1 [110/2] via 10.53.0.2, 00:05:12, GigabitEthernet0/0/1

  <image src="./r1-route.PNG" alt="Таблица маршрутов R1."> 

Запустим Ping до адреса интерфейса R2 Loopback 1 из R1. Выполнение команды ping было успешным.

     R1#ping 192.168.1.1

     Type escape sequence to abort.
     Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
     !!!!!
     Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

  <image src="./r1-ping-loopback.PNG" alt="Пинг R1 - loopback R2.">  
    
### <a name="3"> 3. Реализация различных оптимизаций на каждом маршрутизаторе.</a>  

На R1 настроим приоритет OSPF интерфейса G0/0/1 на 50, чтобы убедиться, что R1 является назначенным маршрутизатором.

     R1(config)#int g0/0/1
     R1(config-if)#ip ospf priority 50
     R1(config-if)#end
     %SYS-5-CONFIG_I: Configured from console by console
     R1#clear ip ospf process
     Reset ALL OSPF processes? [no]: y

     R1#show ip ospf neighbor

          Neighbor ID     Pri   State           Dead Time   Address         Interface
          2.2.2.2           1   FULL/BDR        00:00:32    10.53.0.2       GigabitEthernet0/0/1

Настроим таймеры OSPF на G0/0/1 каждого маршрутизатора для таймера приветствия, составляющего 30 секунд.

     R1(config-if)#int g0/0/1
     R1(config-if)#ip ospf hello-interval 30
     R1(config-if)#ip ospf dead-interval 120

     R1#sh ip ospf neighbor 

          Neighbor ID     Pri   State           Dead Time   Address         Interface
          2.2.2.2           1   FULL/BDR        00:01:52    10.53.0.2       GigabitEthernet0/0/1

     R2#sh ip ospf neighbor 
          Neighbor ID     Pri   State           Dead Time   Address         Interface
          1.1.1.1          50   FULL/DR         00:01:57    10.53.0.1       GigabitEthernet0/0/1

На R1 настроим статический маршрут по умолчанию, который использует интерфейс Loopback 1 в качестве интерфейса выхода.

     R1(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
     %Default route without gateway, if not a point-to-point interface, may impact performance

Распространим маршрут по умолчанию в OSPF.

     R1(config)#router ospf 56
     R1(config-router)#default-information originate 
     R1(config-router)#end

     R1#sh ip route
     Gateway of last resort is 0.0.0.0 to network 0.0.0.0

          10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
     C       10.53.0.0/24 is directly connected, GigabitEthernet0/0/1
     L       10.53.0.1/32 is directly connected, GigabitEthernet0/0/1
          172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
     C       172.16.1.0/24 is directly connected, Loopback1
     L       172.16.1.1/32 is directly connected, Loopback1
          192.168.1.0/32 is subnetted, 1 subnets
     O       192.168.1.1/32 [110/2] via 10.53.0.2, 00:04:28, GigabitEthernet0/0/1
     S*   0.0.0.0/0 is directly connected, Loopback1

     R2#sh ip route
     Gateway of last resort is 10.53.0.1 to network 0.0.0.0

          10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
     C       10.53.0.0/24 is directly connected, GigabitEthernet0/0/1
     L       10.53.0.2/32 is directly connected, GigabitEthernet0/0/1
          192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
     C       192.168.1.0/24 is directly connected, Loopback1
     L       192.168.1.1/32 is directly connected, Loopback1
     O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:01:32, GigabitEthernet0/0/1

Добавим конфигурацию, необходимую для OSPF для обработки R2 Loopback 1 как сети точка-точка. Это привело к тому, что OSPF объявляет Loopback 1 использует маску подсети интерфейса.

     R2#sh ip ospf interface loopback 1

     Loopback1 is up, line protocol is up
     Internet address is 192.168.1.1/24, Area 0
     Process ID 56, Router ID 2.2.2.2, Network Type LOOPBACK, Cost: 1
     Loopback interface is treated as a stub Host

     R2(config)#int loopback 1
     R2(config-if)#ip ospf network point-to-point 

     R2#sh ip ospf interface loopback 1

     Loopback1 is up, line protocol is up
     Internet address is 192.168.1.1/24, Area 0
     Process ID 56, Router ID 2.2.2.2, Network Type POINT-TO-POINT, Cost: 1
     Transmit Delay is 1 sec, State POINT-TO-POINT,
     Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
     Index 2/2, flood queue length 0
     Next 0x0(0)/0x0(0)
     Last flood scan length is 1, maximum is 1
     Last flood scan time is 0 msec, maximum is 0 msec
     Suppress hello for 0 neighbor(s)

Только на R2 добавим конфигурацию, необходимую для предотвращения отправки объявлений OSPF в сеть Loopback 1.

     R2(config)#router ospf 56
     R2(config-router)#passive-interface loopback 1

Изменим базовую пропускную способность для маршрутизаторов. После этой настройки перезапустил OSPF с помощью команды **clear ip ospf process**.

     R1(config)#router ospf 56
     R1(config-router)#auto-cost reference-bandwidth 1000
     % OSPF: Reference bandwidth is changed.
          Please ensure reference bandwidth is consistent across all routers.
          
     R2(config-router)#auto-cost reference-bandwidth 1000
     % OSPF: Reference bandwidth is changed.
          Please ensure reference bandwidth is consistent across all routers.

### <a name="4"> 4. Проверка оптимизации OSPFv2.</a>  

Выполним команду **show ip ospf interface g0/0/1** на R1 и убедимся, что приоритет интерфейса установлен равным 50, а временные интервалы — Hello 30, Dead 120, а тип сети по умолчанию — Broadcast

     R1#sh ip ospf interface g0/0/1

          GigabitEthernet0/0/1 is up, line protocol is up
          Internet address is 10.53.0.1/24, Area 0
          Process ID 56, Router ID 1.1.1.1, **Network Type BROADCAST**, Cost: 10
          Transmit Delay is 1 sec, State DR, **Priority 50**
          Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
          Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
          Timer intervals configured, **Hello 30, Dead 120**, Wait 120, Retransmit 5
          Hello due in 00:00:24
          Index 1/1, flood queue length 0
          Next 0x0(0)/0x0(0)
          Last flood scan length is 1, maximum is 1
          Last flood scan time is 0 msec, maximum is 0 msec
          Neighbor Count is 1, Adjacent neighbor count is 1
          Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
          Suppress hello for 0 neighbor(s)

На R1 выполним команду **show ip route ospf**, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации. 

     R1#sh ip route ospf 
     O    192.168.1.0 [110/10] via 10.53.0.2, 00:04:30, GigabitEthernet0/0/1

Введём команду **show ip route ospf** на маршрутизаторе R2. Единственная информация о маршруте OSPF должна быть распространяемый по умолчанию маршрут R1.

     R2#sh ip route ospf
     O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:05:15, GigabitEthernet0/0/1

Запустим Ping до адреса интерфейса R1 Loopback 1 из R2. Выполнение команды ping успешно.

     R2#ping 172.16.1.1

     Type escape sequence to abort.
     Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
     !!!!!
     Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

 <image src="./r2-ping-loopback.PNG" alt="Пинг R2 - loopback R1.">  
