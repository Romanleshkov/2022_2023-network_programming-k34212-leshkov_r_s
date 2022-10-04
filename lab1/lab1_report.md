University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2022/2023
Group: K34212
Author: Leshkov Roman Sergeevich
Lab: Lab1
Date of create: 20.09.2022
Date of finished: 

Цель работы: 
Целью данной работы является развертывание виртуальной машины в облачном сервисе с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox

Ход работы:
1. Разворачивание сервера

В качестве облачного провайдера выбран YandexCloud. После регистрации на домашней странице(консоль) перейти на вкладку **Compute Cloud**, далее при нажатии кнопки **Создать ВМ** откроется меню создания виртуальной машины, в котором необходимо задать *имя, образ системы, файловое хранилище, вычислительные ресурсы и SSH-ключ*, и после нажать кнопку **Создать ВМ**. 
В лабораторной работе были заданы следующие параметры: 
  - имя - ubuntu-leshkov
  - образ системы - Ubuntu22.04
  - файловое хранилище - 15 ГБ
  - вычислительные ресурсы
    - платформа - Intel Ice Lake
    - гарантированная доля vCPU - 20%
    - vCPU - 2
    - RAM - 1 ГБ
    
Для генерации SSH-ключа в cmd.exe или powershell.exe ввести:
  
      ssh-keygen -t ed25519
      
После указания имени файлов и пароля закрытого ключа в папке *C:\Users\<имя_пользователя>\.ssh\** будут сохраненены два файла - файл закрытого ключа без расширения и файл открытого ключа с расширением *.pub*. В облачном сервисе указывается содержимое файла открытого ключа в одну строку.
  
После запуска машины в облаке для подключения к ней необходимо запомнить выданный ей публичный IP-адрес, посмотреть его можно в вкладке *Compute Cloud/Виртуальные машины/<имя машины>/Обзор/Сеть*. В cmd.exe или powershell.exe вводится команда:
  
      ssh <имя_пользователя>@<публичный_IP-адрес_виртуальной_машины>
      
После ввода команды и последующего ввода пароля происходит подключение.
  
Для дальнейшего выполнения лабораторной необходимо установить Ansible и желательно обновить ПО, для этого в терминале вводятся команды:
  
      sudo apt update & sudo apt upgrade
      sudo do-release-upgrade
      sudo apt install python3-pip
      sudo pip3 install ansible
      ansible --version
      
Теперь приготовления к настройке завершены.
  
2. Установка RouterOS в VirtualBox

Образ RouterOS скачивается с сайта https://mikrotik.com/download из раздела **Cloud Hosted Router** в формате **vdi**. В VirtualBox при создании машины указывается тип - Other, версия Other/Unknown (64-bit), а в качестве хранилища указывается ранее скаченный файл в формате *.vdi*. В настройке ВМ в одном из адаптеров указывается тип подключения: *Сетевой мост*, тип адаптера *Intel PRO/1000 MT Desktop (82540EM)*, неразборчивый режим - *Разрешить всё* и обновляется MAC-адрес.
  
 После запуска в качестве логина вводится *admin* и задается новый пароль. На этом предварительная настройка RouterOS завершена.
  
3. Настройка OpenVPN на сервере

Для настройки OpenVPN необходимо установить два пакета: OpenVPN и Easy-RSA, для этого в терминале вводится команда
  
       sudo apt install openvpn easy-rsa
       
Для предотвращения будующих проблем пакет скриптов Easy-RSA необходимо скопировать в другую директорию например в */etc/openvpn/easy-rsa*:
  
      sudo mkdir /etc/openvpn/easy-rsa
      sudo cp -R /usr/share/easy-rsa /etc/openvpn/
      
Далее необходимо создать сервер сертификации, для этого нужно запустить генерацию папки pki и скриптов для центра сертификации:
  
      cd /etc/openvpn/easy-rsa/
      sudo ./easyrsa init-pki
      
Для создания открытого ключа (сертификата) сервера запускается скрипт *build-ca* (после ввода будет запрошено введение пароля):
  
      sudo ./easyrsa build-ca

Для обмена ключами между клиентом и сервером необходимо создать еще один ключ:
  
      sudo ./easyrsa gen-dh
  
Для генерации закрытого ключа сервера вводится:
  
      sudo ./easyrsa build-server-full server nopass
      
Здесь *server* - имя сервера и имя файлов созданных ключей (в лаборатоной указано имя - UbuntuOvpn)

Теперь все созданные ключи можно скопировать в одну в одну папку:

    cp ./pki/ca.crt /etc/openvpn/ca.crt
    cp ./pki/dh.pem /etc/openvpn/dh.pem
    cp ./pki/issued/UbuntuOvpn.crt /etc/openvpn/UbuntuOvpn.crt
    cp ./pki/private/UbuntuOvpn.key /etc/openvpn/UbuntuOvpn.key
    
Для создания конфигурационного файла можно использовать шаблон от OpenVPN как основу:

    cat /usr/share/doc/openvpn/examples/sample-config-files/server.conf | sudo tee /etc/openvpn/UbuntuOvpn.conf

В скопированном файле нужно указать порт, протокол, ключи и сеть:

    port 1194
    proto tcp
    dev tun
    ca ca.crt
    cert UbuntuOvpn.crt
    key UbuntuOvpn.key
    dh dh.pem
    server 10.8.0.0 255.255.255.0
    ifconfig-pool-persist /var/log/openvpn/ipp.txt
    ;push "redirect-gateway def1 bypass-dhcp"
    push "dhcp-option DNS 8.8.8.8"
    push "dhcp-option DNS 77.88.8.8"
    keepalive 10 120
    cipher AES-256-CBC
    persist-key
    persist-tun
    status /var/log/openvpn/openvpn-status.log
    verb 3

Для запуска OpenVPN по конфигу выполнить:

    sudo openvpn /etc/openvpn/UbuntuOvpn.conf
    
Далее можно запускать так:

    sudo systemctl start openvpn@UbuntuOvpn

Для завершения настройки серверной части необходимо настроить ip-forwarding для того, чтобы клиен имел доступ к внешней сети сервера:

    sudo sysctl -w net.ipv4.ip_forward=1
    
Осталось настроить бранмауэр, нужно посмотреть интерфейсы для внешней сети и для внутренней и разрешить трафик между ними:

    ip -br a
    ![image](https://user-images.githubusercontent.com/92050519/193788578-1c8913a0-8df6-4221-85f7-8b11eca5bab2.png)
    sudo iptables -I FORWARD -i tun0 -o eth0 -j ACCEPT
    sudo iptables -I FORWARD -i eth0 -o tun0 -j ACCEPT

4. Настройка OpenVPN на клиенте

Клиенту OpenVPN для работы нужны три файла 
- сертификат сервера
- сертификат клиента
- ключ клиента

Для удобства сгенировать ключи клиента можно на сервере:

    sudo ./easyrsa build-client-full RouterOS nopass

Осталось переместить ключи в отдельную папку, например в */etc/openvpb/clients/RouterOS*:

    sudo mkdir clients
    sudo mkdir clients/RouterOS
    sudo cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/clients/RouterOS/
    sudo cp /etc/openvpn/easy-rsa/pki/issued/RouterOS.crt /etc/openvpn/clients/RouterOS/
    sudo cp /etc/openvpn/easy-rsa/pki/private/RouterOS.key /etc/openvpn/clients/RouterOS/
    
Осталось перенести ключи на клиент - RouterOS. Во-первых, необходимо перенести ключи на хостовую машину с помощью SSH (на хостовой машине): 

    scp -r sudo rlesh@178.154.204.232:/etc/openvpn/clients/RouterOS D:\Users\tyk\HW\4\СетевоеПрограммирование\1

Далее следует открыть веб-интерфейс RouterOS и перенести туда загруженные три файла.

![image](https://user-images.githubusercontent.com/92050519/193788796-2dac0c03-496f-42ea-bf5a-79ae36b8e51a.png)

Затем, во вкладке *System/Certificates* добавить ключи. 

![image](https://user-images.githubusercontent.com/92050519/193788940-ea4cf716-5931-4e10-bfca-f46740941983.png)

Во вкладке *PPP* добавить *OVPN Client*.

![image](https://user-images.githubusercontent.com/92050519/193789049-f68ec664-13a6-4657-81b7-24d40437d308.png)

Указать внеешний адрес сервера, имя пользователя и сертификат.

![image](https://user-images.githubusercontent.com/92050519/193789238-d811c91f-b99d-4111-88f5-40b46e4039ea.png)

Проверить соединение.

![image](https://user-images.githubusercontent.com/92050519/193790262-4512c73e-5b20-4771-ad1c-a73505aca01f.png)
![image](https://user-images.githubusercontent.com/92050519/193790366-aab96aae-4855-41b7-ab5d-e74034e17ed9.png)
![image](https://user-images.githubusercontent.com/92050519/193790451-e777aca1-bb74-4bc1-a3ec-398e1c417780.png)

Вывод: в ходе выполнения лабораторной работы был получен базовый опыт настройки VPN, используя OpenVPN, и работы с облачным провайдером на примере YandexCloud.
