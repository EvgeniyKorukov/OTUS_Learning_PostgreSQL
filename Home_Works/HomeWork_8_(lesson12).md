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
 * Берем калькулятор параметров [PGTune](https://pgtune.leopard.in.ua/) и считаем параметры для следующей конфигурации:
   * DB version: `15`
   * OS Type: `Linux`
   * DB Type: `Mixed type of application`
   * Total Memory (RAM): `4 GB`
   * Number of CPUs: `2`
   * Number of Connections: `20` 
   * Data Storage: `SSD Storage`
     ```sql
     # DB Version: 15
     # OS Type: linux
     # DB Type: mixed
     # Total Memory (RAM): 4 GB
     # CPUs num: 2
     # Connections num: 20
     # Data Storage: ssd

     max_connections = 20
     shared_buffers = 1GB
     effective_cache_size = 3GB
     maintenance_work_mem = 256MB
     checkpoint_completion_target = 0.9
     wal_buffers = 16MB
     default_statistics_target = 100
     random_page_cost = 1.1
     effective_io_concurrency = 200
     work_mem = 13107kB
     min_wal_size = 1GB
     max_wal_size = 4GB
     ```
 * Изменим еще ряд парамеров для ускорения СУБД, в ущерб надежности:
   * Переведем режим журналирования в минимальный режим т.к. реплик у нас нет, а минимального уровня хватит для восстановаления. 
     * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal)=`minimal`
     * Параметр wal_level определяет, как много информации записывается в WAL. Со значением replica (по умолчанию) в журнал записываются данные, необходимые для поддержки архивирования WAL и репликации, включая запросы только на чтение на ведомом сервере. Вариант minimal оставляет только информацию, необходимую для восстановления после сбоя или аварийного отключения.
   * Количество `max_wal_senders` надо сделать равным `0` т.к. мы изменили wal_level, и если этого не сделать, то кластер PostgreSQL не запуститься
     * [max_wal_senders](https://postgrespro.ru/docs/postgrespro/15/runtime-config-replication#GUC-MAX-WAL-SENDERS)=`0`
   * Переведем кластер PostgreSQL в асинхронный режим, тем самым мы не будем дожидаться оповещения о результате операции. 
     * [synchronous_commit](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-SYNCHRONOUS-COMMIT)=`off`
     * Определяет, после завершения какого уровня обработки WAL сервер будет сообщать об успешном выполнении операции. Допустимые значения: remote_apply (применено удалённо), on (вкл., по умолчанию), remote_write (записано удалённо), local (локально) и off (выкл.).
   * Отключим режим синхронизации(подтверждения физической записи на СХД) СУБД с ОС по дисковому вводу/выводу. Есть риск потерять данные в случает отключения питания. 
     * [fsync](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-FSYNC)=`off`
     * Если этот параметр установлен, сервер Postgres Pro старается добиться, чтобы изменения были записаны на диск физически, выполняя системные вызовы fsync() или другими подобными методами (см. wal_sync_method). Это даёт гарантию, что кластер баз данных сможет вернуться в согласованное состояние после сбоя оборудования или операционной системы.
     * ❗️ Хотя отключение fsync часто даёт выигрыш в скорости, это может привести к неисправимой порче данных в случае отключения питания или сбоя системы. Поэтому отключать fsync рекомендуется, только если вы легко сможете восстановить всю базу из внешнего источника.
   * Отключим запись всего содержимого каждой страницы в wal при checkpoint
     * [full_page_writes](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-FULL-PAGE-WRITES)=`off`
     * Когда этот параметр включён, сервер Postgres Pro записывает в WAL всё содержимое каждой страницы при первом изменении этой страницы после контрольной точки. Это необходимо, потому что запись страницы, прерванная при сбое операционной системы, может выполниться частично, и на диске окажется страница, содержащая смесь старых данных с новыми. При этом информации об изменениях на уровне строк, которая обычно сохраняется в WAL, будет недостаточно для получения согласованного содержимого такой страницы при восстановлении после сбоя. Сохранение образа всей страницы гарантирует, что страницу можно восстановить корректно, ценой увеличения объёма данных, которые будут записываться в WAL. (Так как воспроизведение WAL всегда начинается от контрольной точки, достаточно сделать это при первом изменении каждой страницы после контрольной точки. Таким образом, уменьшить затраты на запись полных страниц можно, увеличив интервалы контрольных точек.)
     * ❗️ Отключение этого параметра ускоряет обычные операции, но может привести к неисправимому повреждению или незаметной порче данных после сбоя системы. Так как при этом возникают практически те же риски, что и при отключении fsync, хотя и в меньшей степени, отключать его следует только при тех же обстоятельствах, которые перечислялись в рекомендациях для вышеописанного параметра.


 * Применяем параметры на СУБД PostgreSQL и перезапускаем кластер, чтобы применились параметры:
    ```sql
    eugin@pg-srv:~$ sudo -u postgres psql
    psql (15.3 (Ubuntu 15.3-1.pgdg20.04+1))
    Type "help" for help.

    postgres=# ALTER SYSTEM SET max_connections = '20';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET shared_buffers = '1GB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET effective_cache_size = '3GB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET maintenance_work_mem = '256MB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET checkpoint_completion_target = '0.9';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET wal_buffers = '16MB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET default_statistics_target = '100';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET random_page_cost = '1.1';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET effective_io_concurrency = '200';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET work_mem = '13107kB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET min_wal_size = '1GB';
    ALTER SYSTEM
    postgres=# ALTER SYSTEM SET max_wal_size = '4GB';
    ALTER SYSTEM
    postgres=#  
    postgres=# alter system set wal_level='minimal';
    ALTER SYSTEM
    postgres=# alter system set max_wal_senders='0';
    ALTER SYSTEM
    postgres=# alter system set synchronous_commit='off';
    ALTER SYSTEM
    postgres=# alter system set fsync='off';
    ALTER SYSTEM
    postgres=# alter system set full_page_writes='off'; 
    ALTER SYSTEM
    postgres=# 
    postgres=# \q
    eugin@pg-srv:~$ 
    eugin@pg-srv:~$ sudo pg_ctlcluster 15 main restart
    eugin@pg-srv:~$ 
    eugin@pg-srv:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
    eugin@pg-srv:~$ 
    ```
*** 

> ### нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
* Проводим измерение нагрузки через pgbench. Параметры запуска pgbench такие:**
    * 20 клиентов `--client=20`
    * Новое подключение для каждой транзакции `--connect`
    * 5 параллельных потоков `--jobs=5`
    * Выводим прогрес каждые 30 секунд `--progress=30`
    * Время на тест 300 секунд или 5 минут `--time=300`
    * База: bench_tests
    ```console  
    eugin@pg-srv:~$ sudo -u postgres pgbench --client=20 --connect --jobs=5 --progress=30 --time=300 bench_tests
    pgbench (15.3 (Ubuntu 15.3-1.pgdg20.04+1))
    starting vacuum...end.
    pgbench: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  sorry, too many clients already
    pgbench: error: client 2 aborted while establishing connection
    progress: 30.0 s, 322.0 tps, lat 52.931 ms stddev 28.149, 0 failed
    progress: 60.0 s, 323.6 tps, lat 52.094 ms stddev 27.503, 0 failed
    progress: 90.0 s, 328.2 tps, lat 51.326 ms stddev 26.695, 0 failed
    progress: 120.0 s, 323.7 tps, lat 52.067 ms stddev 27.247, 0 failed
    progress: 150.0 s, 326.8 tps, lat 51.639 ms stddev 27.110, 0 failed
    progress: 180.0 s, 324.6 tps, lat 52.092 ms stddev 26.365, 0 failed
    progress: 210.0 s, 324.8 tps, lat 51.968 ms stddev 26.565, 0 failed
    progress: 240.0 s, 325.0 tps, lat 51.929 ms stddev 26.235, 0 failed
    progress: 270.0 s, 321.7 tps, lat 52.542 ms stddev 26.743, 0 failed
    progress: 300.0 s, 325.2 tps, lat 51.913 ms stddev 26.672, 0 failed
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 20
    number of threads: 5
    maximum number of tries: 1
    duration: 300 s
    number of transactions actually processed: 97383
    number of failed transactions: 0 (0.000%)
    latency average = 52.047 ms
    latency stddev = 26.935 ms
    average connection time = 6.546 ms
    tps = 324.580605 (including reconnection times)
    pgbench: error: Run was aborted; the above results are incomplete.
    eugin@pg-srv:~$ 
    ```
*** 

> ### написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
* Выводы:
  * `number of transactions actually processed` было `63410` стало `97383`. Количество транзакций увеличилось в `1,54` раза и это хороший результат.
  * `latency average` было `91.079 ms` стало `52.047 ms`. Средняя задержка уменьшилась в `1,75` раза и это хороший результат.
  * `latency stddev` было `84.750 ms` стало `26.935 ms`. Средняя задержка дисковых устройств уменьшилась в `3,15` раза и это очень хороший результат.
  * `average connection time` было `3.554 ms` стало `6.546 ms`. Увеличилось среднее время подключения в `1,84` раза, что не есть хорошо. Но это вызывано тем, что pgbench упирался в количество подключений, которое мы уменьшили.
  * `tps` было `211.293792` стало `324.580605`. В `1,54` раза увеличилось количество транзакций в секунду, что есть очень хорошо.
* Результат: 
  * Производительность стала намного выше, но в ущерб надежности.
  * Второй прогон pgbench выполнился не полностью т.к. уперся в количество подключений `max_connections`, которое мы уменьшили с `100` до `20` в процессе настройки. 
 

*** 

> ### Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки https://github.com/akopytov/sysbench)
  * In Process  


*** 
