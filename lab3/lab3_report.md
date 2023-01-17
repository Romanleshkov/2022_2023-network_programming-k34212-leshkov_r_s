University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2022/2023
Group: K34212
Author: Leshkov Roman Sergeevich
Lab: Lab3
Date of create: 16.11.2022
Date of finished: 15.01.2023

Цель работы: 
Написать плейбуки для занесения ионформации с Netbox на роутеры и с роутеров на Netbox.
Ход работы:
Для начала поднимается Netbox в контейнере, для этого необходимо установить docker и docker-compose.
    
    sudo apt install docker
    sudo apt install docker-compose

После скачивается репозиторий для развертывания контейров с github.

    cd /etc
    sudo git clone -b release https://github.com/netbox-community/netbox-docker.git
    cd netbox-docker
    
В файле env/netbox.env, можно поменять логин, пароль и API-токен. Далее, создается файл docker-compose.override.yml:
    
    sudo tee docker-compose.override.yml <<EOF
    version: '3.4'
    services:
      netbox:
        ports:
          - 8000:8080
    EOF
    
В созданном файле docker-compose.override.yml указан порт хоста 8000, что значит, что Netbox доступен по адресу <адрес хоста>:8000/. 
После установки настроек запускаются контейнеры:

    sudo docker-compose pull
    sudo docker-compose up -d
    
Ключ -d запускает контейнеры в фоне, оставляя доступ к терминалу.

В браузере открываетcя <адрес хоста>:8000/


---
plugin: netbox.netbox.nb_inventory
api_endpoint: http://localhost:8000/
token: 0123456789abcdef0123456789abcdef01234567
validate_certs: False
config_context: False
group_by:
   - device_roles
compose: {ansible_network_os: community.routeros.routeros, ansible_connection: ansible.netcommon.network_cli}
interfaces: 'True'

- name: Get configuration information from Netbox
  connection: local
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
    - name: Save data about devices from Netbox
      copy:
        content: "{{ item.value }}"
        dest: ./files/configs/netbox_data_{{ item.key }}.log
      loop:
        "{{ hostvars | dict2items }}"

ansible-playbook netbox_data_get.yml -i netbox_inventory.yml

                                           


- name: Set data from Netbox to routers
  hosts: device_roles_routers
  gather_facts: false
  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: community.routeros.routeros
    ansible_user: Ansible
    ansible_password: Ansible

  tasks:
    - name: Change name
      routeros_command:
        commands:
          - /system identity set name="{{ inventory_hostname }}"
    - name: Add interfaces
      routeros_command:
        commands:
          - /interface bridge add name="{{ item.display }}"
          - /ip address add address="{{ item.ip_addresses[0].address}}" interface="{{ item.display }}"
      loop:
        "{{ interfaces }}"


ansible-playbook netbox_data_set.yml -i netbox_inventory.yml
pip3 install pynetbox
               
                                           
- name: Set data from routers to Netbox
  hosts: device_roles_routers
  gather_facts: false
  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: community.routeros.routeros
    ansible_user: Ansible
    ansible_password: Ansible
    api_endpoint: http://localhost:8000/
    token: 0123456789abcdef0123456789abcdef01234567
  tasks:
    - name: Get routers config
      routeros_command:
        commands:
          - /system license print
      register: router_config
    - name: Save license on Netbox
      netbox.netbox.netbox_virtual_machine:
        netbox_url: "{{ api_endpoint }}"
        netbox_token: "{{ token }}"
        data:
          name: "{{ inventory_hostname }}"
          custom_fields:
            license: "{{ router_config.stdout_lines[0][-2:] | join('')}}\n"
        state: present
                                           
                                           
ansible-playbook netbox_data_send.yml -i netbox_inventory.yml
                                           
                                           
                                           
