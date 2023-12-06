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

Установим Redis с помощью команды: 
```
sudo apt-get install -y redis-server
```
Скачиваем архив netbox

![Screenshot from 2023-12-05 19-47-54](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/645597fa-6e87-4d9f-99c2-92da634dfe92)

Добавляем пользователя

![Screenshot from 2023-12-05 19-49-24](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/0be3571d-91a5-40e6-9607-007a990c2e37)

Сгенирируем секретный ключ с помощью библиотеки python

![Screenshot from 2023-12-05 19-50-12](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/ea4941fc-b386-4941-9d3a-a0d3cbd212d6

Открываем конфигурационный файл configuration.py и редактируем его:

Укажем параметры: ALLOWED_HOSTS, DATABASE и SECRET_KEY.

![Screenshot from 2023-12-05 19-52-11](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/3cd885e4-f0ad-4d9f-bdb5-91dd5c45366a)

Внутри вирутального окружения python был создан superuser для дальнейшего управления netbox. 
```
python3 manage.py createsuperuser
```
Соберем статистику с помощью команды:
```
python3 manage.py collectstatic --no-input
```
Установим Ansible модули для Netbox. Для возможности запуска netbox были сконфигурированы nginx и gunicorn.
Добавим хост в файл nginx.conf.

#### 2. Заполнение инфомации о CHR

Переходим по ip адресу виртуальной машины и видим интерфейс Netbox:
![5-5](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/182696fa-6d62-4f68-8bc9-9e1008988237)

Добавляем устройства CH1 и CH2 и внесем информацию об интерфесах и IP-адресах:
![6](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/181b56cf-0989-4b4f-ac0d-5908aa90a2a8)

#### 3. Сбор данных Netbox
Сохраним данные из Netbox в отдельный файл netbox_inventory.yml с использованием модуля netbox.netbox.nb_inventory в Ansible.
```
plugin: netbox.netbox.nb_inventory
api_endpoint: https://51.250.29.83
token: токен
validate_certs: False
config_context: False
group_by:
  - device_roles
interfaces: 'True'
```
Всю информацию сохраняем в файл nb_inventory_result.yml.

#### 4. Настройка сценариев для обоих CHR.
Cоздадим Playbook для изменения имени пользователя и ip адреса:
```
- name: Set CHR
  hosts: ungrouped
  tasks:
    - name: Set Name
      community.routeros.command:
        commands:
          - /system identity set name="{{interfaces[0].device.name}}"
    - name: Set IP
      community.routeros.command:
        commands:
        - /ip address add address="{{interfaces[0].ip_addresses[1].address}}" interface="{{interfaces[0].display}}"
```
Запускаем playbook и в каждом роутере выведим ip адреса:
Добавлен интерфейс Lo1 с ip-адресом.

<img width="350" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/fdf05904-e17f-4617-a635-4d4059cad0a0">

<img width="350" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/82f10e77-ee7b-4353-a3a5-ed346a5f456c">

Проверим ping до виртуальной машины:

<img width="400" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/833f4903-3ccb-4f95-89d4-e78dc40b9c49">

Cоздадим Playbook для получения серийного номера CHR и установки его в профиль Netbox
```
- name: Serial Numbers
  hosts: ungrouped
  tasks:
    - name: Serial Number
      community.routeros.command:
        commands:
          - /system license print
      register: license_print
    - name: Name
      community.routeros.command:
        commands:
          - /system identity print
      register: identity_print
    - name: Serial Number to Netbox
      netbox_device:
        netbox_url: http://127.0.0.1:8000
        netbox_token: <API token>
        data:
          name: "{{identity_print.stdout_lines[0][0].split(' ').1}}"
```
Выполним данный playbook:
Проверяем заполнение поля Serial Number:
<img width="329" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/42be1931-0286-470b-be23-9bbac85093c5">
<img width="306" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/ce83a28b-dfe9-450c-93a9-885940c1b442">

#### 5. Вывод

В результате выполнения данной лабораторной работы с помощью Ansible и Netbox была собрана вся возможнуя информация об устройствах и сохранена в отдельном файле.
