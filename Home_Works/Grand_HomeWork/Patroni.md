<div align="center"><h2> 3. Настройка Patroni </h2></div>

***

> ### Немного теории:
  * Для работы в режиме высокой доступности `HA` (High Availability) нужно некоторое ПО, которое контроллирует кто сейчас `master` или `главный сервер`, а кто `secondary` или `реплика`
  * В PostgreSQL это очень важно чтобы не было `Split Brain` чтобы в кластере не появилось больше одного `master`
  * Подобные функции может выполнять следующее ПО:
    * [`Pacemaker and Corosync`](https://clusterlabs.org/)
      * Не умеет восстанавливать упавший `master/primary` или `replica/slave/standby`-это приходится делать вручную
    * [`Patroni`](https://patroni.readthedocs.io/en/latest/) ❗️Именно его мы и будет использовать
      * Умеет восстаанвливать упавший `master/primary` или `replica/slave/standby`
    * [`RepMgr`](https://www.repmgr.org/)
      * Нет защиты от двойного мастера (split brain);
      * Нет нужды в DCS.
    * [`Stolon`](https://github.com/sorintlab/stolon)
      * Проксирует все запросы в мастер ноду, нельзя давать нагрузку на реплики;
      * Мастер выбирается самостоятельно при switchover-e.
    * [`pg_auto_failover`](https://pg-auto-failover.readthedocs.io/en/main/intro.html)
    * [`Slony`](https://www.slony.info/)
   
*** 
### Установка и настройка `PostgreSQL 15`
  * На всех 3 ВМ ставим PostgreSQL 15
  ```bash
  sudo apt update && sudo apt upgrade -y -q && echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee -a /etc/apt/sources.list.d/pgdg.list && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && \
  sudo apt -y install postgresql-15
  ```
  * На всех 3 ВМ удаляем кластер по умолчанию
  ```bash
  sudo -u postgres pg_dropcluster 15 main --stop && \
  pg_lsclusters
  ```

*** 

### Установка и настройка `Patroni`
  * ❗️У нас будет использоваться `Consul` в качестве `DCS`.
  * Устанавливаем `Patroni` и необходимые пакеты на всех 3х ВМ
    ```bash
    sudo apt install -y python3 python3-pip python3-psycopg2 && \
    sudo pip3 install patroni[consul] && \
    sudo mkdir /etc/patroni
    ```
  * Версия `Patroni`
    ```bash
    sudo patroni --version
    ```
    ```console
    sudo patroni --version
    patroni 3.1.0
    ```

***
###  Создаем конфигурацию для `Patroni`
  * Создаем конфигурационный файл [`/etc/patroni/patroni.yml`](patroni.yml)
  * ❗️Разница для каждой ВМ в:
    * `name: pg-srvX`
    * `connect_address: "10.129.0.X:8008"`
    * `connect_address: "10.129.0.X:5432"`
    ```bash

    ```
    ```console
    ```
***    
###  Создаем настраиваем службу в ОС для `Patroni`
  * Создаем службу [`/usr/lib/systemd/system/patroni.service`](patroni.service) в ОС на каждой из 3х ВМ
  * ❗Обратите внимание, что в официальной документации предлагается не перезапускать автоматически службу (Restart=no). Это дает возможность разобраться в причине падения базы️
    ```bash
    sudo vim /usr/lib/systemd/system/patroni.service
    ```
    ```service
    [Unit]
    Description=Patroni service
    After=syslog.target network.target
    
    [Service]
    Type=simple
    User=postgres
    Group=postgres
    ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml
    ExecReload=/bin/kill -s HUP $MAINPID
    KillMode=process
    TimeoutSec=30
    Restart=no
    
    [Install]
    WantedBy=multi-user.target
    ```

  * Перечитываем конфигурацию systemd
    ```bash
    sudo systemctl daemon-reload
    ```

  * Стартуем наш сервис
    ```bash
    sudo systemctl start patroni
    ```

  * Также разрешаем автоматический старт при запуске сервера
    ```bash
    sudo systemctl enable patroni
    ```
    ```console
    ```
    ```console
    ```
    ```console
    ```

  * Смотрим текущее состояние работы сервиса:
    ```bash
    sudo systemctl status patroni
    ```
    ```console
    ```
    ```console
    ```
    ```console
    ```
    

***
###  Полезные команды `Patroni`
  * Посмотреть список нод
    ```bash
    patronictl -c /etc/patroni/patroni.yml list
    ```

  * Изменим параметры с помощью команды
    ```bash
    patronictl -c /etc/patroni/patroni.yml edit-config
    ```

  * Рестарт одной ноды
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart postgres postgresql-patroni1
    ```

  * Рестарт всего кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart postgres
    ```

  * Рестарт reload кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml reload postgres
    ```

  * Плановое переключение
    ```bash
    patronictl -c /etc/patroni/patroni.yml switchover postgres
    ```

  * Реинициализации ноды
    ```bash
    patronictl -c /etc/patroni/patroni.yml reinit postgres postgresql-patroni2
    ```

***

### :+1: Patroni установлен и настроен
