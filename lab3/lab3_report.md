## Отчет по лабораторной работе №3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2023/2024

Group: K34202

Author: Osipova Valeriya Vladimirovna

Lab: Lab3

Date of create: 30.11.2023

Date of finished: 

### Цель работы
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

### Ход работы

#### 1. Устанавливаем Netbox
Сначала установим postgresql и подключимся к нему. 
Также создадим базу данных с помощью команд 
```
CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'PstPass';
GRANT ALL PRIVILEGES ON DA;
GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;
```
![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/2e51c2fc-286c-4d7d-aea5-cc2b5a1e8ce8)



