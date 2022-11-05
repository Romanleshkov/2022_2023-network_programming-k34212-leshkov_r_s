University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2022/2023
Group: K34212
Author: Leshkov Roman Sergeevich
Lab: Lab2
Date of create: 5.11.2022
Date of finished: 

Цель работы: С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

Ход работы:
1. Установить второй CHR 

Образ RouterOS скачивается с сайта https://mikrotik.com/download из раздела **Cloud Hosted Router** в формате **vdi**. В VirtualBox при создании машины указывается тип - Other, версия Other/Unknown (64-bit), а в качестве хранилища указывается ранее скаченный файл в формате *.vdi*. В настройке ВМ в одном из адаптеров указывается тип подключения: *Сетевой мост*, тип адаптера *Intel PRO/1000 MT Desktop (82540EM)*, неразборчивый режим - *Разрешить всё* и обновляется MAC-адрес.
  
 После запуска в качестве логина вводится *admin* и задается новый пароль. На этом предварительная настройка RouterOS завершена.

3. Организовать второй OVPN Client на втором CHR

Клиенту OpenVPN для работы нужны три файла 
- сертификат сервера
- сертификат клиента
- ключ клиента

Для удобства сгенировать ключи клиента можно на сервере:

    sudo ./easyrsa build-client-full RouterOS2 nopass

Осталось переместить ключи в отдельную папку, например в */etc/openvpn/clients/RouterOS2*:

    sudo mkdir clients/RouterOS
    sudo cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/clients/RouterOS2/
    sudo cp /etc/openvpn/easy-rsa/pki/issued/RouterOS2.crt /etc/openvpn/clients/RouterOS2/
    sudo cp /etc/openvpn/easy-rsa/pki/private/RouterOS2.key /etc/openvpn/clients/RouterOS2/
    
Осталось перенести ключи на клиент - RouterOS2. Во-первых, необходимо перенести ключи на хостовую машину с помощью SSH (на хостовой машине): 

    scp -r sudo rlesh@178.154.204.232:/etc/openvpn/clients/RouterOS2 D:\Users\tyk\HW\4\СетевоеПрограммирование\1

Далее следует открыть веб-интерфейс RouterOS и перенести туда загруженные три файла.

![Screenshot_1](https://user-images.githubusercontent.com/92050519/200081173-1e3db164-b72f-448a-94c5-da2759073a3d.jpg)

Затем, во вкладке *System/Certificates* добавить ключи. 

![Screenshot_2](https://user-images.githubusercontent.com/92050519/200081217-1807c931-c925-43a8-b834-cf6db94de72b.jpg)

Указать внеешний адрес сервера, имя пользователя и сертификат.

![Screenshot_3](https://user-images.githubusercontent.com/92050519/200081255-c5d3d527-e0b3-403b-badc-8c85579a53c4.jpg)

5. Используя Ansible настроить сразу два CHR

Сначало необходимо установить Ansible:

    sudo pip install ansible
    
Для файлов Ansible нужно создать дирикторию:

    mkdir /etc/ansible
    
Для использования не настроек по умолчанию можно либо создать файл *ansible.cgf*, который будет использоваться как файл настроек по умолчанию для файлов в текущей директории, или создать файл с иным названием и указывать этот файл при вызове плейбука через флаг *-i*. В лабораторной работе создан файл *ansible.cfg*:
    
    touch /etc/nsible/ansible.cfg
    nano /etc/ansible/ansible.cfg
    
    [defaults]
    inventory = ./hosts
    remote_tmp = /etc/ansible/ansible_tmp/
    
В качестве файла inventory указан файл hosts, в этом файле должны быть прописаны устройства и параметры для подключения к ним:

    touch /etc/ansible/hosts
    nano /etc/ansible/hosts
    
    [routers]
    router1 ansible_host=10.8.0.4 ansible_ssh_user=Ansible ansible_ssh_pass=Ansible
    router2 ansible_host=10.8.0.8 ansible_ssh_user=Ansible ansible_ssh_pass=Ansible
    
    [routers:vars]
    ansible_connection=ansible.netcommon.network_cli
    ansible_network_os=community.routeros.routeros
    
В квадратных скобках указывается имя группы устройств, через *<имя группы>:vars* можно указать повторяющееся параметры для группы устройств. Параметр *ansible_host* указывается адрес устройства, *ansible_ssh_user* и *ansible_ssh_pass* указываются логин и пароль для ssh подключения. Параметр *ansible.netcommon.network_cli* используется для ssh соединения к сетевым устройсвам, а параметр *community.routeros.routeros* указывает операционную систему *RouterOS*, на которой работают CHR.

Команды для выполнения пишутся в таске, таск в плее, плей в плейбуке - файле с разрешением yml. Нужно создать файл и указать команды для выполения:
    
    touch /etc/ansible/data_set.yml
    nano data_set.yml
    
    - name: Routers config set
      hosts: routers
      gather_facts: false
      tasks:
      - name: Login_password set
        routeros_command:
        - /user add name=Ansible2 password=Ansible group=full
      - name: NTP Client set
        routeros_command:
          commands:
            - /system clock set time-zone-name=Europe/Moscow
            - /system ntp client set enabled=yes primary-ntp=88.147.254.230
      - name: OSPF config set
        routeros_command:
          commands:
            - /interface bridge add name=loopback
            - /ip address add address=4.4.4.4/32 interface=loopback
            - /routing ospf instance set router-id=4.4.4.4 number=0
            - /routing ospf network add network=192.168.0.0/24 area=backbone
            - /routing ospf network add network=192.168.0.0/24 area=backbone
            
![Screenshot_4](https://user-images.githubusercontent.com/92050519/200091796-67e7caa4-5a46-4254-8a5e-1db804e11571.jpg)

![Screenshot_8](https://user-images.githubusercontent.com/92050519/200091744-c0ffa075-0c6a-4118-b339-ea393fa3492f.jpg)

![Screenshot_9](https://user-images.githubusercontent.com/92050519/200091746-148cfda6-2e83-4a8e-92b9-25fc001ad27b.jpg)

![Screenshot_10](https://user-images.githubusercontent.com/92050519/200091748-389b65ba-6b3b-4d83-8c26-a7a3baa758d0.jpg)

![Screenshot_11](https://user-images.githubusercontent.com/92050519/200091753-ba8304fa-7a19-4cd1-a29a-11081eeb4bf9.jpg)

В этом плейбуке указан один плей, который называется *Routers config set*, он будет выполнен на устройствах в группе *routers*, будут выполнены три таски: создание пользователя, установка временной зоны на GSM +3 и сервера NTP, и настройка OSPF с router-id=4.4.4.4 и сетью 192.168.0.0/24.

Для сбора данных создан еще один плейбук *data_get.yml*:

    touch /etc/ansible/data_get.yml
    nano data_set.yml
    
    - name: Getting data from routers
      hosts: routers
      gather_facts: false
      tasks:
      - name: get_ospf_topology
        routeros_command:
          commands:
          - /routing ospf neighbor print
        register: ospf_topology
      - name: get_router_config
        routeros_command:
          commands:
          - /export compact
        register: router_config
      - name: Saving ospf topology
        copy:
          content: "{% for x in ospf_topology.stdout.0 if x!=''%}"{{x}}\n{% endfor %}"
          dest: /etc/ansible/files/ospf_topology_{{inventory_hostname}}.log
      - name: Saving router config
        copy:
          content: "{% for x in router_config.stdout.0 if x!=''%}"{{x}}\n{% endfor %}"
          dest: /etc/ansible/files/{{inventory_hostname}}_config.log

![Screenshot_5](https://user-images.githubusercontent.com/92050519/200091810-4025841a-cab3-462f-9b76-1a342c9d4a7f.jpg)

![Screenshot_6](https://user-images.githubusercontent.com/92050519/200095839-29605d9b-5603-4504-bb26-7d399e900230.jpg)

![Screenshot_7](https://user-images.githubusercontent.com/92050519/200095843-4c9028b2-f4df-4138-8948-a35caf5a1005.jpg)

Топология OSPF просматривается через соседей OSPF, так как в Router ID у двух роутеров прописан одинаковый, то делиться путями в одной зоне они не будут, а конфиг получен через команду *export compact*, в конце таски вывод консоли сохраняется в переменную через команду *register*, через блок *copy* переменная записывается в файл.

В итоге, получена следующая топология:

![Screenshot_16](https://user-images.githubusercontent.com/92050519/200093904-d27fd169-0baa-4a4a-b307-4a78e1e852c4.jpg)

По OVPN связь идет:

![Screenshot_12](https://user-images.githubusercontent.com/92050519/200093929-7c505726-090c-42f0-80c2-aabff75a70a7.jpg)

![Screenshot_13](https://user-images.githubusercontent.com/92050519/200093960-7291f11e-70c2-4b32-9bf7-0fd2fcb4f3c3.jpg)

![Screenshot_14](https://user-images.githubusercontent.com/92050519/200093959-4b58c4cf-a6b6-42eb-920c-c47adec3bb58.jpg)

![Screenshot_15](https://user-images.githubusercontent.com/92050519/200093966-0b0adb58-775a-484c-87a7-bbf7e699eed0.jpg)

Вывод: Ansible мощный инструмент для множественного конфигурирования устройств и сбора информации с устройств.
