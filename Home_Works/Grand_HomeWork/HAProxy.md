  <div align="center"><h2> 5. Настройка HAProxy </h2></div>

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
    sudo apt -y install haproxyr
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
    ubuntu@hap1:~$ sudo systemctl stop haproxy
    ubuntu@hap1:~$ sudo systemctl enable haproxy
    Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
    Executing: /lib/systemd/systemd-sysv-install enable haproxy
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
    ubuntu@hap2:~$ sudo systemctl stop haproxy
    ubuntu@hap2:~$ sudo systemctl enable haproxy
    Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
    Executing: /lib/systemd/systemd-sysv-install enable haproxy
    ubuntu@hap2:~$ sudo systemctl status haproxy
    ● haproxy.service - HAProxy Load Balancer
         Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
         Active: inactive (dead) since Sun 2023-08-13 22:43:07 UTC; 1min 40s ago
           Docs: man:haproxy(1)
                 file:/usr/share/doc/haproxy/configuration.txt.gz
       Main PID: 2454 (code=exited, status=0/SUCCESS)
    
    Aug 13 22:42:10 hap2 systemd[1]: Starting HAProxy Load Balancer...
    Aug 13 22:42:10 hap2 haproxy[2454]: [NOTICE] 224/224210 (2454) : New worker #1 (2469) forked
    Aug 13 22:42:10 hap2 systemd[1]: Started HAProxy Load Balancer.
    Aug 13 22:43:07 hap2 systemd[1]: Stopping HAProxy Load Balancer...
    Aug 13 22:43:07 hap2 haproxy[2454]: [WARNING] 224/224307 (2454) : Exiting Master process...
    Aug 13 22:43:07 hap2 haproxy[2454]: [ALERT] 224/224307 (2454) : Current worker #1 (2469) exited with c>
    Aug 13 22:43:07 hap2 haproxy[2454]: [WARNING] 224/224307 (2454) : All workers exited. Exiting... (0)
    Aug 13 22:43:07 hap2 systemd[1]: haproxy.service: Succeeded.
    Aug 13 22:43:07 hap2 systemd[1]: Stopped HAProxy Load Balancer.
    
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
  | :heavy_check_mark: | `HAProxy` | **`hap1`** | `10.129.0.11` | 0 |  |
  | :heavy_check_mark: | `HAProxy` | **`hap2`** | `10.129.0.12` | 0 |  |


***
### :+1: `HAProxy` установлен и настроен
