<div align="center"><h2> 3. Настройка Patroni </h2></div>

***

### Немного теории:
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
### Список ВМ с которыми идет взаимодействие на данном этапе
  :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
  |--------------:|---------------|
  | **`pg-srv1`** | `10.129.0.21` |
  | **`pg-srv2`** | `10.129.0.22` |      
  | **`pg-srv3`** | `10.129.0.23` |
  
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
  * Создаем конфигурационный файл [`/etc/patroni/patroni.yml`](config_files/patroni.yml)
  * ❗️Разница для каждой ВМ в:
    * `name: pg-srvX`
    * `restapi - connect_address: "10.129.0.X:8008"`
    * `postgresql - connect_address: "10.129.0.X:5432"`
    ```bash
    sudo vim /etc/patroni/patroni.yml
    ```
    ```yml
    name: pg-srv1
    scope: pg-15-cluster
    
    watchdog:
      mode: off
    
    consul:
      host: "localhost:8500"
      register_service: true
      #token: <consul-acl-token>
    
    restapi:
      listen: 0.0.0.0:8008
      connect_address: "10.129.0.21:8008"
      auth: 'patroni:patroni'
    
    bootstrap:
      dcs:
        ttl: 30
        loop_wait: 10
        maximum_lag_on_failover: 1048576
        postgresql:
          use_pg_rewind: true
          use_slots: true
          parameters:
            archive_mode: "on"
            wal_level: hot_standby
            max_wal_senders: 10
            wal_keep_segments: 8
            archive_timeout: 1800s
            max_replication_slots: 5
            hot_standby: "on"
            wal_log_hints: "on"
          pg_hba:
            - local all all trust
            - host replication replicator 10.129.0.21/32 trust
            - host replication replicator 10.129.0.22/32 trust
            - host replication replicator 10.129.0.23/32 trust
            - host replication replicator 127.0.0.1/32 trust
            - host all all 0.0.0.0/0 scram-sha-256
        
    
    initdb:
      - encoding: UTF8
      - data-checksums
    
    postgresql:
      pgpass: /var/lib/postgresql/15/.pgpass
      listen: 0.0.0.0:5432
      connect_address: "10.129.0.21:5432"
      data_dir: /var/lib/postgresql/15/main/
      bin_dir: /usr/lib/postgresql/15/bin/
      pg_rewind:
        username: postgres
        password: password
      replication:
        username: replicator
        password: replicator
      superuser:
        username: postgres
        password: postgres
    ```
  * Параметры конфига
    * `name` — имя узла, на котором настраивается данный конфиг.
    * `scope` — имя кластера. Его мы будем использовать при обращении к ресурсу, а также под этим именем будет зарегистрирован сервис в consul.
    * `consul-token` — если наш кластер consul использует ACL, необходимо указать токен.
    * `restapi-connect_address` — адрес на настраиваемом сервере, на который будут приходить подключения к patroni.
    * `restapi-auth` — логин и пароль для аутентификации на интерфейсе API.
    * `pg_hba` — блок конфигурации pg_hba для разрешения подключения к СУБД и ее базам. Необходимо обратить внимание на подсеть для строки host replication replicator. Она должна соответствовать той, которая используется в вашей инфраструктуре.
    * `postgresql-pgpass` — путь до файла, который создаст патрони. В нем будет храниться пароль для подключения к postgresql.
    * `postgresql-connect_address` — адрес и порт, которые будут использоваться для подключения к СУДБ.
    * `postgresql - data_dir` — путь до файлов с данными базы.
    * `postgresql - bin_dir` — путь до бинарников postgresql.
    * `pg_rewind, replication, superuser` — логины и пароли, которые будут созданы для базы.

***    
###  Создаем настраиваем службу в ОС для `Patroni`
  * Создаем службу [`/usr/lib/systemd/system/patroni.service`](config_files/patroni.service) в ОС на каждой из 3х ВМ
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
    ubuntu@pg-srv1:~$ sudo systemctl enable patroni
    Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /lib/systemd/system/patroni.service.
    ubuntu@pg-srv1:~$    
    ```
    ```console
    ubuntu@pg-srv2:~$ sudo systemctl enable patroni
    Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /lib/systemd/system/patroni.service.
    ubuntu@pg-srv2:~$    
    ```
    ```console
    ubuntu@pg-srv3:~$ sudo systemctl enable patroni
    Created symlink /etc/systemd/system/multi-user.target.wants/patroni.service → /lib/systemd/system/patroni.service.
    ubuntu@pg-srv3:~$    
    ```

  * Смотрим текущее состояние работы сервиса:
    ```bash
    sudo systemctl status patroni
    ```
    ```console
    ubuntu@pg-srv1:~$ sudo systemctl status patroni
    ● patroni.service - Patroni service
         Loaded: loaded (/lib/systemd/system/patroni.service; enabled; vendor preset: enabled)
         Active: active (running) since Fri 2023-08-11 16:08:38 UTC; 2min 37s ago
       Main PID: 7348 (patroni)
          Tasks: 12 (limit: 4631)
         Memory: 106.4M
         CGroup: /system.slice/patroni.service
                 ├─7348 /usr/bin/python3 /usr/local/bin/patroni /etc/patroni/patroni.yml
                 ├─7398 /usr/lib/postgresql/15/bin/postgres -D /var/lib/postgresql/15/main/ --config-file=/var/lib/postgresql/15/main/postgresql.conf --listen_addresses=0.0>
                 ├─7403 postgres: postgres: checkpointer
                 ├─7404 postgres: postgres: background writer
                 ├─7408 postgres: postgres: walwriter
                 ├─7409 postgres: postgres: autovacuum launcher
                 ├─7410 postgres: postgres: archiver
                 ├─7411 postgres: postgres: logical replication launcher
                 └─7413 postgres: postgres: postgres postgres 127.0.0.1(53556) idle
    
    Aug 11 16:09:42 pg-srv1 patroni[7348]: 2023-08-11 16:09:42,887 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:09:52 pg-srv1 patroni[7348]: 2023-08-11 16:09:52,887 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:02 pg-srv1 patroni[7348]: 2023-08-11 16:10:02,887 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:12 pg-srv1 patroni[7348]: 2023-08-11 16:10:12,887 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:22 pg-srv1 patroni[7348]: 2023-08-11 16:10:22,903 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:32 pg-srv1 patroni[7348]: 2023-08-11 16:10:32,919 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:42 pg-srv1 patroni[7348]: 2023-08-11 16:10:42,888 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:10:52 pg-srv1 patroni[7348]: 2023-08-11 16:10:52,888 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:11:02 pg-srv1 patroni[7348]: 2023-08-11 16:11:02,887 INFO: no action. I am (pg-srv1), the leader with the lock
    Aug 11 16:11:12 pg-srv1 patroni[7348]: 2023-08-11 16:11:12,887 INFO: no action. I am (pg-srv1), the leader with the lock
    lines 3-27/27 (END)
    ```
    ```console
    ubuntu@pg-srv2:~$ sudo systemctl status patroni
    ● patroni.service - Patroni service
         Loaded: loaded (/lib/systemd/system/patroni.service; enabled; vendor preset: enabled)
         Active: active (running) since Fri 2023-08-11 16:12:21 UTC; 7s ago
       Main PID: 6714 (patroni)
          Tasks: 6 (limit: 4631)
         Memory: 68.0M
         CGroup: /system.slice/patroni.service
                 ├─6714 /usr/bin/python3 /usr/local/bin/patroni /etc/patroni/patroni.yml
                 └─6740 /usr/lib/postgresql/15/bin/postgres --describe-config
    
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,221 INFO: No PostgreSQL configuration items >
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,226 INFO: Deregister service postgres/pg-srv2
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,241 INFO: Lock owner: pg-srv1; I am pg-srv2
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,242 INFO: Deregister service postgres/pg-srv2
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,257 INFO: trying to bootstrap from leader 'p>
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,263 INFO: Lock owner: pg-srv1; I am pg-srv2
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,266 WARNING: Could not register service: unk>
    Aug 11 16:12:22 pg-srv2 patroni[6714]: 2023-08-11 16:12:22,295 INFO: bootstrap from leader 'pg-srv1' in>
    Aug 11 16:12:29 pg-srv2 patroni[6714]: 2023-08-11 16:12:29,470 INFO: replica has been created using bas>
    Aug 11 16:12:29 pg-srv2 patroni[6714]: 2023-08-11 16:12:29,471 INFO: bootstrapped from leader 'pg-srv1'
    lines 1-20/20 (END)
    ```
    ```console
    ubuntu@pg-srv3:~$ sudo systemctl status patroni
    ● patroni.service - Patroni service
         Loaded: loaded (/lib/systemd/system/patroni.service; enabled; vendor preset: enab>
         Active: active (running) since Fri 2023-08-11 16:13:05 UTC; 5s ago
       Main PID: 6979 (patroni)
          Tasks: 6 (limit: 4631)
         Memory: 69.1M
         CGroup: /system.slice/patroni.service
                 ├─6979 /usr/bin/python3 /usr/local/bin/patroni /etc/patroni/patroni.yml
                 └─7001 /usr/lib/postgresql/15/bin/postgres --describe-config
    
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,142 INFO: No PostgreSQL con>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,154 INFO: Deregister servic>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,164 INFO: Lock owner: pg-sr>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,164 INFO: Deregister servic>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,183 INFO: trying to bootstr>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,185 INFO: Lock owner: pg-sr>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,187 INFO: Deregister servic>
    Aug 11 16:13:06 pg-srv3 patroni[6979]: 2023-08-11 16:13:06,188 INFO: bootstrap from le>
    Aug 11 16:13:10 pg-srv3 patroni[6979]: 2023-08-11 16:13:10,725 INFO: replica has been >
    Aug 11 16:13:10 pg-srv3 patroni[6979]: 2023-08-11 16:13:10,727 INFO: bootstrapped from>
    lines 1-20/20 (END)
    ```
  * Посмотреть список нод
    ```bash
    patronictl -c /etc/patroni/patroni.yml list
    ```
    ```console
    ubuntu@pg-srv1:~$ patronictl -c /etc/patroni/patroni.yml list
    + Cluster: pg-15-cluster ----+---------+-----------+----+-----------+
    | Member  | Host        | Role    | State     | TL | Lag in MB |
    +---------+-------------+---------+-----------+----+-----------+
    | pg-srv1 | 10.129.0.21 | Leader  | running   |  1 |           |
    | pg-srv2 | 10.129.0.22 | Replica | streaming |  1 |         0 |
    | pg-srv3 | 10.129.0.23 | Replica | streaming |  1 |         0 |
    +---------+-------------+---------+-----------+----+-----------+
    ubuntu@pg-srv1:~$ 
    ```
 
***

### Применим настройка `PostgreSQL 15` в конфигурации `Patroni` на основании [`PGTune`](https://pgtune.leopard.in.ua/)
  * Для нашеших ВМ получаем следующее:
    ```console
    # DB Version: 15
    # OS Type: linux
    # DB Type: mixed
    # Total Memory (RAM): 4 GB
    # CPUs num: 2
    # Connections num: 100
    # Data Storage: ssd
    
    max_connections = 100
    shared_buffers = 1GB
    effective_cache_size = 3GB
    maintenance_work_mem = 256MB
    checkpoint_completion_target = 0.9
    wal_buffers = 16MB
    default_statistics_target = 100
    random_page_cost = 1.1
    effective_io_concurrency = 200
    work_mem = 2621kB
    min_wal_size = 1GB
    max_wal_size = 4GB
    ```
  * Редактируем конфигурацию `Patroni`
    ```bash
    patronictl -c /etc/patroni/patroni.yml edit-config
    ```
    ```console
    ubuntu@pg-srv2:~$ patronictl -c /etc/patroni/patroni.yml edit-config
    --- 
    +++ 
    @@ -10,6 +10,18 @@
         wal_keep_segments: 8
         wal_level: hot_standby
         wal_log_hints: 'on'
    +    max_connections: 100
    +    shared_buffers: '1GB'
    +    effective_cache_size: '3GB'
    +    maintenance_work_mem: '256MB'
    +    checkpoint_completion_target: 0.9
    +    wal_buffers: '16MB'
    +    default_statistics_target: 100
    +    random_page_cost: 1.1
    +    effective_io_concurrency: 200
    +    work_mem: '2621kB'
    +    min_wal_size: '1GB'
    +    max_wal_size: '4GB'
       pg_hba:
       - local all all trust
       - host replication replicator 10.129.0.21/32 trust
    
    Apply these changes? [y/N]: y
    Configuration changed
    ubuntu@pg-srv2:~$ 
    ```
 * Видим, что для примеенния определенных переменный нужен рестарт кластера `Pending restart`
    ```console
    ubuntu@pg-srv1:~$ patronictl -c /etc/patroni/patroni.yml list
    + Cluster: pg-15-cluster ---------+-----------+----+-----------+-----------------+
    | Member  | Host        | Role    | State     | TL | Lag in MB | Pending restart |
    +---------+-------------+---------+-----------+----+-----------+-----------------+
    | pg-srv1 | 10.129.0.21 | Replica | streaming |  4 |         0 | *               |
    | pg-srv2 | 10.129.0.22 | Replica | streaming |  4 |         0 | *               |
    | pg-srv3 | 10.129.0.23 | Leader  | running   |  4 |           | *               |
    +---------+-------------+---------+-----------+----+-----------+-----------------+
    ubuntu@pg-srv1:~$ 
    ```
  * Для примемения параметров - перезагружаем кластер
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart pg-15-cluster
    ```
    ```console
    ubuntu@pg-srv1:~$ patronictl -c /etc/patroni/patroni.yml restart pg-15-cluster
    + Cluster: pg-15-cluster ---------+-----------+----+-----------+-----------------+
    | Member  | Host        | Role    | State     | TL | Lag in MB | Pending restart |
    +---------+-------------+---------+-----------+----+-----------+-----------------+
    | pg-srv1 | 10.129.0.21 | Replica | streaming |  4 |         0 | *               |
    | pg-srv2 | 10.129.0.22 | Replica | streaming |  4 |         0 | *               |
    | pg-srv3 | 10.129.0.23 | Leader  | running   |  4 |           | *               |
    +---------+-------------+---------+-----------+----+-----------+-----------------+
    When should the restart take place (e.g. 2023-08-11T18:29)  [now]: 
    Are you sure you want to restart members pg-srv1, pg-srv2, pg-srv3? [y/N]: y
    Restart if the PostgreSQL version is less than provided (e.g. 9.5.2)  []: 
    Success: restart on member pg-srv1
    Success: restart on member pg-srv2
    Success: restart on member pg-srv3
    ubuntu@pg-srv1:~$ 
    ```
  * Смотрим, что параметры применились
    ```bash
    patronictl -c /etc/patroni/patroni.yml show-config
    ```
    ```console
    ubuntu@pg-srv1:~$ patronictl -c /etc/patroni/patroni.yml show-config
    loop_wait: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      parameters:
        archive_mode: 'on'
        archive_timeout: 1800s
        checkpoint_completion_target: 0.9
        default_statistics_target: 100
        effective_cache_size: 3GB
        effective_io_concurrency: 200
        hot_standby: 'on'
        maintenance_work_mem: 256MB
        max_connections: 100
        max_replication_slots: 5
        max_wal_senders: 10
        max_wal_size: 4GB
        min_wal_size: 1GB
        random_page_cost: 1.1
        shared_buffers: 1GB
        wal_buffers: 16MB
        wal_keep_segments: 8
        wal_level: hot_standby
        wal_log_hints: 'on'
        work_mem: 2621kB
      pg_hba:
      - local all all trust
      - host replication replicator 10.129.0.21/32 trust
      - host replication replicator 10.129.0.22/32 trust
      - host replication replicator 10.129.0.23/32 trust
      - host replication replicator 127.0.0.1/32 trust
      - host all all 0.0.0.0/0 scram-sha-256
      use_pg_rewind: true
      use_slots: true
    ttl: 30
    
    ubuntu@pg-srv1:~$ 
    ```
    ```console
    ubuntu@pg-srv1:~$ patronictl -c /etc/patroni/patroni.yml list
    + Cluster: pg-15-cluster ---------+-----------+----+-----------+
    | Member  | Host        | Role    | State     | TL | Lag in MB |
    +---------+-------------+---------+-----------+----+-----------+
    | pg-srv1 | 10.129.0.21 | Replica | streaming |  4 |         0 |
    | pg-srv2 | 10.129.0.22 | Replica | streaming |  4 |         0 |
    | pg-srv3 | 10.129.0.23 | Leader  | running   |  4 |           |
    +---------+-------------+---------+-----------+----+-----------+
    ubuntu@pg-srv1:~$ 
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
    patronictl -c /etc/patroni/patroni.yml restart pg-15-cluster pg-srv1
    ```

  * Рестарт всего кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart pg-15-cluster
    ```

  * Рестарт reload кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml reload pg-15-cluster
    ```

  * Плановое переключение
    ```bash
    patronictl -c /etc/patroni/patroni.yml switchover pg-15-cluster
    ```

  * Реинициализации ноды
    ```bash
    patronictl -c /etc/patroni/patroni.yml reinit pg-15-cluster pg-srv2
    ```

***

### :+1: Patroni установлен и настроен
