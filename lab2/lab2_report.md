## Отчет по лабораторной работе №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2023/2024

Group: K34202

Author: Osipova Valeriya Vladimirovna

Lab: Lab2

Date of create: 02.11.2023

Date of finished: 

### Цель работы
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

### Ход работы

#### 1. Создание сертификатов для второго роутера
```console
valery@net-prog:/etc/openvpn/easy-rsa$ sudo ./easyrsa gen-req chr2 nopass
valery@net-prog:/etc/openvpn/easy-rsa$ sudo ./easyrsa sign-req client chr2
valery@net-prog:/etc/openvpn/easy-rsa$ sudo cp ./pki/private/chr2.key /etc/openvpn/certs/
valery@net-prog:/etc/openvpn/easy-rsa$ sudo cp ./pki/issued/chr2.crt /etc/openvpn/certs/
```

#### 2. Импорт сертификатов
```console
[admin@MikroTik] > certificate import file-name=ca.crt 
passphrase: *** 
     certificates-imported: 1
     private-keys-imported: 0
            files-imported: 1
       decryption-failures: 0
  keys-with-no-certificate: 0

[admin@MikroTik] > certificate import file-name=chr2.crt 
passphrase: *** 
     certificates-imported: 1
     private-keys-imported: 0
            files-imported: 1
       decryption-failures: 0
  keys-with-no-certificate: 0

[admin@MikroTik] > certificate import file-name=chr2.key 
passphrase: *** 
     certificates-imported: 0
     private-keys-imported: 1
            files-imported: 1
       decryption-failures: 0
  keys-with-no-certificate: 0
```

#### 3. Добавление OpenVPN соединения
```console
[admin@MikroTik] > /interface ovpn-client add certificate=chr2.crt_0 cipher=aes256 connect-to=51.250.47.204 mac-address=02:66:F6:83:E0:77 name=ovpn-out1 port=993 user=admin
```

#### 4. Создание файла inventory
```conf
[routers]
chr1 ansible_host=10.8.0.6 rid=10.10.1.1
chr2 ansible_host=10.8.0.10 rid=10.10.1.2

[routers:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=community.routeros.routeros
ansible_user=admin
ansible_ssh_pass=admin
```

#### 5. Получение ифромации о хостах
```console
valery@net-prog:~$ ansible-inventory --list -i inventory
{
    "_meta": {
        "hostvars": {
            "chr1": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.8.0.6",
                "ansible_network_os": "community.routeros.routeros",
                "ansible_ssh_pass": "admin",
                "ansible_user": "admin",
                "rid": "10.10.1.1"
            },
            "chr2": {
                "ansible_connection": "ansible.netcommon.network_cli",
                "ansible_host": "10.8.0.10",
                "ansible_network_os": "community.routeros.routeros",
                "ansible_ssh_pass": "admin",
                "ansible_user": "admin",
                "rid": "10.10.1.2"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "routers"
        ]
    },
    "routers": {
        "hosts": [
            "chr1",
            "chr2"
        ]
    }
}
```

#### 6. Выполним пинг 

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/399e76b8-ea73-4498-8336-59123914252f)

#### 7. Конфигурационный файл для playbook

```yml
- name: Configure
  hosts: routers
  gather_facts: false
  tasks:
    - name: add user
      routeros_command:
        commands :
          - /user add name=valery password=remote group=full
    - name: set NTP client
      routeros_command:
        commands :
          - /system ntp client set enabled yes primary-ntp [:resolve @.ru.pool.ntp.org] secondary-ntp [:resolve 1.ru.pool.ntp.org]
    - name: set OSPF
      routeros_command:
        commands :
          - /ip address add address={{rid}}/24 interface=ether1
          - /routing ospf network add network=10.10.1.0/24 area=backbone

- name: Get information
  hosts: routers
  gather_facts: false
  tasks:
    - name: get configs
      register: configuration
      community.routeros.facts:
        gather_subset:
          - config
    - name: collect OSPF topology
      register: ospf
      routeros_command:
        commands :
          - /routing ospf neighbor print
    - name: Print configuration
      debug:
        msg: "{{ configuration }}"
    - name: Print ospf
      debug:
        msg: "{{ ospf }}"
```

Выполним команду, результат которого запишем в [res.txt](res.txt)
```console
valery@net-prog:~$ ansible-playbook -i inventory play.yml >> res.txt
```

#### 8. Проверка связанности роутеров

<img width="478" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/d8cac383-f432-43b5-96e7-1c0a53030ccb">


Пинг chr2 с chr1

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/5b4db887-a9c0-49fa-88e4-3b7ef28a3f0c)

Пинг chr1 с chr2

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/8f79c106-059f-4e6b-b3e7-08fef104b836)

#### 9. Вывод
В результате выполнения данной лабораторной работы удалось ознакомиться с системой управления конфигурацией Ansible, использующаяся для автоматизации настройки и развертывания программного обеспечения. С помощью Ansible были настроены несколько сетевых устройств, также была собрана информация о них. Был собран файл Inventory.
