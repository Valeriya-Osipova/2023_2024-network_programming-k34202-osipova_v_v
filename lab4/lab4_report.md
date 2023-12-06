## Отчет по лабораторной работе №4 "Базовая 'коммутация' и туннелирование используя язык программирования P4"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2023/2024

Group: K34202

Author: Osipova Valeriya Vladimirovna

Lab: Lab4

Date of create: 02.12.2023

Date of finished: 07.12.2023

### Цель работы
Изучить синтаксис языка программирования P4 и выполнить 2 обучающих задания от Open network foundation для ознакомления на практике с P4.

### Ход работы

#### 1. Implementing Basic Forwarding

Склонируем репозиторий. Установим Vagrant, переходим в папку vm-ubuntu-20.04. Используя Vagrant, развернем тестовую среду: vagrant up. В результате установки на VirtualBox появляется виртуальная машина с аккаунтами vagrant и p4.

В данном задании необходимо дописать программу, реализующую перенаправление пакетов между коммутаторами, находящимися в разных сегментах сети. С помощью консольной команды make run эмулируем сеть, соответствующую представленной диаграмме:

<img width="674" alt="image" src="https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/4c6576b1-cd60-4c6f-89a0-4a224b8492fe">

Заходим в виртуальную машину под пользователем p4.

Выполняем команду make run, а затем проверяем доступность узлов:

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/54e9dde5-a54b-4801-8738-f8942e31c09d)

Пинг между хостами не работает, так как по умолчанию все пакеты автоматически отбрасываются.
Чтобы это исправить, необходимо внести изменения в файл basic.p4.

Добавляем в парсер заголовки ipv4 и ethernet:

```
parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start {
        /* TODO: add parser logic */
        transition parse_ethernet;
        #transition accept;
    }
    
    state parse_ethernet {
        packet.extract(hdr.ethernet);
        transition select(hdr.ethernet.etherType) {
            TYPE_IPV4 : parse_ipv4;
            default : accept;
        }    
    }

    state parse_ipv4 {
        packet.extract(hdr.ipv4);
        transition accept;
    }
}
```

Для пересылки ipv4 пакетов устанавливаем выходной порт, обновляем исходный mac-адрес и адрес назначения, меняем значение TTL, добавляем таблицу маршрутизации и условие проверки заголовка IPv4.
```
control MyIngress(inout headers hdr,
                  inout metadata meta,
                  inout standard_metadata_t standard_metadata) {
    action drop() {
        mark_to_drop(standard_metadata);
    }

    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {
        /* TODO: fill out code in action body */
        standard_metadata.egress_spec = port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }

    table ipv4_lpm {
        key = {
            hdr.ipv4.dstAddr: lpm;
        }
        actions = {
            ipv4_forward;
            drop;
            NoAction;
        }
        size = 1024;
        default_action = NoAction();
    }

    apply {
        /* TODO: fix ingress control logic
         *  - ipv4_lpm should be applied only when IPv4 header is valid
         */
        if (hdr.ipv4.isValid()) {
            ipv4_lpm.apply();
        }
    }
}
```

Добавляем заголовок:
```
control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        /* TODO: add deparser logic */
        packet.emit(hdr.ethernet);
        packet.emit(hdr.ipv4);
    }
}
```
Снова выполняем команду make run, а затем проверяем подключение - пакеты доходят.

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/397b741c-19f7-4bf0-b4a0-9853b21baf30)

Результирующий файл: [basic.p4](basic.p4)

#### 2. Implementing Basic Tunneling

Во данном задании перенаправление пакетов уже реализовано в шаблоне программы, необходимо дополнить шаблон таким образом, чтобы была возможность осуществлять туннелирование между всеми коммутаторами в сети, представленной на диаграмме:

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/b128f677-2dc8-40c5-96fd-222fe6ef649e)

Вносим изменения в файл basic_tunnel.p4.
Добавляем новый заголовок myTunnel_t

```
#// NOTE: added new header type
header myTunnel_t {
    bit<16> proto_id;
    bit<16> dst_id;
}
```

```
#// NOTE: Added new header type to headers struct
struct headers {
    ethernet_t   ethernet;
    myTunnel_t   myTunnel;
    ipv4_t       ipv4;
}
```

Изменяем парсер parse_ethernet: в зависимости от значения etherType происходит извлечение заголовка ipv4 или myTunnel.

```
state parse_ethernet {
    packet.extract(hdr.ethernet);
    transition select(hdr.ethernet.etherType) {
        TYPE_IPV4 : parse_ipv4;
        TYPE_MYTUNNEL : parse_myTunnel;
        default : accept;
    }
}
```

В парсер добавим функцию извлечения заголовка myTunnel:
```
state parse_myTunnel {
    packet.extract(hdr.myTunnel);
    transition select(hdr.myTunnel.proto_id) {
        TYPE_IPV4 : parse_ipv4;
        default : accept;    
    }
}
```

Определение выходного порта:
```
#// TODO: declare a new action: myTunnel_forward(egressSpec_t port)
action myTunnel_forward(egressSpec_t port) {
    standard_metadata.egress_spec = port;
}
```

Определение таблицы для маршрутизации пакетов:
```
#// TODO: declare a new table: myTunnel_exact
#// TODO: also remember to add table entries!
table myTunnel_exact {
    key = {
        hdr.myTunnel.dst_id: exact;
    }
    actions = {
        myTunnel_forward;
        drop;
        NoAction;
    }
    size = 1024;
    default_action = NoAction();
}
```

Функция apply использует таблицу, соответствующую типу пакета
```
apply {
    #// TODO: Update control flow
    if (hdr.ipv4.isValid() && !hdr.myTunnel.isValid()) {
        ipv4_lpm.apply();
    }
    if (hdr.myTunnel.isValid()) {
        myTunnel_exact.apply();
    }
}
```

Далее обновляем депарсер
```
control MyDeparser(packet_out packet, in headers hdr) {
    apply {
        packet.emit(hdr.ethernet);
        #// TODO: emit myTunnel header as well
        packet.emit(hdr.myTunnel);
        packet.emit(hdr.ipv4);
    }
}
```

Для проверки запускаем хосты h1 и h2.
Для начаоа отправляем пакет без данных, необходимых заголовку туннелирования. Итоговый пакет доходит до адресата и содержаит заголовки TCP, Ethernet, IPv4 и наше переданное сообщение.

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/008e71da-0eec-4f40-b7c5-b1f80e9c9094)

К сообщению добавляем данные для заголовка туннелирования - dst_id, соответствующий второму коммутатору. Пакет также доходит до адресата и содержит дополнительный заголовок MyTunnel.

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/ccb3cfbf-5409-4d30-922c-426c16b76f61)

Отправляем пакет на IP-адрес третьего коммутатора, но при этом указываем dst_id второго коммутатора. Пакет уходит на второй коммутатор, потому что при наличии корректного заголовка myTunnel реализуется таблица маршрутизации myTunnel_exact, и IP-адрес при этом не имеет значения.

![image](https://github.com/Valeriya-Osipova/2023_2024-network_programming-k34202-osipova_v_v/assets/64967406/ba75a544-d8c7-41c2-86fd-2df06c6a4a60)

#### 3. Вывод

В результате выполнения данной лабораторной работы удалось изучить синтаксис языка программирования P4 и выполнить 2 обучающих задания от Open network foundation для ознакомления на практике с P4.
