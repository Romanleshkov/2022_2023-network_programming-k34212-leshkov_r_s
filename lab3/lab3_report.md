University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)
Year: 2022/2023
Group: K34212
Author: Leshkov Roman Sergeevich
Lab: Lab3
Date of create: 16.11.2022
Date of finished: 13.01.2023

cd /etc
sudo apt install docker
sudo apt install docker-compose
sudo git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
sudo tee docker-compose.override.yml <<EOF
version: '3.4'
services:
  netbox:
    ports:
      - 8000:8080
EOF
sudo docker-compose pull
sudo docker-compose up
