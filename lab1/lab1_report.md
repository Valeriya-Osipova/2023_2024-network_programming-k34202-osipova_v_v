## Отчет по лабораторной работе №1 "Установка CHR и Ansible, настройка VPN"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2023/2024

Group: K34202

Author: Osipova Valeriya Vladimirovna

Lab: Lab1

Date of create: 11.10.2023

Date of finished: 

### Цель работы
Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox

### Ход работы
#### 1. Развертывание виртуальной машины.
Для создания виртуальной машины, где будет находится VPN сервер была выбрана платформа Яндекс.Облако. На ней была создана вм с Ubuntu 22.

<img width="913" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/030c2b01-c579-45c1-a114-69d57b0e7ecb">

Получаем публичный ip адрес, переходим в консоль и подключаемся к созданной виртуальной машине с помощью команды 
```
ssh admin@62.84.120.115
```
#### 2. Установка python3 и Ansible
Для установки python3 пишем команду
```
sudo apt install python3-pip
```
Для установки Ansible пишем команду
```
sudo pip3 install ansible-pylibssh
```
#### 3. Поднимаем VPN
В консоли выполняем следующие команды:
```
apt-get install ppp pptpd
```

В файлике `/etc/pptpd.conf` добавим строки
```
localip 192.168.0.1
remoteip 192.168.0.2-200
```

`localip` – ip адрес из выбранной подсети, который будет являться локальным шлюзом для клиентов VPN.

`remoteip` – пул ip адресов для раздачи клиентам VPN.

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/6bd0edcc-0edc-447a-a233-eb49e96cb322)


localip – ip адрес из выбранной подсети, который будет являться локальным шлюзом для клиентов VPN.\
remoteip – пул ip адресов для раздачи клиентам VPN.

Также раскомментируем строку `net.ipv4.ip_forward=1` в файле `/etc/sysctl.conf`

В файл `/etc/ppp/pptpd-options` добавляем строки: 
```
mtu 1400
mru 1400
auth
require-mppe
```

Добавляем необходимые правила в iptables:
```
iptables -A INPUT -p gre -j ACCEPT
iptables -A INPUT -m tcp -p tcp --dport 1723 -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Добавляем пользователя для подключения к VPN серверу в файле `/etc/ppp/chap-secrets`
```
user1	pptpd	password1	"*"
```
Перезапускаем сервис pptpd для применения новых настроек:
```
service pptpd restart
```
#### 4. Установка CHR
Для установки виртуального роутера переходим на сайт `https://mikrotik.com/download` и скачиваем образ.

<img width="396" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/b2b47710-b435-469f-b41b-68a90b8dcb10">

Загруженный файл открываем в VirtualBox и запускаем машину.

Заходим в WinBox и определяем адрем роутера. Подключаемся к нему для дальнейшей настройки.

<img width="487" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/f8e59a36-7e4d-4123-85ac-6a7e9dfa260f">

Настраиваем подключение роутера:

Connect to: ip адрес виртуалки
user/password: user1/password1

<img width="542" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/109ecd80-9a85-4b21-9d8c-ec81bc85fd26">

<img width="333" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/d8887e45-eef0-4649-855f-1451b877ad16">

Получаем local и remote ip-адрес 192.168.0.2 / 192.168.0.1

#### 5. Ping роутера

Переходим в терминал и делаем пинг на адрес роутера: 192.168.0.2

<img width="400" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/7855d21e-cb52-4770-886f-f4af22233102">

#### 6. Вывод
В результате выполнения данной лабораторной работы была создана виртуальная машина на базе платформы Яндекс.облако с установленной системой контроля конфигураций Ansible. На компьютер был установлен CHR. Был настроен VPN туннель между сервером автоматизации и локальным CHR. Была проверена ip связанность между роутером и сервером.
