**Цель автоматизации сетей связи, средства достижения этой цели**

Автоматизация сети служит для упрощения и повышения эффктивности работы сетевого инжинера, автоматизируя 
задачи, котрые без автоматизации инженер писал бы вручную, оставляя выполнение задач на программные решения, что ускоряет выполнение задач и сводит человеческий фактор к минимуму.

В лабараторных использовался Ansible, как система управления конфигурациями, что в паре с Netbox, дало возможность реализовать цетнр истины, и написать плейбук для обновления конфига группы роутеров одним запуском.

Также в лабораторных использовались Vagrant и Docker, которые позволяли автоматизировать загрузку и настройку сервисов и окружения. 

Еще одним примером автоматизации можно назвать фаерволл, который проверяет все входящие пакеты, а пропускает только те, что разрешены правилами (или не запрещены).

Время потраченное на проверку и размещение написанного кода также можно автоматизировать используя CI/CD подход, котрый при прохождении всех тестов запущенных автоматичемки развертывает приложение в сервер.

Как было сказано равнее, для длстижения автоматизации сети нужно использовать систему управления конфигурациями, например Ansible, котрая позволяет как запрашивать конфиги, так и выдавать, IPAM - средство для управления ip-адресами, используя Netbox можно получить, как IPAM, так и инвентарную систему, что хранит общую информацию о оборудовании в сети. Используя Ansible с бд, можно создать что-то вроде системы мониторинга и резервного копирования, которая по расписанию запрашиваеи конфиги и сравнивает их со старыми версиями, фиксируя изменения и сообщая о них.



**Что есть SDN, зачем он нужен и как с ним жить?**

Суть SDN - отделение управления трафикамии с оборудования в отдельное ПО. В итоге все управление(маршрутизация и коммутация) переносится на выделенный сервер. В итоге для свичей оставалось только физическая часть - перенаправление трафика, и хранение таблиц для перенаправления трафика, а на сервер возложен рассчет путей и управление маршрутизаторами и свичами через API

Для связи сервера с свичами используется протокол OpenFlow. Коммутатор с Openflow содержит таблицу коммутации для перенаправления трафика.


Плюсы:
Централизованное упраление сетью с сервера, используя программы -> упрощение конфигурирования сети
Экономия на дорогих свичах

недостатки:

требеет доп оборудование
