# Лабораторная работа - Настройка NAT для IPv4

# Задание
### [1. Создать сеть согласно топологии и произвести базовую настройку оборудования.](#1)
### [2. Настройка сетей VLAN на коммутаторах.](#2)
### [3. Настройка и проверка PAT для IPv4.](#3)
### [4. Настройка и проверка статического NAT для IPv4.](#4)


## Таблица адресации

|Устройство | Интерфейс | IP-адрес          | Маска подсети   |
|-----------|-----------|-------------------|-----------------|
|R1         |G0/0/0     |209.165.200.230    | 255.255.255.248 |
|           |G0/0/1     |192.168.1.1        | 255.255.255.0   |
|R2         |G0/0/0     |209.165.200.225    | 255.255.255.248 |
|           |Lo1        |209.165.200.1      | 255.255.255.224 |
|S1         |VLAN 1     |192.168.1.11       | 255.255.255.0   |
|S2         |VLAN 1     |192.168.1.12       | 255.255.255.0   |
|PC-A       |NIC        |192.168.1.2        | 255.255.255.0   |
|PC-B       |NIC        |192.168.1.3        | 255.255.255.0   |

## Цели

Часть 1. Создание сети и настройка основных параметров устройства <br>
Часть 2. Настройка и проверка NAT для IPv4 <br>
Часть 3. Настройка и проверка PAT для IPv4 <br>
Часть 4. Настройка и проверка статического NAT для IPv4.

### Решение

### <a name="1"> 1. Создать сеть согласно топологии и произвести базовую настройку оборудования.</a>  

##### Создание сети

## Топология

![топология](image.png)

Сеть согласно топологии

##### Настройка и проверка основных параметров маршрутизаторов

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
int g0/0
# командой ip address настроили IP-адресации интерфейсов в соответствии с таблицей адресации
# Командой ip route настроили маршрут по умолчанию от R2 на R1
copy running-config startup-config 
```

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

### <a name="2"> 2. Настроим NAT на R1, используя пул из трех адресов 209.165.200.226-209.165.200.228.</a>  

##### Настройка NAT на R1, используя пул из трех адресов 209.165.200.226-209.165.200.228

Настроим простой список доступа, который определяет, какие хосты будут разрешены для трансляции. В этом случае все устройства в локальной сети R1 имеют право на трансляцию.

     R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255 

Создадим пул NAT и укажем ему имя и диапазон используемых адресов.

     R1(config)# ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248 

Настроим перевод, связывая ACL и пул с процессом преобразования.

     R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS 

Зададим внутренний (inside) интерфейс.

     R1(config)# interface g0/0/1
     R1(config-if)# ip nat inside

Определим внешний (outside) интерфейс.

     R1(config)# interface g0/0/0
     R1(config-if)# ip nat outside

<image src="./trans1.PNG" alt="Dynamic1.">  

##### Проверка конфигурации

С **PC-B**, запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** проверим таблицу NAT с помощью команды **show ip nat translations**.

     R1#sh ip nat tr
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.227:389 192.168.1.3:389    209.165.200.1:389  209.165.200.1:389
     icmp 209.165.200.227:390 192.168.1.3:390    209.165.200.1:390  209.165.200.1:390
     icmp 209.165.200.227:391 192.168.1.3:391    209.165.200.1:391  209.165.200.1:391
     icmp 209.165.200.227:392 192.168.1.3:392    209.165.200.1:392  209.165.200.1:392

<image src="./trans2.PNG" alt="Dynamic2."> 

**Вопросы:**
**Во что был транслирован внутренний локальный адрес PC-B?** 

>Внутренний локальный адрес (Inside local) был транслирован во **внутренний глобальный адрес** (**Inside global**). **192.168.1.3** был транслирован в **209.165.200.227**

**Какой тип адреса NAT является переведенным адресом?** 

>Внутренний глобальный адрес

С **PC-A**, запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** проверим таблицу NAT с помощью команды **show ip nat translations**.

     R1#sh ip nat tr
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.226:45  192.168.1.2:45     209.165.200.1:45   209.165.200.1:45
     icmp 209.165.200.226:46  192.168.1.2:46     209.165.200.1:46   209.165.200.1:46
     icmp 209.165.200.226:47  192.168.1.2:47     209.165.200.1:47   209.165.200.1:47
     icmp 209.165.200.226:48  192.168.1.2:48     209.165.200.1:48   209.165.200.1:48
     icmp 209.165.200.228:393 192.168.1.3:393    209.165.200.1:393  209.165.200.1:393
     icmp 209.165.200.228:394 192.168.1.3:394    209.165.200.1:394  209.165.200.1:394
     icmp 209.165.200.228:395 192.168.1.3:395    209.165.200.1:395  209.165.200.1:395
     icmp 209.165.200.228:396 192.168.1.3:396    209.165.200.1:396  209.165.200.1:396

Обратим внимание, что предыдущая трансляция для **PC-B** все еще находится в таблице. Из **S1** запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** проверим таблицу NAT.

     R1# show ip nat translations
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.226:45  192.168.1.2:45     209.165.200.1:45   209.165.200.1:45
     icmp 209.165.200.226:46  192.168.1.2:46     209.165.200.1:46   209.165.200.1:46
     icmp 209.165.200.226:47  192.168.1.2:47     209.165.200.1:47   209.165.200.1:47
     icmp 209.165.200.226:48  192.168.1.2:48     209.165.200.1:48   209.165.200.1:48
     icmp 209.165.200.227:2   192.168.1.11:2     209.165.200.1:2    209.165.200.1:2
     icmp 209.165.200.227:3   192.168.1.11:3     209.165.200.1:3    209.165.200.1:3
     icmp 209.165.200.227:4   192.168.1.11:4     209.165.200.1:4    209.165.200.1:4
     icmp 209.165.200.227:5   192.168.1.11:5     209.165.200.1:5    209.165.200.1:5
     icmp 209.165.200.228:393 192.168.1.3:393    209.165.200.1:393  209.165.200.1:393
     icmp 209.165.200.228:394 192.168.1.3:394    209.165.200.1:394  209.165.200.1:394
     icmp 209.165.200.228:395 192.168.1.3:395    209.165.200.1:395  209.165.200.1:395
     icmp 209.165.200.228:396 192.168.1.3:396    209.165.200.1:396  209.165.200.1:396
<image src="./trans3.PNG" alt="Dynamic3.">  

После этого попробуем запустить пинг интерфейса **Lo1** **R2** с коммутатора **S2**. Пинг не прошёл и, к сожалению на R1 не вывелось никакой информации об ошибке.

В данном случае это ожидаемый результат, потому что выделено только **3** адреса, и мы попытались использовать ping Lo1 с четырех устройств. А **NAT** — это трансляция «один-в-один».
Узнать как много выделено трансляций можно введя команду **show ip nat translations verbose**, и в выводе, вероятно, можно увидеть, что ответ будет 3. Но в эмуляторе не работает данная команда.

     R1# show ip nat translations verbose 

Проверим пинг интерфейса **Lo1** **R2** с коммутатора **S2** очистив трансляции.

     R1#sh ip nat tr
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.226:11  192.168.1.12:11    209.165.200.1:11   209.165.200.1:11
     icmp 209.165.200.226:12  192.168.1.12:12    209.165.200.1:12   209.165.200.1:12
     icmp 209.165.200.226:13  192.168.1.12:13    209.165.200.1:13   209.165.200.1:13
     icmp 209.165.200.226:14  192.168.1.12:14    209.165.200.1:14   209.165.200.1:14
     icmp 209.165.200.226:15  192.168.1.12:15    209.165.200.1:15   209.165.200.1:15

Учитывая, что пул ограничен тремя адресами, NAT для пула адресов недостаточно для нашего приложения. Очистим преобразование NAT и статистику, и перейдём к **PAT**.

     R1# clear ip nat translation *

### <a name="3"> 3. Настройка и проверка PAT для IPv4.</a>  

##### Удалите команду преобразования на R1

Чтобы начать работу в части 3, удалим команду, связывающую **ACL** и **POOL** вместе.

     R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS 

##### Добавим команду PAT на R1

Теперь настроим преобразование ***PAT*** в пул адресов. Так как ACL и Pool уже настроены, то единственная команда, которую нам нужно изменить с NAT на PAT:

     R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS overload

<image src="./pat1.PNG" alt="Pat.">  

##### Протестируем и проверим конфигурацию

Проверим, что **PAT** работает. С **PC-B**,  запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** отобразим таблицу **NAT** с помощью команды **show ip nat translations**.

     R1#sh ip nat translations 
     Pro  Inside global     Inside local       Outside local      Outside global
     icmp 209.165.200.227:5 192.168.1.3:5      209.165.200.1:5    209.165.200.1:5
     icmp 209.165.200.227:6 192.168.1.3:6      209.165.200.1:6    209.165.200.1:6
     icmp 209.165.200.227:7 192.168.1.3:7      209.165.200.1:7    209.165.200.1:7
     icmp 209.165.200.227:8 192.168.1.3:8      209.165.200.1:8    209.165.200.1:8

<image src="./pat2.PNG" alt="Pat."> 

**Во что был транслирован внутренний локальный адрес PC-B?** 

>Внутренний локальный адрес (Inside local) был транслирован во **внутренний глобальный адрес** (**Inside global**). **192.168.1.3** был транслирован в **209.165.200.226**

**Какой тип адреса NAT является переведенным адресом?**

>Внутренний глобальный адрес

**Чем отличаются выходные данные команды show ip nat translations из упражнения NAT?**

>Ничем.

С **PC-A**, запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** отобразим таблицу **NAT** с помощью команды **show ip nat translations**.

     R1#sh ip nat tr
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.227:10  192.168.1.2:10     209.165.200.1:10   209.165.200.1:10
     icmp 209.165.200.227:11  192.168.1.2:11     209.165.200.1:11   209.165.200.1:11
     icmp 209.165.200.227:12  192.168.1.2:12     209.165.200.1:12   209.165.200.1:12
     icmp 209.165.200.227:5   192.168.1.3:5      209.165.200.1:5    209.165.200.1:5
     icmp 209.165.200.227:6   192.168.1.3:6      209.165.200.1:6    209.165.200.1:6
     icmp 209.165.200.227:7   192.168.1.3:7      209.165.200.1:7    209.165.200.1:7
     icmp 209.165.200.227:8   192.168.1.3:8      209.165.200.1:8    209.165.200.1:8
     icmp 209.165.200.227:9   192.168.1.2:9      209.165.200.1:9    209.165.200.1:9

Видим, что трансляций несколько.
Отправим ping еще раз, и быстро вернёмся к маршрутизатору, введём команду **show ip nat translations verbose** 

<image src="./verb.PNG" alt="verb.">  

Cгенерируем трафик с нескольких устройств для наблюдения **PAT**. На PC-A и PC-B используем параметр -t с командой ping, чтобы отправить безостановочный ping на интерфейс Lo1 R2 (ping -t 209.165.200.1).
Затем на R1 выполним команду show ip nat translations:

     R1# show ip nat translations
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.227:1024192.168.1.3:13     209.165.200.1:13   209.165.200.1:1024
     icmp 209.165.200.227:1025192.168.1.3:14     209.165.200.1:14   209.165.200.1:1025
     icmp 209.165.200.227:1026192.168.1.3:15     209.165.200.1:15   209.165.200.1:1026
     icmp 209.165.200.227:1027192.168.1.3:16     209.165.200.1:16   209.165.200.1:1027
     icmp 209.165.200.227:10  192.168.1.3:10     209.165.200.1:10   209.165.200.1:10
     icmp 209.165.200.227:11  192.168.1.3:11     209.165.200.1:11   209.165.200.1:11
     icmp 209.165.200.227:12  192.168.1.3:12     209.165.200.1:12   209.165.200.1:12
     icmp 209.165.200.227:13  192.168.1.2:13     209.165.200.1:13   209.165.200.1:13
     icmp 209.165.200.227:14  192.168.1.2:14     209.165.200.1:14   209.165.200.1:14
     icmp 209.165.200.227:15  192.168.1.2:15     209.165.200.1:15   209.165.200.1:15
     icmp 209.165.200.227:16  192.168.1.2:16     209.165.200.1:16   209.165.200.1:16
     icmp 209.165.200.227:17  192.168.1.2:17     209.165.200.1:17   209.165.200.1:17
     icmp 209.165.200.227:18  192.168.1.2:18     209.165.200.1:18   209.165.200.1:18
     icmp 209.165.200.227:19  192.168.1.2:19     209.165.200.1:19   209.165.200.1:19
     icmp 209.165.200.227:20  192.168.1.2:30     209.165.200.1:30   209.165.200.1:30
     icmp 209.165.200.227:9   192.168.1.3:9      209.165.200.1:9    209.165.200.1:9

Отметим, что внутренний глобальный адрес одинаков для обоих сеансов.

Вопрос:
**Как маршрутизатор отслеживает, куда идут ответы?** 
>По номеру порта. Получая пакет от клиента, он использует свой номер порта источника, чтобы уникальным образом определить конкретное преобразование NAT.

Перед переходом к PAT с перегрузкой интерфейса очистим трансляции и статистику:

     R1# clear ip nat translation * 

##### На R1 удалите команды преобразования nat pool

Так как созданный список доступа (список доступа 1) по-прежнему корректен для сетевого сценария, то нет необходимости воссоздавать его. Кроме того, внутренний и внешний интерфейсы не меняются. Чтобы начать работу с PAT к интерфейсу, очистим конфигурацию, удалив пул NAT и команду, связывающую ACL и пул вместе.

     R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS overload 
     R1(config)# no ip nat pool PUBLIC_ACCESS

##### Добавьте команду PAT overload, указав внешний интерфейс

Добавим команду **PAT**, которая вызовет перегрузку внешнего интерфейса.

     R1(config)# ip nat inside source list 1 interface g0/0/0 overload 

##### Протестируйте и проверьте конфигурацию

Проверим, что **PAT** работает. С **PC-B**,  запустим эхо-запрос интерфейса **Lo1** (209.165.200.1) на **R2**. На **R1** отобразим таблицу **NAT** с помощью команды **show ip nat translations**.

     R1# show ip nat translations
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.230:30  192.168.1.3:30     209.165.200.1:30   209.165.200.1:30
     icmp 209.165.200.230:31  192.168.1.3:31     209.165.200.1:31   209.165.200.1:31
     icmp 209.165.200.230:32  192.168.1.3:32     209.165.200.1:32   209.165.200.1:32
     icmp 209.165.200.230:33  192.168.1.3:33     209.165.200.1:33   209.165.200.1:33

<image src="./pat3.PNG" alt="Pat.">  

Запустим трафик с нескольких устройств для наблюдения PAT. На PC-A и PC-B используем параметр -t с командой ping для отправки безостановочного ping на интерфейс Lo1 R2 (ping -t 209.165.200.1). На S1 и S2 выполним команду **ping 209.165.200.1**. Затем на R1 и выполнил команду **show ip nat translations**.

     R1#sh ip nat translations 
     Pro  Inside global       Inside local       Outside local      Outside global
     icmp 209.165.200.230:1024192.168.1.11:2     209.165.200.1:2    209.165.200.1:1024
     icmp 209.165.200.230:1025192.168.1.11:3     209.165.200.1:3    209.165.200.1:1025
     icmp 209.165.200.230:1026192.168.1.11:4     209.165.200.1:4    209.165.200.1:1026
     icmp 209.165.200.230:1027192.168.1.11:5     209.165.200.1:5    209.165.200.1:1027
     icmp 209.165.200.230:2   192.168.1.12:2     209.165.200.1:2    209.165.200.1:2
     icmp 209.165.200.230:34  192.168.1.3:34     209.165.200.1:34   209.165.200.1:34
     icmp 209.165.200.230:35  192.168.1.3:35     209.165.200.1:35   209.165.200.1:35
     icmp 209.165.200.230:36  192.168.1.3:36     209.165.200.1:36   209.165.200.1:36
     icmp 209.165.200.230:37  192.168.1.3:37     209.165.200.1:37   209.165.200.1:37
     icmp 209.165.200.230:3   192.168.1.12:3     209.165.200.1:3    209.165.200.1:3
     icmp 209.165.200.230:46  192.168.1.2:46     209.165.200.1:46   209.165.200.1:46
     icmp 209.165.200.230:47  192.168.1.2:47     209.165.200.1:47   209.165.200.1:47
     icmp 209.165.200.230:48  192.168.1.2:48     209.165.200.1:48   209.165.200.1:48
     icmp 209.165.200.230:49  192.168.1.2:49     209.165.200.1:49   209.165.200.1:49
     icmp 209.165.200.230:4   192.168.1.12:4     209.165.200.1:4    209.165.200.1:4
     icmp 209.165.200.230:50  192.168.1.2:50     209.165.200.1:50   209.165.200.1:50
     icmp 209.165.200.230:5   192.168.1.12:5     209.165.200.1:5    209.165.200.1:5

Теперь все внутренние глобальные адреса сопоставляются с IP-адресом интерфейса g0/0/0.

<image src="./pat2_1.PNG" alt="Pat.">

### <a name="4"> 4. Настройка и проверка статического NAT для IPv4.</a>  

Настроим статический NAT таким образом, чтобы PC-A был доступен напрямую из Интернета. PC-A будет доступен из R2 по адресу 209.165.200.229.

##### На R1 очистим текущие трансляции и статистику

     R1# clear ip nat translation * 

##### На R1 настроим NAT, для статического сопоставления внутреннего адреса с внешним адресом

Для этого настроим статическое сопоставление между 192.168.1.11 и 209.165.200.1 с помощью следующей команды:

     R1(config)#ip nat inside source static 192.168.1.2 209.165.200.229

##### Протестируйте и проверьте конфигурацию

Проверяем, что статический NAT работает. На R1 отобразим таблицу NAT с помощью команды **show ip nat translations**.
Видим статическое сопоставление.

     R1# show ip nat translations
     Pro  Inside global     Inside local       Outside local      Outside global
     ---  209.165.200.229   192.168.1.2        ---                ---

Таблица перевода показывает, что статическое преобразование действует. Проверим это, запустив ping с R2 на 209.165.200.229. Пинги должны работать.

     R2#ping 209.165.200.229
     Type escape sequence to abort.
     Sending 5, 100-byte ICMP Echos to 209.165.200.229, timeout is 2 seconds:
     .!!!!
     Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms

На R1 отобразите таблицу NAT на R1 с помощью команды show ip nat translations, и вы увидите статическое сопоставление и преобразование на уровне порта для входящих pings.

     R1# show ip nat translations
     Pro  Inside global     Inside local       Outside local      Outside global
     icmp 209.165.200.229:2 192.168.1.2:2      209.165.200.225:2  209.165.200.225:2
     icmp 209.165.200.229:3 192.168.1.2:3      209.165.200.225:3  209.165.200.225:3
     icmp 209.165.200.229:4 192.168.1.2:4      209.165.200.225:4  209.165.200.225:4
     icmp 209.165.200.229:5 192.168.1.2:5      209.165.200.225:5  209.165.200.225:5
     ---  209.165.200.229   192.168.1.2        ---                ---

Это подтверждает, что статический NAT работает.
