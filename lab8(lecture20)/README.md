# Задание
### [1. Создать сеть согласно топологии.](#1)
### [2. Настроить базовые параметры для маршрутизаторов.](#2)
### [3. Проверка назначения адреса SLAAC от R1.](#3)
### [4.	Настройка и проверка сервера DHCPv6 на R1.](#4)
### [5. Настройка сервера DHCPv6 с сохранением состояния на R1.](#5)
### [6. Настройка R2 в качестве агента DHCP-ретрансляции.](#6)
### [7. Создание сети и настройка основных параметров устройств для реализации DHCPv4.](#7)
### [8. Проверка работы DHCPv4.](#8)

# Решение   
### <a name="1"> 1. Создать сеть согласно топологии.</a>  

<image src="./topology.PNG" alt="Configuration.">  

### Таблица Адресации

| Устройство   | Интерфейс   | IPv6-адрес                           | 
| :---------   | :---------- | :----------------------------------  | 
|  R1          | G0/0/0      | 2001:db8:acad:2::1/64 <br> fe80::1   |
|  R1          | G0/0/1      | 2001:db8:acad:1::1/64 <br> fe80::1   | 
|  R2          | G0/0/0      | 2001:db8:acad:2::2/64 <br> fe80::2   | 
|  R2          | G0/0/1      | 2001:db8:acad:3::1/64 <br>   fe80::1 | 
| PC-A         | NIC         | DHCP                                 | 
| PC-B         | NIC         | DHCP                                 | 


  
### <a name="2"> 2. Настроить базовые параметры для маршрутизаторов и коммутаторов.</a>  

#### Конфигурация коммутатора S1
```
S1#show running-config 
Building configuration...

Current configuration : 1449 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S1
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
enable password 7 0822455D0A16
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 shutdown
!
interface FastEthernet0/2
 shutdown
!
interface FastEthernet0/3
 shutdown
!
interface FastEthernet0/4
 shutdown
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
 shutdown
!
interface FastEthernet0/8
 shutdown
!
interface FastEthernet0/9
 shutdown
!
interface FastEthernet0/10
 shutdown
!
interface FastEthernet0/11
 shutdown
!
interface FastEthernet0/12
 shutdown
!
interface FastEthernet0/13
 shutdown
!
interface FastEthernet0/14
 shutdown
!
interface FastEthernet0/15
 shutdown
!
interface FastEthernet0/16
 shutdown
!
interface FastEthernet0/17
 shutdown
!
interface FastEthernet0/18
 shutdown
!
interface FastEthernet0/19
 shutdown
!
interface FastEthernet0/20
 shutdown
!
interface FastEthernet0/21
 shutdown
!
interface FastEthernet0/22
 shutdown
!
interface FastEthernet0/23
 shutdown
!
interface FastEthernet0/24
 shutdown
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
banner motd ^C"Hello! Unauthorized access is forbiden!"^C
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!
end
```
#### Конфигурация коммутатора S2
```
S2#show running-config 
Building configuration...

Current configuration : 1443 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname S2
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
enable password cisco
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 shutdown
!
interface FastEthernet0/2
 shutdown
!
interface FastEthernet0/3
 shutdown
!
interface FastEthernet0/4
 shutdown
!
interface FastEthernet0/5
!
interface FastEthernet0/6
 shutdown
!
interface FastEthernet0/7
 shutdown
!
interface FastEthernet0/8
 shutdown
!
interface FastEthernet0/9
 shutdown
!
interface FastEthernet0/10
 shutdown
!
interface FastEthernet0/11
 shutdown
!
interface FastEthernet0/12
 shutdown
!
interface FastEthernet0/13
 shutdown
!
interface FastEthernet0/14
 shutdown
!
interface FastEthernet0/15
 shutdown
!
interface FastEthernet0/16
 shutdown
!
interface FastEthernet0/17
 shutdown
!
interface FastEthernet0/18
!
interface FastEthernet0/19
 shutdown
!
interface FastEthernet0/20
 shutdown
!
interface FastEthernet0/21
 shutdown
!
interface FastEthernet0/22
 shutdown
!
interface FastEthernet0/23
 shutdown
!
interface FastEthernet0/24
 shutdown
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
banner motd ^C"Hello! Unauthorized access is forbiden!"^C
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!
end
```
#### Конфигурация маршрутизатора R1
```
R1#show running-config 
Building configuration...

Current configuration : 1002 bytes
!
version 15.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname R1
!
!
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
enable password 7 0822455D0A16
!
!
!
!
!
!
ip cef
ipv6 unicast-routing
!
no ipv6 cef
!
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0/0
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
!
interface GigabitEthernet0/0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
!
ip flow-export version 9
!
ipv6 route 2001:DB8:ACAD:2::/64 GigabitEthernet0/0/0
!
!
no cdp run
!
banner motd ^C"Hello! Unauthorized access is forbiden!"^C
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```
#### Конфигурация маршрутизатора R2
```
R2#show running-config 
Building configuration...

Current configuration : 1004 bytes
!
version 15.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname R2
!
!
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
enable password 7 0822455D0A16
!
!
!
!
!
!
ip cef
ipv6 unicast-routing
!
no ipv6 cef
!
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0/0
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:ACAD:2::2/64
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:ACAD:3::1/64
!
interface GigabitEthernet0/0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
!
ip flow-export version 9
!
ipv6 route 2001:DB8:ACAD:2::/64 GigabitEthernet0/0/0
!
!
no cdp run
!
banner motd ^C "Hello! Unauthorized access is forbiden!" ^C
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```  
  
### <a name="3"> 3. Проверка назначения адреса SLAAC от R1.</a> 

<image src="./slaac-pcA.PNG" alt="SLAAC PC-A.">  

  * Часть адреса с идентификатором хоста была сгенерирована алгоритмом EUI-64 на основе MAC адреса.  
  
### <a name="4"> 4. Настройка и проверка сервера DHCPv6 на R1.</a>  
Создаем новый пул:  

  * R1(config)#ipv6 dhcp pool R1-STATELESS  
  * R1(config)#dns-server 2001:db8:acad::254  
  * R1(config)#domain-name STATELESS.com  
  
Настраивем интерфейс G0/0/1 на R1:  

  * R1(config-if)#ipv6 nd other-config-flag  
  * R1(config-if)#ipv6 dhcp server R1-STATELESS  
  
<image src="./r1-stateless.PNG" alt="IPCONFIG.">  
   
Для того, чтобы обеспечить Ip связанность пропишем маршруты на роутерах:  
    
  * R1(config)#ipv6 route 2001:db8:acad:3::/64 2001:db8:acad:2::2  
  * R2(config)#ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::1  
  
<image src="./Ping-Stateless.PNG" alt="IPCONFIG.">  

### <a name="5"> 5. Настройка сервера DHCPv6 с сохранением состояния на R1.</a>  
Создаем новый пул с префиксом:  
  
  * R1(config)#ipv6 dhcp pool R2-STATEFUL  
  * R1(config)#address prefix 2001:db8:acad:3:aaa::/80  
  * R1(config)#dns-server 2001:db8:acad::254  
  * R1(config)#domain-name STATEFUL.com  
  
Настраивем интерфейс G0/0/0 на R1:  
  
  * R1(config-if)#ipv6 dhcp server R2-STATEFUL  
  
Проверка назначения адреса SLAAC на PC-B.  
  
<image src="./slaac-pc-B.PNG" alt="SLAAC на PC-B">   
  
### <a name="6"> 6. Настройка R2 в качестве агента DHCP-ретрансляции</a>  

  * R2(config-if)#ipv6 nd managed-config-flag  
  * R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0  

### <a name="7"> 7. Создание сети и настройка основных параметров устройств для реализации DHCPv4</a>  
  *Подсеть А: 192.168.1.0 255.255.255.192 - 1 адрес 192.168.1.1*  
  *Подсеть Б: 192.168.1.64 255.255.255.224 - 1 адрес 192.168.1.65*  
  *Подсеть В: 192.168.1.96 255.255.255.240 - 1 адрес 192.168.1.97*   
  S1  Config:  

<image src="./s1-vlan.PNG" alt="R1">   

  R1 Config:  

<image src="./n-r1v.PNG" alt="R1">  

  R2 Config:

<image src="./n-r2.PNG" alt="R1">  

  R1 DHCP настройки:  

<image src="./n-dhcp.PNG" alt="R1">  

### <a name="8"> 8. Проверка работы DHCPv4</a>  

  PC-A:  

<image src="./pc-a.PNG" alt="R1">  

  PC-B:  

<image src="./n-pc2.PNG" alt="R1">  
