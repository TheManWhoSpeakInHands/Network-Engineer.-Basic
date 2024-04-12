# Network Engineer. Basic. ДЗ №5 по лекции №12.

## Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH


| Устройство  | Интерфейс   | IP-адрес|Маска подсети|Шлюз по умолчанию|
| :------------ |:---------------:| :---------------:| :---------------:| :-----:|
| R1      | G0/0/1 | 192.168.1.1 |255.255.255.0 |-|
| S1      | VLAN 1 | 192.168.1.11 |255.255.255.0 |192.168.1.1 |
| PC-A      | NIC       |  192.168.1.3 |255.255.255.0 |192.168.1.1 |

## Задание

##### [Часть 1. Настройка основных параметров устройства](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/main/lab5(lecture12)/README.md#%D1%87%D0%B0%D1%81%D1%82%D1%8C-1-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%BD%D1%8B%D1%85-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%BE%D0%B2-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%B0-1)

##### [Часть 2. Настройка маршрутизатора для доступа по протоколу SSH](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/main/lab5(lecture12)/README.md#%D1%87%D0%B0%D1%81%D1%82%D1%8C-2-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%82%D0%BE%D1%80%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%B0-%D0%BF%D0%BE-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D1%83-ssh-1)

##### [Часть 3. Настройка коммутатора для доступа по протоколу SSH](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/main/lab5(lecture12)/README.md#%D1%87%D0%B0%D1%81%D1%82%D1%8C-3-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%BA%D0%BE%D0%BC%D0%BC%D1%83%D1%82%D0%B0%D1%82%D0%BE%D1%80%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%B0-%D0%BF%D0%BE-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D1%83-ssh-1)

##### [Часть 4. SSH через интерфейс командной строки (CLI) коммутатора](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/main/lab5(lecture12)/README.md#%D1%87%D0%B0%D1%81%D1%82%D1%8C-4-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D0%B0-ssh-%D1%81-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%D0%BC-%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D1%84%D0%B5%D0%B9%D1%81%D0%B0-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%BD%D0%BE%D0%B9-%D1%81%D1%82%D1%80%D0%BE%D0%BA%D0%B8-cli-%D0%BA%D0%BE%D0%BC%D0%BC%D1%83%D1%82%D0%B0%D1%82%D0%BE%D1%80%D0%B0)

### Часть 1. Настройка основных параметров устройства

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_1.jpg)
> Сеть согласно топологии в ПО СРТ
```
On Switch:
Switch# delete flash:vlan.dat
Switch# erase startup-config
Switch# reload
```
```
On Router:
Router# erase startup-config
Router# reload
```

##### Настроим маршрутизатор согласно заданию.
```
router> enable
router# configure terminal
router(config)# no ip domain-lookup
router(config)# enable secret class
router(config)# line console 0
router(config-line)# password cisco
router(config-line)# login
router(config)# line vty 0 4
router(config-line)# password cisco
router(config-line)# login
router(config)# service password-encryption
router(config)# banner motd #
Warning!!!
Warning!!!
Warning!!!
Private territory of 5th homework
Strongly recoomend to leave the area
Warning!!!
Warning!!!
Warning!!! #
router(config)# interface g0/0/1
router(config-if)# ip address 192.168.1.1 255.255.255.0
router(config-if)# no shutdown
router# copy running-config startup-config
```
##### Настройте компьютер PC-A.

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_2.jpg)

##### Проверьте подключение к сети.

Отправим эхо-запрос на маршрутизатор R1, с PC-A, командой ping.

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_3.jpg)

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH


#### Настройте аутентификацию устройств.
```
router(config)# hostname R1
R1(config)# ip domain-name domaine
```

#### Создайте ключ шифрования с указанием его длины.
```
R1(config)#crypto key generate rsa
The name for the keys will be: R1.domain
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

R1(config)#
```
#### Создайте имя пользователя в локальной базе учетных записей.

Настроим имя пользователя, используя admin в качестве имени пользователя и Adm1nP@55 в качестве пароля.
```
R1(config)#username admin secret Adm1nP@55
```
#### Активируйте протокол SSH на линиях VTY.
Активируем протоколы Telnet и SSH на входящих линиях VTY с помощью команды *transport input*.
```
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
```
Изменим способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.
```
R1(config-line)# login local
R1(config-line)# end
```
#### Cохраните текущую конфигурацию в файл загрузочной конфигурации.
```
R1# copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
R1#
```
##### Запустите Tera Term с PC-A.

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_4.jpg)

##### Установите SSH-подключение к R1. Use the username admin and password Adm1nP@55. У вас должно получиться установить SSH-подключение к R1.

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_5.jpg)

### Часть 3. Настройка коммутатора для доступа по протоколу SSH

#### Настройте основные параметры коммутатора.

Произведём стандартную первичную настройку коммутатора
```
switch> enable
switch# configure terminal
switch(config)# no ip domain-lookup
switch(config)# enable secret class
switch(config)# line console 0
switch(config-line)# password cisco
switch(config-line)# login
switch(config)# line vty 0 15
switch(config-line)# password cisco
switch(config-line)# login
switch(config)# service password-encryption
switch(config)# banner motd $ Authorized Users Only! $
switch(config)# interface vlan 1
switch(config-if)# ip address 192.168.1.11 255.255.255.0
switch(config-if)# no shutdown
Switch# copy running-config startup-config
```
#### По аналогии с роутером произведём настройку ssh на коммутаторе
```
switch(config)# hostname S1
S1(config)# ip domain-name domainS
S1(config)#crypto key generate rsa
The name for the keys will be: S1.domainS
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
S1(config)#
S1(config)# username admin secret Adm1nP@55
S1(config)# line vty 0 15
S1(config-line)# transport input ssh
S1(config-line)# login local
S1(config-line)# end
```

#### Установите соединение с коммутатором по протоколу SSH.

Запустите программу Tera Term на PC-A, затем установите подключение по протоколу SSH к интерфейсу SVI коммутатора S1.

![](https://github.com/TheManWhoSpeakInHands/Network-Engineer.-Basic/blob/2a89cfc5ba9b4a70876311139fc3ee3dbc052a36/lab5(lecture12)/Screenshot_6.jpg)

### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора


#### Необходимо установить соединение с маршрутизатором по протоколу SSH, используя интерфейс командной строки коммутатора.

##### Посмотрите доступные параметры для клиента SSH в Cisco IOS.


Откройте окно конфигурации

Используйте вопросительный знак (?), чтобы отобразить варианты параметров для команды ssh.
```
S1#ssh ?
```
>-l  Log in using this user name
>
>-v  Specify SSH Protocol Version
  
##### Установите с коммутатора S1 соединение с маршрутизатором R1 по протоколу SSH.

(Чтобы подключиться к маршрутизатору R1 по протоколу SSH, введите команду **–l admin**. Это позволит вам войти в систему под именем admin. При появлении приглашения введите в качестве пароля Adm1nP@55)

```
S1# ssh -l admin 192.168.1.1
Password:

Authorized Users Only!
R1>
```

```
S1#ssh -l admin 192.168.1.1
Password: 

Warning!!!
Warning!!!
Warning!!!
Private territory of 5th homework
Strongly recoomend to leave the area
Warning!!!
Warning!!!
Warning!!!
R1>
```
##### Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, нажмите комбинацию клавиш Ctrl+Shift+6. Отпустите клавиши Ctrl+Shift+6 и нажмите x. Отображается приглашение привилегированного режима EXEC коммутатора.
```
R1>
S1#
```
##### Чтобы вернуться к сеансу SSH на R1, нажмите клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, нажмите клавишу Enter еще раз.
```
S1#
[Resuming connection 1 to 192.168.1.1 ... ]
R1>
R1(config)#
S1#
S1#
[Resuming connection 1 to 192.168.1.1 ... ]
R1(config)#
```
##### Чтобы завершить сеанс SSH на маршрутизаторе R1, введите в командной строке маршрутизатора команду exit.
```
R1# exit
[Connection to 192.168.1.1 closed by foreign host]
S1#
R1(config)#
R1(config)#ex
R1#ex
[Connection to 192.168.1.1 closed by foreign host]
S1#
```
Вопрос:

Какие версии протокола SSH поддерживаются при использовании интерфейса командной строки?

>version1 & version2


### Вопрос для повторения

Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя?

>Создать несколько пользовательских учетных записей
