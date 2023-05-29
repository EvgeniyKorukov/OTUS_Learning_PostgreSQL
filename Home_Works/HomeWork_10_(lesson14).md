<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Виды и устройство репликации в PostgreSQL. Практика применени" </h2></div>

***

> ### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
  * Создаем ВМ 1 в YandexCloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv1`** | Гарантированная доля vCPU | `20%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.21` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `51.250.19.127` |
    | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `10 ГБ` |
    | RAM | `4 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

    ```console
    eugink@nb-xiaomi ~ $ yc compute instance create \
      --name pg-srv1 \
      --hostname pg-srv1 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=10G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.21 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
       ```
  * Подключаемся к ВМ 1 
    ```console
    eugink@nb-xiaomi ~ $ ssh ubuntu@51.250.19.127
    ubuntu@pg-srv1:~$
    ```
  * Устанавливаем PostgreSQL 15 (логи убрал т.к. полезного там мало)
    ```console
    ubuntu@pg-srv1:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    ubuntu@pg-srv1:~$ 
    ubuntu@pg-srv1:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log    
    ```
  * Правим параметры и рестартуем postgres, чтобы параметры применились:
    * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-WAL-LEVEL) = `logical`
    * [listen_addresses](https://postgrespro.ru/docs/enterprise/15/runtime-config-connection#GUC-LISTEN-ADDRESSES)='*'
      ```console
      ubuntu@pg-srv1:~$ sudo -u postgres psql -c "alter user postgres with password 'Pass1234';"
      ALTER ROLE
      ubuntu@pg-srv1:~$
      ubuntu@pg-srv1:~$ sudo -u postgres psql -c 'alter system set wal_level=logical;'
      ALTER SYSTEM
      ubuntu@pg-srv1:~$ sudo -u postgres psql -c "alter system set listen_addresses = '*';"
      ALTER SYSTEM
      ubuntu@pg-srv1:~$ sudo pg_ctlcluster 15 main restart
      ubuntu@pg-srv1:~$ 
      ubuntu@pg-srv1:~$ sudo -u postgres psql -c 'show wal_level'
       wal_level 
      -----------
       logical
      (1 row)
      
      ubuntu@pg-srv1:~$ 
      ubuntu@pg-srv1:~$ sudo -u postgres psql -c 'show listen_addresses'
       listen_addresses 
      ------------------
       *
      (1 row)

      ubuntu@pg-srv1:~$ 
      ```
  * Правим `pg_hba` и делаем `reload`, чтобы применились параметры:
    * Добавляем:
      * host    all             all             10.129.0.0/24           trust
      ```console
      ubuntu@pg-srv1:~$ sudo vim /etc/postgresql/15/main/pg_hba.conf 
      ubuntu@pg-srv1:~$ sudo pg_ctlcluster 15 main reload
      ```
  * Создаем таблицы и выдаем права на них для пользователя `otus_replica`
   * ❗️ Доступ будет предоставлен по ролевой модели для пользователя `otus_replica` и входить надо будет под этим пользователем
     ```pgsql
     postgres=# create user otus_replica with password 'Pass1234';
     CREATE ROLE
     postgres=# create table test (id int, txt text);
     CREATE TABLE
     postgres=# create table test2 (id2 int, txt2 text);
     CREATE TABLE
     postgres=# grant select on table test2 to otus_replica;
     GRANT
     postgres=# grant all on table test to otus_replica;
     GRANT
     ```
   * Проверяем работу прав доступа на ВМ 1
     ```console
     ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test values (11, 'test_11');"
     Password for user otus_replica: 
     INSERT 0 1
     ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test2 values (55, 'test_55');"
     Password for user otus_replica: 
     ERROR:  permission denied for table test2
     ubuntu@pg-srv1:~$ 
     ```     
***

> ### 2. Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
  * Создаем публикацию таблицы test
    ```pgsql
    postgres=# CREATE PUBLICATION test_pub FOR TABLE test;
    CREATE PUBLICATION
    postgres=# \dRp+
                                Publication test_pub
      Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root 
    ----------+------------+---------+---------+---------+-----------+----------
     postgres | f          | t       | t       | t       | t         | f
    Tables:
        "public.test"

    postgres=# 
    ```
  * Подписываемся на публикацию таблицы test2 с ВМ №2
  * ❗️ Это будет возможным после настройки PostgreSQL на ВМ №2. Но мы считаем, что уже выполнили пункты 3 и 4 и вернулись сюда 
    ```pgsql
    postgres=# CREATE SUBSCRIPTION test2_sub 
    postgres-# CONNECTION 'host=10.129.0.22 port=5432 user=postgres password=Pass1234 dbname=postgres' 
    postgres-# PUBLICATION test2_pub WITH (copy_data = true);
    NOTICE:  created replication slot "test2_sub" on publisher
    CREATE SUBSCRIPTION
    postgres=# 
    ```
    
***
> ### 3. На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.
  * Создаем ВМ 2 в YandexCloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv2`** | Гарантированная доля vCPU | `20%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.22` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `51.250.100.25` |
    | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `10 ГБ` |
    | RAM | `4 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

    ```console
    eugink@nb-xiaomi ~ $ yc compute instance create \
      --name pg-srv2 \
      --hostname pg-srv2 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=10G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.22 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
    ```
   
  * Подключаемся к ВМ 2
    ```console
    eugink@nb-xiaomi ~ $ ssh ubuntu@51.250.100.25
    ubuntu@pg-srv2:~$
    ```
  * Устанавливаем PostgreSQL 15 (логи убрал т.к. полезного там мало)
    ```console
    ubuntu@pg-srv2:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    ubuntu@pg-srv2:~$ 
    ubuntu@pg-srv2:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log    
    ```
  * Правим параметры и рестартуем postgres, чтобы параметры применились:
    * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-WAL-LEVEL) = `logical`
    * [listen_addresses](https://postgrespro.ru/docs/enterprise/15/runtime-config-connection#GUC-LISTEN-ADDRESSES) = '*'
      ```console
      ubuntu@pg-srv2:~$ sudo -u postgres psql -c "alter user postgres with password 'Pass1234';"
      ALTER ROLE
      ubuntu@pg-srv2:~$      
      ubuntu@pg-srv2:~$ sudo -u postgres psql -c 'alter system set wal_level=logical;'
      ALTER SYSTEM
      ubuntu@pg-srv2:~$ sudo -u postgres psql -c "alter system set listen_addresses = '*';"
      ALTER SYSTEM
      ubuntu@pg-srv2:~$ sudo pg_ctlcluster 15 main restart
      ubuntu@pg-srv2:~$ 
      ubuntu@pg-srv2:~$ sudo -u postgres psql -c 'show wal_level'
       wal_level 
      -----------
       logical
      (1 row)
      
      ubuntu@pg-srv2:~$ 
      ubuntu@pg-srv2:~$ sudo -u postgres psql -c 'show listen_addresses'
       listen_addresses 
      ------------------
       *
      (1 row)

      ubuntu@pg-srv2:~$ 
      ```
  * Правим `pg_hba` и делаем `reload`, чтобы применились параметры:
    * Добавляем:
      * host    all             all             10.129.0.0/24           trust
      ```console
      ubuntu@pg-srv2:~$ sudo vim /etc/postgresql/15/main/pg_hba.conf 
      ubuntu@pg-srv2:~$ sudo pg_ctlcluster 15 main reload
      ```
  * Создаем таблицы и выдаем права на них для пользователя `otus_replica`
   * ❗️ Доступ будет предоставлен по ролевой модели для пользователя `otus_replica` и входить надо будет под этим пользователем
     ```pgsql
     postgres=# create user otus_replica with password 'Pass1234';
     CREATE ROLE
     postgres=# create table test (id int, txt text);
     CREATE TABLE
     postgres=# create table test2 (id2 int, txt2 text);
     CREATE TABLE
     postgres=# grant select on table test to otus_replica;
     GRANT
     postgres=# grant all on table test2 to otus_replica;
     GRANT
     ```  
   * Проверяем работу прав доступа на ВМ 2
     ```console
     ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test values (22, 'test_22');"
     Password for user otus_replica: 
     ERROR:  permission denied for table test
     ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test2 values (44, 'test_44');"
     Password for user otus_replica: 
     INSERT 0 1
     ubuntu@pg-srv2:~$ 
     ```       
***

> ### 4. Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
  * Создаем публикацию таблицы test
    ```pgsql
    postgres=# CREATE PUBLICATION test2_pub FOR TABLE test2;
    CREATE PUBLICATION
    postgres=# 
    postgres=# \dRp+
                               Publication test2_pub
      Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root 
    ----------+------------+---------+---------+---------+-----------+----------
     postgres | f          | t       | t       | t       | t         | f
    Tables:
        "public.test2"

    postgres=# 
    ```
  * Подписываемся на публикацию таблицы test1 с ВМ №1
    ```pgsql
    postgres=# CREATE SUBSCRIPTION test_pub 
    postgres-# CONNECTION 'host=10.129.0.21 port=5432 user=postgres password=Pass1234 dbname=postgres' 
    postgres-# PUBLICATION test_pub WITH (copy_data = true);
    NOTICE:  created replication slot "test_pub" on publisher
    CREATE SUBSCRIPTION
    postgres=# 
    ```
   
***
> ### ❗️ 2.1 Проверка работы логической реликации на ВМ 1
  * ВМ 1 до теста
    ```console
    ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "select * from  test;"
    Password for user otus_replica: 
     id |   txt   
    ----+---------
     11 | test_11
    (1 row)

    ubuntu@pg-srv1:~$ 
    ```
  * ВМ 2 до теста
    ```console
    ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "select * from  test;"
    Password for user otus_replica: 
     id |   txt   
    ----+---------
     11 | test_11
    (1 row)

    ubuntu@pg-srv2:~$ 
    ``` 
  * ВМ 1 после теста
    ```console
    ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test values (22, 'test_22');"
    Password for user otus_replica: 
    INSERT 0 1
    ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "select * from  test;"
    Password for user otus_replica: 
     id |   txt   
    ----+---------
     11 | test_11
     22 | test_22
    (2 rows)

    ubuntu@pg-srv1:~$ 
    ```
  * ВМ 2 после теста
    ```console
    ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "select * from  test;"
    Password for user otus_replica: 
     id |   txt   
    ----+---------
     11 | test_11
     22 | test_22
    (2 rows)

    ubuntu@pg-srv2:~$ 
    ```
  * :+1: **Вывод: Логическая репликация работает на таблице test между ВМ1 и ВМ2**  

***
> ### ❗️ 4.1 Проверка работы логической реликации на ВМ 2
  * ВМ 2 до теста
    ```console
    ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c 'select * from test2;'
    Password for user otus_replica: 
     id2 |  txt2   
    -----+---------
      44 | test_44
    (1 row)

    ubuntu@pg-srv2:~$ 
    ```
  * ВМ 1 до теста
    ```console
    ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c 'select * from test2;'
    Password for user otus_replica: 
     id2 |  txt2   
    -----+---------
      44 | test_44
    (1 row)

    ubuntu@pg-srv1:~$ 
    ``` 
  * ВМ 2 после теста
    ```console
    ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test2 values (88, 'test_88');"
    Password for user otus_replica: 
    INSERT 0 1
    ubuntu@pg-srv2:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c 'select * from test2;'
    Password for user otus_replica: 
     id2 |  txt2   
    -----+---------
      44 | test_44
      88 | test_88
    (2 rows)

    ubuntu@pg-srv2:~$ 
    ```
  * ВМ 1 после теста
    ```console
    ubuntu@pg-srv1:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c 'select * from test2;'
    Password for user otus_replica: 
     id2 |  txt2   
    -----+---------
      44 | test_44
      88 | test_88
    (2 rows)

    ubuntu@pg-srv1:~$ 
    ```
  * :+1: **Вывод: Логическая репликация работает на таблице test2 между ВМ2 и ВМ1**  

***


> ### 5. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
  * Создаем ВМ 3 в YandexCloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv3`** | Гарантированная доля vCPU | `20%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.23` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `62.84.121.169` |
    | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `10 ГБ` |
    | RAM | `4 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

    ```console
    eugink@nb-xiaomi ~ $ yc compute instance create \
      --name pg-srv3 \
      --hostname pg-srv3 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=10G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.23 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
    ```
   
  * Подключаемся к ВМ 2
    ```console
    eugink@nb-xiaomi ~ $ ssh ubuntu@62.84.121.169
    ubuntu@pg-srv3:~$
    ```
  * Устанавливаем PostgreSQL 15 (логи убрал т.к. полезного там мало)
    ```console
    ubuntu@pg-srv3:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    ubuntu@pg-srv3:~$ 
    ubuntu@pg-srv3:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log    
    ```
  * Правим параметры и рестартуем postgres, чтобы параметры применились:
    * [listen_addresses](https://postgrespro.ru/docs/enterprise/15/runtime-config-connection#GUC-LISTEN-ADDRESSES)='*'
      ```console
      ubuntu@pg-srv3:~$ sudo -u postgres psql -c "alter user postgres with password 'Pass1234';"
      ALTER ROLE
      ubuntu@pg-srv3:~$      
      ubuntu@pg-srv3:~$ sudo -u postgres psql -c "alter system set listen_addresses = '*';"
      ALTER SYSTEM
      ubuntu@pg-srv3:~$ sudo pg_ctlcluster 15 main restart
      ubuntu@pg-srv3:~$ 
      ubuntu@pg-srv3:~$ sudo -u postgres psql -c 'show listen_addresses'
       listen_addresses 
      ------------------
       *
      (1 row)

      ubuntu@pg-srv3:~$ 
      ```
  * Правим `pg_hba` и делаем `reload`, чтобы применились параметры:
    * Добавляем:
      * host    all             all             10.129.0.0/24           trust
      ```console
      ubuntu@pg-srv3:~$ sudo vim /etc/postgresql/15/main/pg_hba.conf 
      ubuntu@pg-srv3:~$ sudo pg_ctlcluster 15 main reload
      ```
  * Создаем таблицы и выдаем права на них для пользователя `otus_replica`
   * ❗️ Доступ будет предоставлен по ролевой модели для пользователя `otus_replica` и входить надо будет под этим пользователем
     ```pgsql
     postgres=# create user otus_replica with password 'Pass1234';
     CREATE ROLE
     postgres=# create table test (id int, txt text);
     CREATE TABLE
     postgres=# create table test2 (id2 int, txt2 text);
     CREATE TABLE
     postgres=# grant select on table test to otus_replica;
     GRANT
     postgres=# grant select on table test2 to otus_replica;
     GRANT
     ```  
   * Проверяем работу прав доступа на ВМ 3
     ```console
     ubuntu@pg-srv3:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test values (33, 'test_33');"
     Password for user otus_replica: 
     ERROR:  permission denied for table test
     ubuntu@pg-srv3:~$ sudo -u postgres psql -d postgres -U otus_replica -h localhost -c "insert into test2 values (77, 'test_77');"
     Password for user otus_replica: 
     ERROR:  permission denied for table test2
     ubuntu@pg-srv3:~$ 
     ```       
 
 * Подписываемся на таблицы из ВМ №1 и №2 и проверяем, что все изменения есть
     ```sql
     postgres=# CREATE SUBSCRIPTION test2_3_sub 
     postgres-# CONNECTION 'host=10.129.0.22 port=5432 user=postgres password=Pass1234 dbname=postgres' 
     postgres-# PUBLICATION test2_pub WITH (copy_data = true);
     NOTICE:  created replication slot "test2_3_sub" on publisher
     CREATE SUBSCRIPTION
     postgres=# 
     postgres=# CREATE SUBSCRIPTION test_3_pub x
     postgres-# CONNECTION 'host=10.129.0.21 port=5432 user=postgres password=Pass1234 dbname=postgres' 
     postgres-# PUBLICATION test_pub WITH (copy_data = true);
     NOTICE:  created replication slot "test_3_pub" on publisher
     CREATE SUBSCRIPTION
     postgres=# 
     postgres=# select * from test;
      id |   txt   
     ----+---------
      11 | test_11
      22 | test_22
     (2 rows)

     postgres=# select * from test2;
      id2 |  txt2   
     -----+---------
       44 | test_44
       88 | test_88
     ```
***

> ### 6. Задачка под звездочкой: реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.
  * Для решения этой части надо потоковую репликацию 
    * Начнем с настройки параметров PostgreSQL на ВМ №3 и перезагрузки для применения изменений
      * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-WAL-LEVEL) = `replica`
      * [max_wal_senders](https://postgrespro.ru/docs/postgrespro/15/runtime-config-replication#GUC-MAX-WAL-SENDERS) = `10`
      ```console
      ubuntu@pg-srv3:~$ sudo -u postgres psql -c 'alter system set wal_level=replica'
      ALTER SYSTEM
      ubuntu@pg-srv3:~$ 
      ubuntu@pg-srv3:~$ sudo -u postgres psql -c 'alter system set max_wal_senders=10'
      ALTER SYSTEM
      ubuntu@pg-srv3:~$ 
      ubuntu@pg-srv3:~$ sudo pg_ctlcluster 15 main restart
      ubuntu@pg-srv3:~$ 
      ```
  * Создаем ВМ 4 в YandexCloud
     :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
     --------------:|---------------|--------------:|---------------|
     | Название ВМ | **`pg-srv4`** | Гарантированная доля vCPU | `20%` | 
     | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.24` | 
     | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `130.193.41.201` |
     | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
     | vCPU | `2` | Объём дискового пространства | `10 ГБ` |
     | RAM | `4 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
     | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

     ```console
     eugink@nb-xiaomi ~ $ yc compute instance create \
       --name pg-srv4 \
       --hostname pg-srv4 \
       --cores 2 \
       --memory 4 \
       --create-boot-disk size=10G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
       --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.24 \
       --zone ru-central1-b \
       --core-fraction 20 \
       --preemptible \
       --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
     ```
   
     * Подключаемся к ВМ №4
     ```console
     eugink@nb-xiaomi ~ $ ssh ubuntu@130.193.41.201
     ubuntu@pg-srv4:~$
     ```
     * Устанавливаем PostgreSQL 15 (логи убрал т.к. полезного там мало)
     ```console
     ubuntu@pg-srv4:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
     ubuntu@pg-srv4:~$ 
     ubuntu@pg-srv4:~$ pg_lsclusters 
     Ver Cluster Port Status Owner    Data directory              Log file
     15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log    
     ```
         
***

