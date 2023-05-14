<div align="center"><h2> Отчет о выполнении домашнего задания по теме: Журналы</h2></div>


***

> ### 1. Настройте выполнение контрольной точки раз в 30 секунд
* Поскольку `context=sighup`, то чтобы применить параметр, будет достаточно `SELECT pg_reload_conf()`
  ```sql
  postgres=# select name, setting, unit, context from pg_settings where name='checkpoint_timeout';
          name        | setting | unit | context 
  --------------------+---------+------+---------
   checkpoint_timeout | 300     | s    | sighup
  (1 row)

  postgres=# alter system set checkpoint_timeout = 30;
  ALTER SYSTEM
  postgres=# 
  postgres=# select pg_reload_conf();
   pg_reload_conf 
  ----------------
   t
  (1 row)

  postgres=# 
  postgres=# select name, setting, unit, context from pg_settings where name='checkpoint_timeout';
          name        | setting | unit | context 
  --------------------+---------+------+---------
   checkpoint_timeout | 30      | s    | sighup
  (1 row)

  postgres=# 
  ```

***

> ### 2. 10 минут c помощью утилиты pgbench подавайте нагрузку.
* Для выполнения пункта(задания) 3, нам надо выполнить запрос, сохранив его данные, до и после тестов с pgbench.
* Инициализацию pgbench, для БД `test_db1` провел заранее, командой `pgbench -i test_db1`
  ```console
  ubuntu@srv-postgres:~$ sudo -u postgres psql -d test_db1
  psql (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
  Type "help" for help.

  test_db1=# SELECT pg_current_wal_lsn(), pg_current_wal_insert_lsn(), pg_walfile_name(pg_current_wal_lsn()) file_current_wal_lsn, pg_walfile_name(pg_current_wal_insert_lsn()) file_current_wal_insert_lsn;              
   pg_current_wal_lsn | pg_current_wal_insert_lsn |   file_current_wal_lsn   | file_current_wal_insert_lsn 
  --------------------+---------------------------+--------------------------+-----------------------------
   1/65410AC8         | 1/65410AC8                | 000000010000000100000065 | 000000010000000100000065
  (1 row)

  test_db1=# 
  test_db1=# \q
  ubuntu@srv-postgres:~$
  ubuntu@srv-postgres:~$ sudo -u postgres pgbench -P 30 -T 600 -c 10 test_db1
  pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
  starting vacuum...end.
  progress: 30.0 s, 253.1 tps, lat 39.323 ms stddev 18.350, 0 failed
  progress: 60.0 s, 257.3 tps, lat 38.854 ms stddev 18.449, 0 failed
  progress: 90.0 s, 258.1 tps, lat 38.742 ms stddev 17.978, 0 failed
  progress: 120.0 s, 258.3 tps, lat 38.716 ms stddev 18.426, 0 failed
  progress: 150.0 s, 262.2 tps, lat 38.132 ms stddev 18.431, 0 failed
  progress: 180.0 s, 264.2 tps, lat 37.855 ms stddev 18.073, 0 failed
  progress: 210.0 s, 258.1 tps, lat 38.734 ms stddev 18.581, 0 failed
  progress: 240.0 s, 258.7 tps, lat 38.644 ms stddev 18.574, 0 failed
  progress: 270.0 s, 262.4 tps, lat 38.111 ms stddev 18.592, 0 failed
  progress: 300.0 s, 261.0 tps, lat 38.317 ms stddev 18.481, 0 failed
  progress: 330.0 s, 256.5 tps, lat 38.973 ms stddev 18.245, 0 failed
  progress: 360.0 s, 262.9 tps, lat 38.030 ms stddev 17.721, 0 failed
  progress: 390.0 s, 260.8 tps, lat 38.335 ms stddev 18.061, 0 failed
  progress: 420.0 s, 263.1 tps, lat 38.009 ms stddev 17.724, 0 failed
  progress: 450.0 s, 262.9 tps, lat 38.029 ms stddev 18.349, 0 failed
  progress: 480.0 s, 260.1 tps, lat 38.444 ms stddev 18.071, 0 failed
  progress: 510.0 s, 253.1 tps, lat 39.508 ms stddev 18.379, 0 failed
  progress: 540.0 s, 256.7 tps, lat 38.953 ms stddev 18.822, 0 failed
  progress: 570.0 s, 258.9 tps, lat 38.633 ms stddev 17.867, 0 failed
  progress: 600.0 s, 259.1 tps, lat 38.581 ms stddev 18.103, 0 failed
  transaction type: <builtin: TPC-B (sort of)>
  scaling factor: 1
  query mode: simple
  number of clients: 10
  number of threads: 1
  maximum number of tries: 1
  duration: 600 s
  number of transactions actually processed: 155633
  number of failed transactions: 0 (0.000%)
  latency average = 38.541 ms
  latency stddev = 18.270 ms
  initial connection time = 114.318 ms
  tps = 259.419597 (without initial connection time)

  ubuntu@srv-postgres:~$ 
  ubuntu@srv-postgres:~$ sudo -u postgres psql -d test_db1 -c 'SELECT pg_current_wal_lsn(), pg_current_wal_insert_lsn(), pg_walfile_name(pg_current_wal_lsn()) file_current_wal_lsn, pg_walfile_name(pg_current_wal_insert_lsn()) file_current_wal_insert_lsn'

   pg_current_wal_lsn | pg_current_wal_insert_lsn |   file_current_wal_lsn   | file_current_wal_insert_lsn 
  --------------------+---------------------------+--------------------------+-----------------------------
   1/7AFB2868         | 1/7AFB2868                | 00000001000000010000007A | 00000001000000010000007A
  (1 row)

  ubuntu@srv-postgres:~$ 
  ```

***

> ### 3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
* Объем журнальных файлов был сгенерирован за это время. Данные lsn взяли из пункта(задания) 2
  * Вариант 1 из курса
    ```sql
    test_db1=# SELECT '1/7AFB2868'::pg_lsn - '1/65410AC8'::pg_lsn wal_size;
     wal_size  
    -----------
     364518816
    (1 row)
    ```
  * Вариант 2 из курса, обвернутый в функцию `pg_size_pretty` с человеческим видом размера
    ```sql
    test_db1=# SELECT pg_size_pretty('1/7AFB2868'::pg_lsn - '1/65410AC8'::pg_lsn) wal_size;
     wal_size  
    -----------
     348 MB
    (1 row)
    ```
  * Вариант 3 через функцию `pg_wal_lsn_diff`
    ```sql
    test_db1=# select pg_wal_lsn_diff('1/7AFB2868'::pg_lsn,'1/65410AC8'::pg_lsn) wal_size;
     wal_size  
    -----------
     364518816
    (1 row)
    ```  
  * Вариант 4 через функцию `pg_wal_lsn_diff`, обвернутый в функцию `pg_size_pretty` с человеческим видом размера
    ```sql
    test_db1=# select pg_size_pretty(pg_wal_lsn_diff('1/7AFB2868'::pg_lsn,'1/65410AC8'::pg_lsn)) wal_size;
     wal_size  
    -----------
     348 MB
    (1 row)
    ```
    
* Какой объем приходится в среднем на одну контрольную точку
  * Объем сгенерированных wal `364 518 816 Кб` или `348 MB`
  * Тест с pgbench выполнялся `10 минут` т.е. `600 секунд`
  * Контрольная точка выполняется `раз в 30 секунд` `checkpoint_timeout=30`
  * Получается: `364 518 816 Кб/600*30`=`18 225 940,8 Кб` или `348 MB/600*30`=`17,4 MB`

***
> ### 4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
* Здесь вставлю цитату из статьи [WAL в PostgreSQL: 3. Контрольная точка](https://habr.com/ru/companies/postgrespro/articles/460423/)
  * Таким образом, большая часть контрольных точек происходит по расписанию: раз в checkpoint_timeout единиц времени. Но при повышенной нагрузке контрольная точка вызывается чаще, при достижении объема max_wal_size.

***
> ### 5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
  * Параметры синхронного режима из лекции:
    * synchronous_commit = on
    * commit_delay = 0
    * commit_siblings = 5
  * Проводим тестирование pgbench на синхронном режиме работы
    ```sql
    ubuntu@srv-postgres:~$ sudo -u postgres pgbench -P 1 -T 10 test_db1
    pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
    starting vacuum...end.
    progress: 1.0 s, 152.0 tps, lat 6.460 ms stddev 0.911, 0 failed
    progress: 2.0 s, 149.0 tps, lat 6.741 ms stddev 0.577, 0 failed
    progress: 3.0 s, 154.0 tps, lat 6.495 ms stddev 0.389, 0 failed
    progress: 4.0 s, 167.0 tps, lat 5.977 ms stddev 0.163, 0 failed
    progress: 5.0 s, 154.0 tps, lat 6.489 ms stddev 0.268, 0 failed
    progress: 6.0 s, 159.0 tps, lat 6.299 ms stddev 0.483, 0 failed
    progress: 7.0 s, 160.0 tps, lat 6.222 ms stddev 0.395, 0 failed
    progress: 8.0 s, 172.0 tps, lat 5.829 ms stddev 0.288, 0 failed
    progress: 9.0 s, 172.0 tps, lat 5.819 ms stddev 0.270, 0 failed
    progress: 10.0 s, 163.0 tps, lat 6.111 ms stddev 0.646, 0 failed
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 1
    number of threads: 1
    maximum number of tries: 1
    duration: 10 s
    number of transactions actually processed: 1603
    number of failed transactions: 0 (0.000%)
    latency average = 6.230 ms
    latency stddev = 0.564 ms
    initial connection time = 12.167 ms
    tps = 160.469327 (without initial connection time)
    ubuntu@srv-postgres:~$ 
    ```
  * Параметры асинхронного режима из лекции:
    * synchronous_commit = off
    * wal_writer_delay = 200ms

  * Переводим Postgres в асинхронный режим и проводим тестирование с помощью pgbench
    ```sql
    test_db1=# ALTER SYSTEM SET synchronous_commit = off;
    ALTER SYSTEM
    test_db1=# SELECT pg_reload_conf();
     pg_reload_conf 
    ----------------
     t
    (1 row)

    test_db1=# show synchronous_commit;
     synchronous_commit 
    --------------------
     off
    (1 row)

    test_db1=# \q
    ubuntu@srv-postgres:~$ 
    ubuntu@srv-postgres:~$ sudo -u postgres pgbench -P 1 -T 10 test_db1
    [sudo] password for ubuntu: 
    pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
    starting vacuum...end.
    progress: 1.0 s, 264.0 tps, lat 3.740 ms stddev 0.506, 0 failed
    progress: 2.0 s, 271.0 tps, lat 3.688 ms stddev 0.235, 0 failed
    progress: 3.0 s, 273.0 tps, lat 3.655 ms stddev 0.264, 0 failed
    progress: 4.0 s, 276.0 tps, lat 3.619 ms stddev 0.180, 0 failed
    progress: 5.0 s, 276.0 tps, lat 3.629 ms stddev 0.222, 0 failed
    progress: 6.0 s, 275.0 tps, lat 3.630 ms stddev 0.273, 0 failed
    progress: 7.0 s, 277.0 tps, lat 3.609 ms stddev 0.500, 0 failed
    progress: 8.0 s, 283.0 tps, lat 3.532 ms stddev 0.185, 0 failed
    progress: 9.0 s, 284.9 tps, lat 3.513 ms stddev 0.197, 0 failed
    progress: 10.0 s, 281.0 tps, lat 3.557 ms stddev 0.201, 0 failed
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 1
    number of threads: 1
    maximum number of tries: 1
    duration: 10 s
    number of transactions actually processed: 2762
    number of failed transactions: 0 (0.000%)
    latency average = 3.616 ms
    latency stddev = 0.306 ms
    initial connection time = 12.050 ms
    tps = 276.437653 (without initial connection time)
    ubuntu@srv-postgres:~$ 
    ```
  * Результат:
    * Увеличились `tps` c `tps = 160.469327` до `tps = 276.437653`. Значительно увеличилось количество транзакций в секунду.
    * Уменьшилась `latency average` с `latency average = 6.230 ms` до `latency average = 3.616 ms`. Уменьшилось среднее время задержки.
    * Уменьшилась `latency stddev` с `latency stddev = 0.564 ms` до `latency stddev = 0.306 ms`. Уменьшилось время задержки с дисковым устройством.
    * Да, производительность сервера увеличилась, но упала надежность и отказоустойчивость. Тут надо выбирать в зависимости от задачи, или надежнось или производителность, в ущерб надежности.
    
***

> ### 6. Создайте новый кластер с включенной контрольной суммой страниц. 
  ```sql
    ubuntu@srv-postgres:~$ sudo pg_createcluster 15 main2 --start -- --data-checksums
    Creating new PostgreSQL cluster 15/main2 ...
    /usr/lib/postgresql/15/bin/initdb -D /var/lib/postgresql/15/main2 --auth-local peer --auth-host scram-sha-256 --no-instructions --data-checksums
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are enabled.

    fixing permissions on existing directory /var/lib/postgresql/15/main2 ... ok
    creating subdirectories ... ok
    selecting dynamic shared memory implementation ... posix
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting default time zone ... Etc/UTC
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... ok
    syncing data to disk ... ok
    Ver Cluster Port Status Owner    Data directory               Log file
    15  main2   5433 online postgres /var/lib/postgresql/15/main2 /var/log/postgresql/postgresql-15-main2.log
    ubuntu@srv-postgres:~$ 
    ubuntu@srv-postgres:~$ sudo -u postgres psql -p 5433 -c 'show data_checksums'
    could not change directory to "/home/ubuntu": Permission denied
     data_checksums 
    ----------------
     on
    (1 row)

    ubuntu@srv-postgres:~$ 
  ```
> Создайте таблицу. 
> Вставьте несколько значений. 
> Выключите кластер. 
> Измените пару байт в таблице. 
> Включите кластер и сделайте выборку из таблицы. 
  ```sql
  ubuntu@srv-postgres:~$ sudo -u postgres psql -p 5433 
  could not change directory to "/home/ubuntu": Permission denied
  psql (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
  Type "help" for help.

  postgres=# 
  postgres=# CREATE TABLE test_text(t text);
  INSERT INTO test_text SELECT 'строка '||s.id FROM generate_series(1,500) AS s(id); 
  CREATE TABLE
  INSERT 0 500
  postgres=# 
  postgres=# SELECT pg_relation_filepath('test_text');
   pg_relation_filepath 
  ----------------------
   base/5/16387
  (1 row)

  postgres=# 
  postgres=# \q
  ubuntu@srv-postgres:~$ sudo -u postgres pg_ctlcluster 15 main2 stop 
  ubuntu@srv-postgres:~$ sudo -u postgres pg_lsclusters 
  Ver Cluster Port Status Owner    Data directory               Log file
  15  main    5432 online postgres /var/lib/postgresql/15/main  /var/log/postgresql/postgresql-15-main.log
  15  main2   5433 down   postgres /var/lib/postgresql/15/main2 /var/log/postgresql/postgresql-15-main2.log
  ubuntu@srv-postgres:~$ 
  ubuntu@srv-postgres:~$  sudo dd if=/dev/zero of=/var/lib/postgresql/15/main2/base/5/16387 oflag=dsync conv=notrunc bs=1 count=8
  8+0 records in
  8+0 records out
  8 bytes copied, 0.0214502 s, 0.4 kB/s
  ubuntu@srv-postgres:~$ 
  ubuntu@srv-postgres:~$ sudo -u postgres pg_ctlcluster 15 main2 start
  ubuntu@srv-postgres:~$ 
  ubuntu@srv-postgres:~$ sudo -u postgres psql -p 5433 -c 'select * from test_text'
  could not change directory to "/home/ubuntu": Permission denied
  WARNING:  page verification failed, calculated checksum 63015 but expected 49533
  ERROR:  invalid page in block 0 of relation base/5/16387
  ubuntu@srv-postgres:~$ 
  
  ```
> Что и почему произошло? 
* При обращение к объекту обнаружилось нарушение в контрольной сумме. Или, другими словами, обнаружились поврежденные данные. 

> как проигнорировать ошибку и продолжить работу?
  * Можно использовать параметр `ignore_checksum_failure = on` для того чтобы считать, не поврежденные, данные из таблицы.

***

