# Network Engineer. Basic. ДЗ №6 по лекции №14.
Таблица Адресации
| Устройство   | Интерфейс   | IP-адрес      | Маска подсети | Шлюз по умолчанию |
| :---------   | :---------- | :------------ | :------------ | :---------------- |
|  R1          | G0/0/1.10   | 192.168.10.1  | 255.255.255.0 | -                 |
|  R1          | G0/0/1/20   | 192.168.20.1  | 255.255.255.0 |  -                |
|  R1          | G0/0/1.30   | 192.168.30.1  | 255.255.255.0 |  -                |
|  R1          | G0/0/1.1000 | -             | -             | -                 |
|  S1          | VLAN 10     | 192.168.10.11 | 255.255.255.0 | 192.168.10.1      |
|  S2          | VLAN 10     | 192.168.10.12 | 255.255.255.0 | 192.168.10.1      |
| PC-A         | NIC         | 192.168.20.3  | 255.255.255.0 | 192.168.20.1      |
| PC-B         | NIC         | 192.168.30.3  | 255.255.255.0 | 192.168.30.1      |

Таблица VLAN

| VLAN | Имя | Назначенный интерфейс |
| :--- | :-- | :-------------------- |
| 10   | Управление | S1: VLAN 10 <br>S2 VLAN 10 |
| 20 | Sales | S1: F0/6 |
| 30 | Operations | S2: F0/18 |
| 999 | Parking_Lot | C1: F0/2-4, F0/7-24, G0/1-2 <br> С2: F0/2-17, F0/19-24, G0/1-2 |
| 1000 | Собственная | - |

# Задание
### [1. Создать сеть согласно топологии.](#1)
### [2. Настроить базовые параметры для маршрутизатора.](#2)
### [3. Настроить базовые параметры каждого коммутатора.](#3)
### [4. Настроить узлы ПК.](#4)
### [5. Создать сети VLAN на коммутаторах.](#5)
### [6. Назначить сети VLAN соответствующим интерфейсам коммутатора.](#6)
### [7. Вручную настроить магистральный интерфейс F0/1 на коммутаторах S1 и S2.](#7)
### [8. Вручную настроить магистральный интерфейс F0/5 на коммутаторе S1.](#8)
### [9. Настроить подинтерфейсы маршрутизатора.](#9)
### [10. Проверка работы маршрутизации между VLAN.](#10)
# Решение   
### <a name="1"> 1. Создать сеть согласно топологии.</a>  

<image src="./scheme.PNG" alt="Configuration."> 

>Сеть согласно топологии
  

### <a name="2"> 2. Настроить базовые параметры для маршрутизатора.</a>  
  ```
  enable  
  conf t  
  hostname R1  
  no ip domain-lookup  
  ena passw cisco  
  ena secr class  
  service password-enc  
  banner motd +"Hello! Unauthorized access is forbiden!"+  
  vty 0 15  
  password cisco  
  transport input ssh  
  end  
  clock set 20:00:00 Apr 29 2024  
  copy running-config startup-config  
  ```
### <a name="3"> 3. Настроить базовые параметры каждого коммутатора.</a>  
  *Аналогично п.2*    
  ```
  hostname S1  
  int vlan 10  
  ip address 192.168.10.11 255.255.255.0  
  no shutd  
  exit  
  ip default-gateway 192.168.10.1  
  
  hostname S2  
  int vlan 10  
  ip address 192.168.10.12 255.255.255.0  
  no shutd  
  exit  
  ip default-gateway 192.168.10.1 
  ```
### <a name="4"> 4. Настроить узлы ПК.</a> 
  <image src="./PC-A.PNG" alt="PC-A.">  
 
  <image src="./pc_b.PNG" alt="PC-B.">  

### <a name="5"> 5. Создать сети VLAN на коммутаторах.</a>  
  <image src="./S1_VLAN.PNG" alt="VLAN.">

  ```
  interface range fastEthernet 0/1 - 24, gigabitEthernet 0/1 - 2  
  shutdown  
  switchport mode access  
  switchport access vlan 999
  ```  
### <a name="6"> 6. Назначить сети VLAN соответствующим интерфейсам коммутатора.</a> 

  ```
  S1(config)#interface f0/6  
  S1(config-if)#switchport access vlan 20  
  S1(config-if)#no shutdown   
    
  S2(config)#interface f0/18  
  S2(config-if)#switchport access vlan 30  
  S2(config-if)#no shutdown  
  ```
  <image src="./S1_Check.PNG" alt="S1 Check.">  
    
  <image src="./S2_Check.PNG" alt="S2 Check.">  
    
### <a name="7"> 7. Вручную настроить магистральный интерфейс F0/1 на коммутаторах S1 и S2.</a>  
  ```
  interface fastEthernet 0/1  
  switchport mode trunk  
  switchport trunk native vlan 1000  
  switchport trunk allowed vlan 10,20,30,1000  
  no shutdown  
  ```
  <image src="./S2-Trunk_Check.PNG" alt="Trunk Check.">  
    
### <a name="8"> 8. Вручную настроить магистральный интерфейс F0/5 на коммутаторе S1.</a>  
  ```     
  interface fastEthernet 0/5  
  switchport mode trunk  
  switchport trunk native vlan 1000  
  switchport trunk allowed vlan 10,20,30,1000  
  no shutdown  
  ```
  <image src="./S1-Trunk_Check.PNG" alt="Trunk Check.">  
    
  Что произойдет, если G0/0/1 на R1 будет отключен?  
  * Мы не увидим этого замечательного сообщения на S1:  
  > %LINK-5-CHANGED: Interface FastEthernet0/5, changed state to up  

  > %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to up  
 
### <a name="9"> 9. Настроить подинтерфейсы маршрутизатора.</a>  
  ```  
  interface gigabitEthernet 0/0/1.10  
  description Def GW for vlan 10  
  encapsulation dot1Q 10  
  ip address 192.168.10.1 255.255.255.0  
  no shutdown  

  interface gigabitEthernet 0/0/1.20  
  description Def GW for vlan 20  
  encapsulation dot1Q 20  
  ip address 192.168.20.1 255.255.255.0  
  no shutdown  
    
  interface gigabitEthernet 0/0/1.30  
  description Def GW for vlan 30  
  encapsulation dot1Q 30  
  ip address 192.168.30.1 255.255.255.0  
  no shutdown  

  interface gigabitEthernet 0/0/1.1000  
  description native vlan  
  encapsulation dot1Q 1000 native  
  no shutdown  
  ```  
### <a name="10"> 10. Проверка работы маршрутизации между VLAN.</a>  
  * Эхо-запрос с PC-A на шлюз по умолчанию.  
    <image src="./ping-PC-A-DGW.PNG" alt="ping-PC-A-DGW.PNG">  
  * Эхо-запрос с PC-A на PC-B.  
    <image src="./ping-PC-A-PC-B.PNG" alt="ping-PC-A-PC-B.PNG">  
  * Эхо-запрос с PC-A на коммутатор S2.  
    <image src="./Ping-PC-A-S2.PNG" alt="Ping-PC-A-S2.PNG">  
  * Команда tracert с PC-B на адрес PC-A.  
    <image src="./tracert-PC-B-PC-A.PNG" alt="tracert-PC-B-PC-A.PNG">  
