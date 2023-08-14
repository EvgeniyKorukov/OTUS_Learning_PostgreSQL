  <div align="center"><h2> 6. Настройка HAProxy </h2></div>

***
### Немного теории:
  * Для балансировки нагрузки используется соответствующее ПО. В нашем случае балансировка будет между `PgBouncer`
  * В качестве балансировщика нагрузки могут выступать:
      * [`HAProxy`](https://www.haproxy.org/) ❗️Именно его мы и будет использовать
      * [`Nginx`](http://nginx.org/)

*** 
### Список ВМ с которыми идет взаимодействие на данном этапе
  :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
  |--------------:|---------------|
  | **`hap1`** | `10.129.0.11` |
  | **`hap2`** | `10.129.0.12` |      
  
***
### Установка `HAProxy`
  * Выполняем устаноку на всех 2х ВМ
    ```bash
    sudo apt -y install haproxy -y
    ```
    ```console
    ubuntu@hap1:~$ sudo apt -y install haproxy -y
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      liblua5.3-0
    Suggested packages:
      vim-haproxy haproxy-doc
    The following NEW packages will be installed:
      haproxy liblua5.3-0
    0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
    Need to get 1,657 kB of archives.
    After this operation, 3,811 kB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal/main amd64 liblua5.3-0 amd64 5.3.3-1.1ubuntu2 [116 kB]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 haproxy amd64 2.0.31-0ubuntu0.1 [1,541 kB]
    Fetched 1,657 kB in 0s (15.2 MB/s) 
    Selecting previously unselected package liblua5.3-0:amd64.
    (Reading database ... 102602 files and directories currently installed.)
    Preparing to unpack .../liblua5.3-0_5.3.3-1.1ubuntu2_amd64.deb ...
    Unpacking liblua5.3-0:amd64 (5.3.3-1.1ubuntu2) ...
    Selecting previously unselected package haproxy.
    Preparing to unpack .../haproxy_2.0.31-0ubuntu0.1_amd64.deb ...
    Unpacking haproxy (2.0.31-0ubuntu0.1) ...
    Setting up liblua5.3-0:amd64 (5.3.3-1.1ubuntu2) ...
    Setting up haproxy (2.0.31-0ubuntu0.1) ...
    Created symlink /etc/systemd/system/multi-user.target.wants/haproxy.service → /lib/systemd/system/haproxy.service.
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for rsyslog (8.2001.0-1ubuntu1.3) ...
    Processing triggers for systemd (245.4-4ubuntu3.22) ...
    ubuntu@hap1:~$ 
    ```
    ```console
    ubuntu@hap2:~$ sudo apt -y install haproxy -y
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      liblua5.3-0
    Suggested packages:
      vim-haproxy haproxy-doc
    The following NEW packages will be installed:
      haproxy liblua5.3-0
    0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
    Need to get 1,657 kB of archives.
    After this operation, 3,811 kB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal/main amd64 liblua5.3-0 amd64 5.3.3-1.1ubuntu2 [116 kB]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 haproxy amd64 2.0.31-0ubuntu0.1 [1,541 kB]
    Fetched 1,657 kB in 0s (14.2 MB/s) 
    Selecting previously unselected package liblua5.3-0:amd64.
    (Reading database ... 102602 files and directories currently installed.)
    Preparing to unpack .../liblua5.3-0_5.3.3-1.1ubuntu2_amd64.deb ...
    Unpacking liblua5.3-0:amd64 (5.3.3-1.1ubuntu2) ...
    Selecting previously unselected package haproxy.
    Preparing to unpack .../haproxy_2.0.31-0ubuntu0.1_amd64.deb ...
    Unpacking haproxy (2.0.31-0ubuntu0.1) ...
    Setting up liblua5.3-0:amd64 (5.3.3-1.1ubuntu2) ...
    Setting up haproxy (2.0.31-0ubuntu0.1) ...
    Created symlink /etc/systemd/system/multi-user.target.wants/haproxy.service → /lib/systemd/system/haproxy.service.
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for rsyslog (8.2001.0-1ubuntu1.3) ...
    Processing triggers for systemd (245.4-4ubuntu3.22) ...
    ubuntu@hap2:~$ 
    ```
  * Останавливаем службу/сервис `HAProxy` в ОС на всех 2х ВМ т.к. сначала ее надо настроить
    ```bash
    sudo systemctl stop haproxy
    ```
  * Включаем автозагрузку службы/сервиса `HAProxy` в ОС на всех 2х ВМ
    ```bash
    sudo systemctl enable haproxy
    ```

***
### Конфигурируем `HAProxy`
  * Создаем файл конфигурации [`/etc/haproxy/haproxy.cfg`](config_files/haproxy.cfg) на всех 2х ВМ
  * ❗️Сохраняем оригинальный файл перед правкой
    ```bash
    sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.orig
    ```
  * ❗️Файл конфигурации одинаковый для всех 2х ВМ   
    ```bash
    sudo vim /etc/haproxy/haproxy.cfg
    ```

  * Запускаем `HAProxy` на 2х ВМ
    ```bash
    sudo systemctl start haproxy
    ```

    
***
### Полезные команды `HAProxy`
  * Посмотреть логи `HAProxy` в ОС
    ```bash
    tail -20f /var/log/haproxy.log
    или
    sudo tail -20f /var/log/syslog
    ```

  * Подключение к `мастеру` `PostgreSQL` через `HAProxy`
    ```bash
    psql -h 10.129.0.10 -p 5002 -U postgres -c "SELECT pg_is_in_recovery()"
    ```
    ```console
    ubuntu@hap1:~$ psql -h 10.129.0.10 -p 5002 -U postgres -c "SELECT pg_is_in_recovery()"
    Password for user postgres: 
     pg_is_in_recovery 
    -------------------
     f
    (1 row)
    
    ubuntu@hap1:~$ 
    ```

  * Подключение к `реплике` `PostgreSQL` через `HAProxy`
    ```bash
    psql -h 10.129.0.10 -p 5003 -U postgres -c "SELECT pg_is_in_recovery()"
    ```
    ```console
    ubuntu@hap2:~$ psql -h 10.129.0.10 -p 5003 -U postgres -c "SELECT pg_is_in_recovery()"
    Password for user postgres: 
     pg_is_in_recovery 
    -------------------
     t
    (1 row)
    
    ubuntu@hap2:~$ 
    ```

  * Доступен веб-интерфейс для `HAProxy`
    ```bash
    http://<ip-server HAProxy>:7000
    ```    

***

### Какие возможности у нас появились на данном этапе
  | :electric_plug: Появилось на этом этапе | :electric_plug: Компонент | :hammer_and_wrench: Название ВМ | :link: ip-адрес | :magnet: порт | :memo: Описание возможности |
  |:--:|:---|:--------|:----------|------:|:------------------------------------------|
  |  | `Consul` | **`pg-srv1`** | `10.129.0.21` | 8500 | Веб-интерфейс (http://10.129.0.21:8500) |
  |  | `Consul` | **`pg-srv2`** | `10.129.0.22` | 8500 | Веб-интерфейс (http://10.129.0.22:8500) |
  |  | `Consul` | **`pg-srv3`** | `10.129.0.23` | 8500 | Веб-интерфейс (http://10.129.0.23:8500) |
  |  | `PostgreSQL` | **`pg-srv1`** | `10.129.0.21` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `PostgreSQL` | **`pg-srv2`** | `10.129.0.22` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `PostgreSQL` | **`pg-srv3`** | `10.129.0.23` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `Patroni` | **`pg-srv1`** | `10.129.0.21` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.21:8008) |
  |  | `Patroni` | **`pg-srv2`** | `10.129.0.22` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.22:8008) |
  |  | `Patroni` | **`pg-srv3`** | `10.129.0.23` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.23:8008) |
  |  | `PgBouncer` | **`pg-srv1`** | `10.129.0.21` | 6432 | Подключение через `PgBouncer` к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.21) |
  |  | `PgBouncer` | **`pg-srv2`** | `10.129.0.22` | 6432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.22) |
  |  | `PgBouncer` | **`pg-srv3`** | `10.129.0.23` | 6432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.23) |
  |  | `KeepAlived` | **`hap1`** | `10.129.0.11` |  | Потенциальная ВМ для поднятия VIP-адреса |
  |  | `KeepAlived` | **`hap2`** | `10.129.0.12` |  | Потенциальная ВМ для поднятия VIP-адреса |
  |  | `KeepAlived-VIP` | **`hap1/hap2`** | `10.129.0.10` |  | VIP-адрес для подключения к `PostgreSQL`. Он может перемещаться между ВМ в зависимости от выполнения условий |
  | :heavy_check_mark: | `HAProxy` | **`hap1`** | `10.129.0.11` | 7000 | Веб-интерфейс (http://10.129.0.11:7000) |
  | :heavy_check_mark: | `HAProxy` | **`hap2`** | `10.129.0.12` | 7000 | Веб-интерфейс (http://10.129.0.12:7000) |
  | :heavy_check_mark: | `HAProxy` | **`hap1/hap2`** | `10.129.0.10` | 5002 | Подключение к `мастеру` `PostgreSQL` через `HAProxy` (psql -h 10.129.0.10 -p 5002 -U postgres) |
  | :heavy_check_mark: | `HAProxy` | **`hap1/hap2`** | `10.129.0.10` | 5003 | Подключение к `реплике` `PostgreSQL` через `HAProxy` (psql -h 10.129.0.10 -p 5003 -U postgres) |


***
### :+1: `HAProxy` установлен и настроен
