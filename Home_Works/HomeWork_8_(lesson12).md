<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Настройка PostgreSQL" </h2></div>


***

> ### развернуть виртуальную машину любым удобным способом
  * Развернул виртуальную машину в Yandex Cloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv`** | Гарантированная доля vCPU | `100%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.11` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `84.201.152.215` |
    | Платформа | `Intel Ice Lake` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `5 ГБ` |
    | RAM | `4 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

***

> ### поставить на неё PostgreSQL 15 любым способом
  * Устанавливаем PostgreSQL 15
    ```console
    eugin@pg-srv:~$ sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-15
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease                                              
    Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease                                            
    Hit:4 http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease                                        
    Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease                                          
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    All packages are up to date.
    N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease' doesn't support architecture 'i386'
    Reading package lists...
    Building dependency tree...
    Reading state information...
    Calculating upgrade...
    The following packages were automatically installed and are no longer required:
      libcommon-sense-perl libidn11 libjson-perl libjson-xs-perl libllvm10 libsensors-config libsensors5 libtypes-serialiser-perl libxslt1.1 lynx-common postgresql-common ssl-cert sysstat
    Use 'sudo apt autoremove' to remove them.
    Get more security updates through Ubuntu Pro with 'esm-apps' enabled:
      lynx-common
    Learn more about Ubuntu Pro at https://ubuntu.com/pro
    0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    OK
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease                                              
    Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease                                            
    Hit:4 http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease                                        
    Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease                                          
    Reading package lists... Done
    N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease' doesn't support architecture 'i386'
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
      libidn11 lynx-common
    Use 'sudo apt autoremove' to remove them.
    The following NEW packages will be installed:
      postgresql-15
    0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
    Need to get 16.3 MB of archives.
    After this operation, 53.5 MB of additional disk space will be used.
    Get:1 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-15 amd64 15.3-1.pgdg20.04+1 [16.3 MB]
    Fetched 16.3 MB in 1s (22.5 MB/s)         
    Preconfiguring packages ...
    Selecting previously unselected package postgresql-15.
    (Reading database ... 105326 files and directories currently installed.)
    Preparing to unpack .../postgresql-15_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking postgresql-15 (15.3-1.pgdg20.04+1) ...
    Setting up postgresql-15 (15.3-1.pgdg20.04+1) ...
    update-alternatives: updating alternative /usr/share/postgresql/15/man/man1/psql.1.gz because link group psql.1.gz has changed slave links
    Processing triggers for postgresql-common (249.pgdg20.04+1) ...
    Building PostgreSQL dictionaries from installed myspell/hunspell packages...
    Removing obsolete dictionary files:
    eugin@pg-srv:~$ 
    eugin@pg-srv:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
    eugin@pg-srv:~$ 
    ```
  * ❗️ **Проводим измерение нагрузки через pgbench, со значениями по умолчанию для PostgreSQL 15. Параметры запуска pgbench такие:**
    * 20 клиентов `--client=20`
    * Новое подключение для каждой транзакции `--connect`
    * 5 параллельных потоков `--jobs=5`
    * Выводим прогрес каждые 30 секунд `--progress=30`
    * Время на тест 300 секунд или 5 минут `--time=300`
    * База: bench_tests
    ```console
    eugin@pg-srv:~$ sudo -u postgres psql
    psql (15.3 (Ubuntu 15.3-1.pgdg20.04+1))
    Type "help" for help.

    postgres=# \l
                                                     List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
    -----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
    (3 rows)

    postgres=# create database bench_tests;
    CREATE DATABASE
    postgres=# \q
    eugin@pg-srv:~$ 
    eugin@pg-srv:~$ sudo -u postgres pgbench -i bench_tests
    dropping old tables...
    NOTICE:  table "pgbench_accounts" does not exist, skipping
    NOTICE:  table "pgbench_branches" does not exist, skipping
    NOTICE:  table "pgbench_history" does not exist, skipping
    NOTICE:  table "pgbench_tellers" does not exist, skipping
    creating tables...
    generating data (client-side)...
    100000 of 100000 tuples (100%) done (elapsed 1.24 s, remaining 0.00 s)
    vacuuming...
    creating primary keys...
    done in 2.07 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 1.75 s, vacuum 0.04 s, primary keys 0.27 s).
    eugin@pg-srv:~$ 
    eugin@pg-srv:~$ sudo -u postgres pgbench --client=20 --connect --jobs=5 --progress=30 --time=300 bench_tests
    pgbench (15.3 (Ubuntu 15.3-1.pgdg20.04+1))
    starting vacuum...end.
    progress: 30.0 s, 207.2 tps, lat 92.675 ms stddev 90.687, 0 failed
    progress: 60.0 s, 225.3 tps, lat 85.105 ms stddev 83.888, 0 failed
    progress: 90.0 s, 216.7 tps, lat 88.646 ms stddev 73.884, 0 failed
    progress: 120.0 s, 206.0 tps, lat 93.796 ms stddev 78.239, 0 failed
    progress: 150.0 s, 218.2 tps, lat 87.970 ms stddev 67.603, 0 failed
    progress: 180.0 s, 231.6 tps, lat 82.635 ms stddev 60.447, 0 failed
    progress: 210.0 s, 183.1 tps, lat 105.927 ms stddev 126.609, 0 failed
    progress: 240.0 s, 222.9 tps, lat 85.975 ms stddev 81.786, 0 failed
    progress: 270.0 s, 184.7 tps, lat 104.719 ms stddev 95.683, 0 failed
    progress: 300.0 s, 217.1 tps, lat 88.755 ms stddev 79.458, 0 failed
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 20
    number of threads: 5
    maximum number of tries: 1
    duration: 300 s
    number of transactions actually processed: 63410
    number of failed transactions: 0 (0.000%)
    latency average = 91.079 ms
    latency stddev = 84.750 ms
    average connection time = 3.554 ms
    tps = 211.293792 (including reconnection times)
    eugin@pg-srv:~$ 
    ```
*** 

> ### настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
  * In Process


*** 

> ### нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
  * In Process
  

*** 

> ### написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
  * In Process  
 

*** 

> ### Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки https://github.com/akopytov/sysbench)
  * In Process  


*** 
