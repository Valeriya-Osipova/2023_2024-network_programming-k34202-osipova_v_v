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
#### 1. Расвертывание виртуальной машины.
Для создания виртуальной машины, где будет находится OpenVPN сервер была выбрана платформа Яндекс.Облако. На ней была создана вм с Ubuntu 22.
<img width="913" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/618bca49-c5f5-459e-b02f-1aa17e5123e8"> \
Получаем публичный ip адрес, переходим в консоль и подключаемся к созданной виртуальной машине с помощью команды 
```
ssh admin@158.160.77.48
```
#### 2. Поднимаем VPN
В консоли выполняем следующие команды:
```
apt-get install ppp pptpd
```
Раскомментируем строки 

localip 172.16.0.1 \
remoteip 172.16.0.2-254 

В файлике /etc/pptpd.conf в помощью команды:
```
sudo vim /etc/pptpd.conf
```
<img width="358" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/9691cbbf-98c2-4bb3-97d2-eb939cfd6398">

localip – ip адрес из выбранной подсети, который будет являться локальным шлюзом для клиентов VPN.\
remoteip – пул ip адресов для раздачи клиентам VPN. \

Также раскомментируем строку `net.ipv4.ip_forward=1` в файле /etc/sysctl.conf \
В файл /etc/ppp/pptpd-options добавляем строки: 
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
Добавляем пользователя для подключения к VPN серверу в файле /etc/ppp/chap-secrets
```
user1	pptpd	password1	"*"
```
Перезапускаем сервис pptpd для применения новых настроек:
```
service pptpd restart
```
#### 3. Установка python3 и Ansible
Для установки python3 пишем команду
```
sudo apt install python3-pip
```
Для установки Ansible пишем команду
```
sudo pip3 install ansible-pylibssh
```

