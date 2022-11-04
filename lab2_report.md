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
