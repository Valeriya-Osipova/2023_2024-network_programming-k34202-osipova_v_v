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

localip 172.16.0.1, 
remoteip 172.16.0.2-254

В файлике /etc/pptpd.conf в помощью команды:
```
sudo vim /etc/pptpd.conf
```
