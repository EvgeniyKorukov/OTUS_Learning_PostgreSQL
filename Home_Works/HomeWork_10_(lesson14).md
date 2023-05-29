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
    * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-WAL-LEVEL)=logical
    * [listen_addresses](https://postgrespro.ru/docs/enterprise/15/runtime-config-connection#GUC-LISTEN-ADDRESSES)='*'
      ```console
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
    * [wal_level](https://postgrespro.ru/docs/postgrespro/15/runtime-config-wal#GUC-WAL-LEVEL)=logical
    * [listen_addresses](https://postgrespro.ru/docs/enterprise/15/runtime-config-connection#GUC-LISTEN-ADDRESSES)='*'
      ```console
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

> ### 5. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
  * Создаем ВМ 3
  * Подключаемся к ВМ 3
  * Устанавливаем PostgreSQL 15 
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
    done (29s)
    id: epde1o12moeeisiqjif2
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-05-28T20:34:47Z"
    name: pg-srv3
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "4294967296"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epdrn9uvad7m7ib0poep
      auto_delete: true
      disk_id: epdrn9uvad7m7ib0poep
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:e0:e0:22:b6
        subnet_id: e2lamcqbnklme222m12r
        primary_v4_address:
          address: 10.129.0.23
          one_to_one_nat:
            address: 62.84.121.169
            ip_version: IPV4
    gpu_settings: {}
    fqdn: pg-srv3.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}

    eugink@nb-xiaomi ~ $ 
    eugink@nb-xiaomi ~ $ ssh ubuntu@62.84.121.169
    Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-148-generic x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.

    /usr/bin/xauth:  file /home/ubuntu/.Xauthority does not exist
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

    ubuntu@pg-srv3:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease [114 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease [108 kB]                
    Get:4 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]    
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main i386 Packages [825 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 Packages [2,571 kB]
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main Translation-en [434 kB]
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 c-n-f Metadata [16.6 kB]
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/restricted amd64 Packages [1,879 kB]
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/restricted Translation-en [264 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal-updates/universe i386 Packages [728 kB]
    Get:12 http://mirror.yandex.ru/ubuntu focal-updates/universe amd64 Packages [1,063 kB]
    Get:13 http://mirror.yandex.ru/ubuntu focal-updates/universe Translation-en [253 kB]
    Get:14 http://mirror.yandex.ru/ubuntu focal-updates/universe amd64 c-n-f Metadata [24.3 kB]
    Get:15 http://mirror.yandex.ru/ubuntu focal-backports/main amd64 Packages [45.7 kB]
    Get:16 http://mirror.yandex.ru/ubuntu focal-backports/main i386 Packages [36.1 kB]
    Get:17 http://mirror.yandex.ru/ubuntu focal-backports/main amd64 c-n-f Metadata [1,420 B]
    Get:18 http://mirror.yandex.ru/ubuntu focal-backports/universe i386 Packages [13.8 kB]
    Get:19 http://mirror.yandex.ru/ubuntu focal-backports/universe amd64 Packages [25.0 kB]
    Get:20 http://mirror.yandex.ru/ubuntu focal-backports/universe amd64 c-n-f Metadata [880 B]
    Get:21 http://security.ubuntu.com/ubuntu focal-security/main i386 Packages [597 kB]
    Get:22 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [2,192 kB]
    Get:23 http://security.ubuntu.com/ubuntu focal-security/main Translation-en [354 kB]
    Get:24 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [13.0 kB]
    Get:25 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [1,770 kB]
    Get:26 http://security.ubuntu.com/ubuntu focal-security/restricted Translation-en [249 kB]
    Get:27 http://security.ubuntu.com/ubuntu focal-security/universe i386 Packages [598 kB]
    Get:28 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [839 kB]
    Get:29 http://security.ubuntu.com/ubuntu focal-security/universe Translation-en [172 kB]
    Get:30 http://security.ubuntu.com/ubuntu focal-security/universe amd64 c-n-f Metadata [17.8 kB]
    Fetched 15.3 MB in 4s (3,814 kB/s)                                 
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    11 packages can be upgraded. Run 'apt list --upgradable' to see them.
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    Calculating upgrade... Done
    The following NEW packages will be installed:
      linux-headers-5.4.0-149 linux-headers-5.4.0-149-generic linux-image-5.4.0-149-generic linux-modules-5.4.0-149-generic linux-modules-extra-5.4.0-149-generic
    The following packages will be upgraded:
      ca-certificates cloud-init libncurses6 libncursesw6 libtinfo6 linux-generic linux-headers-generic linux-image-generic ncurses-base ncurses-bin ncurses-term
    11 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
    11 standard LTS security updates
    Need to get 78.6 MB of archives.
    After this operation, 380 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 ncurses-bin amd64 6.2-0ubuntu2.1 [172 kB]
    Get:2 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 ncurses-base all 6.2-0ubuntu2.1 [18.9 kB]
    Get:3 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libncursesw6 amd64 6.2-0ubuntu2.1 [132 kB]
    Get:4 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libncurses6 amd64 6.2-0ubuntu2.1 [101 kB]
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libtinfo6 amd64 6.2-0ubuntu2.1 [87.4 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 ca-certificates all 20230311ubuntu0.20.04.1 [152 kB]
    Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-modules-5.4.0-149-generic amd64 5.4.0-149.166 [15.0 MB]
    Get:8 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-image-5.4.0-149-generic amd64 5.4.0-149.166 [10.5 MB]
    Get:9 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-modules-extra-5.4.0-149-generic amd64 5.4.0-149.166 [39.2 MB]
    Get:10 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-generic amd64 5.4.0.149.147 [1,896 B]                                                                                                                     
    Get:11 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-image-generic amd64 5.4.0.149.147 [2,492 B]                                                                                                               
    Get:12 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-5.4.0-149 all 5.4.0-149.166 [11.0 MB]                                                                                                             
    Get:13 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-5.4.0-149-generic amd64 5.4.0-149.166 [1,365 kB]                                                                                                  
    Get:14 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 linux-headers-generic amd64 5.4.0.149.147 [2,360 B]                                                                                                             
    Get:15 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 ncurses-term all 6.2-0ubuntu2.1 [249 kB]                                                                                                                        
    Get:16 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 cloud-init all 23.1.2-0ubuntu0~20.04.2 [532 kB]                                                                                                                 
    Fetched 78.6 MB in 13s (6,198 kB/s)                                                                                                                                                                                            
    Preconfiguring packages ...
    (Reading database ... 102583 files and directories currently installed.)
    Preparing to unpack .../ncurses-bin_6.2-0ubuntu2.1_amd64.deb ...
    Unpacking ncurses-bin (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Setting up ncurses-bin (6.2-0ubuntu2.1) ...
    (Reading database ... 102583 files and directories currently installed.)
    Preparing to unpack .../ncurses-base_6.2-0ubuntu2.1_all.deb ...
    Unpacking ncurses-base (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Setting up ncurses-base (6.2-0ubuntu2.1) ...
    (Reading database ... 102583 files and directories currently installed.)
    Preparing to unpack .../libncursesw6_6.2-0ubuntu2.1_amd64.deb ...
    Unpacking libncursesw6:amd64 (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Preparing to unpack .../libncurses6_6.2-0ubuntu2.1_amd64.deb ...
    Unpacking libncurses6:amd64 (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Preparing to unpack .../libtinfo6_6.2-0ubuntu2.1_amd64.deb ...
    Unpacking libtinfo6:amd64 (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Setting up libtinfo6:amd64 (6.2-0ubuntu2.1) ...
    (Reading database ... 102583 files and directories currently installed.)
    Preparing to unpack .../00-ca-certificates_20230311ubuntu0.20.04.1_all.deb ...
    Unpacking ca-certificates (20230311ubuntu0.20.04.1) over (20211016ubuntu0.20.04.1) ...
    Selecting previously unselected package linux-modules-5.4.0-149-generic.
    Preparing to unpack .../01-linux-modules-5.4.0-149-generic_5.4.0-149.166_amd64.deb ...
    Unpacking linux-modules-5.4.0-149-generic (5.4.0-149.166) ...
    Selecting previously unselected package linux-image-5.4.0-149-generic.
    Preparing to unpack .../02-linux-image-5.4.0-149-generic_5.4.0-149.166_amd64.deb ...
    Unpacking linux-image-5.4.0-149-generic (5.4.0-149.166) ...
    Selecting previously unselected package linux-modules-extra-5.4.0-149-generic.
    Preparing to unpack .../03-linux-modules-extra-5.4.0-149-generic_5.4.0-149.166_amd64.deb ...
    Unpacking linux-modules-extra-5.4.0-149-generic (5.4.0-149.166) ...
    Preparing to unpack .../04-linux-generic_5.4.0.149.147_amd64.deb ...
    Unpacking linux-generic (5.4.0.149.147) over (5.4.0.148.146) ...
    Preparing to unpack .../05-linux-image-generic_5.4.0.149.147_amd64.deb ...
    Unpacking linux-image-generic (5.4.0.149.147) over (5.4.0.148.146) ...
    Selecting previously unselected package linux-headers-5.4.0-149.
    Preparing to unpack .../06-linux-headers-5.4.0-149_5.4.0-149.166_all.deb ...
    Unpacking linux-headers-5.4.0-149 (5.4.0-149.166) ...
    Selecting previously unselected package linux-headers-5.4.0-149-generic.
    Preparing to unpack .../07-linux-headers-5.4.0-149-generic_5.4.0-149.166_amd64.deb ...
    Unpacking linux-headers-5.4.0-149-generic (5.4.0-149.166) ...
    Preparing to unpack .../08-linux-headers-generic_5.4.0.149.147_amd64.deb ...
    Unpacking linux-headers-generic (5.4.0.149.147) over (5.4.0.148.146) ...
    Preparing to unpack .../09-ncurses-term_6.2-0ubuntu2.1_all.deb ...
    Unpacking ncurses-term (6.2-0ubuntu2.1) over (6.2-0ubuntu2) ...
    Preparing to unpack .../10-cloud-init_23.1.2-0ubuntu0~20.04.2_all.deb ...
    Unpacking cloud-init (23.1.2-0ubuntu0~20.04.2) over (23.1.2-0ubuntu0~20.04.1) ...
    Setting up cloud-init (23.1.2-0ubuntu0~20.04.2) ...
    Setting up ca-certificates (20230311ubuntu0.20.04.1) ...
    Updating certificates in /etc/ssl/certs...
    rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
    19 added, 6 removed; done.
    Setting up libncurses6:amd64 (6.2-0ubuntu2.1) ...
    Setting up linux-modules-5.4.0-149-generic (5.4.0-149.166) ...
    Setting up libncursesw6:amd64 (6.2-0ubuntu2.1) ...
    Setting up linux-headers-5.4.0-149 (5.4.0-149.166) ...
    Setting up ncurses-term (6.2-0ubuntu2.1) ...
    Setting up linux-image-5.4.0-149-generic (5.4.0-149.166) ...
    I: /boot/vmlinuz.old is now a symlink to vmlinuz-5.4.0-148-generic
    I: /boot/initrd.img.old is now a symlink to initrd.img-5.4.0-148-generic
    I: /boot/vmlinuz is now a symlink to vmlinuz-5.4.0-149-generic
    I: /boot/initrd.img is now a symlink to initrd.img-5.4.0-149-generic
    Setting up linux-modules-extra-5.4.0-149-generic (5.4.0-149.166) ...
    Setting up linux-headers-5.4.0-149-generic (5.4.0-149.166) ...
    Setting up linux-image-generic (5.4.0.149.147) ...
    Setting up linux-headers-generic (5.4.0.149.147) ...
    Setting up linux-generic (5.4.0.149.147) ...
    Processing triggers for rsyslog (8.2001.0-1ubuntu1.3) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    Processing triggers for ca-certificates (20230311ubuntu0.20.04.1) ...
    Updating certificates in /etc/ssl/certs...
    0 added, 0 removed; done.
    Running hooks in /etc/ca-certificates/update.d...
    done.
    Processing triggers for linux-image-5.4.0-149-generic (5.4.0-149.166) ...
    /etc/kernel/postinst.d/initramfs-tools:
    update-initramfs: Generating /boot/initrd.img-5.4.0-149-generic
    /etc/kernel/postinst.d/zz-update-grub:
    Sourcing file `/etc/default/grub'
    Sourcing file `/etc/default/grub.d/init-select.cfg'
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-5.4.0-149-generic
    Found initrd image: /boot/initrd.img-5.4.0-149-generic
    Found linux image: /boot/vmlinuz-5.4.0-148-generic
    Found initrd image: /boot/initrd.img-5.4.0-148-generic
    Found linux image: /boot/vmlinuz-5.4.0-42-generic
    Found initrd image: /boot/initrd.img-5.4.0-42-generic
    done
    OK
    Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
    Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease                                             
    Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease                                           
    Get:4 http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease [117 kB]  
    Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease             
    Get:6 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 Packages [264 kB]
    Fetched 380 kB in 2s (251 kB/s)   
    Reading package lists... Done
    N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease' doesn't support architecture 'i386'
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
      linux-headers-5.4.0-42 linux-headers-5.4.0-42-generic linux-image-5.4.0-42-generic linux-modules-5.4.0-42-generic linux-modules-extra-5.4.0-42-generic
    Use 'sudo apt autoremove' to remove them.
    The following additional packages will be installed:
      libcommon-sense-perl libgdbm-compat4 libjson-perl libjson-xs-perl libllvm10 libperl5.30 libpq5 libsensors-config libsensors5 libtypes-serialiser-perl libxslt1.1 perl perl-modules-5.30 postgresql-client-15
      postgresql-client-common postgresql-common ssl-cert sysstat
    Suggested packages:
      lm-sensors perl-doc libterm-readline-gnu-perl | libterm-readline-perl-perl make libb-debug-perl liblocale-codes-perl postgresql-doc-15 openssl-blacklist isag
    The following NEW packages will be installed:
      libcommon-sense-perl libgdbm-compat4 libjson-perl libjson-xs-perl libllvm10 libperl5.30 libpq5 libsensors-config libsensors5 libtypes-serialiser-perl libxslt1.1 perl perl-modules-5.30 postgresql-15 postgresql-client-15
      postgresql-client-common postgresql-common ssl-cert sysstat
    0 upgraded, 19 newly installed, 0 to remove and 0 not upgraded.
    Need to get 41.6 MB of archives.
    After this operation, 186 MB of additional disk space will be used.
    Get:1 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl-modules-5.30 all 5.30.0-9ubuntu0.3 [2,739 kB]
    Get:2 http://mirror.yandex.ru/ubuntu focal/main amd64 libgdbm-compat4 amd64 1.18.1-5 [6,244 B]
    Get:3 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libperl5.30 amd64 5.30.0-9ubuntu0.3 [3,951 kB]
    Get:4 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-client-common all 250.pgdg20.04+1 [93.3 kB]
    Get:5 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 perl amd64 5.30.0-9ubuntu0.3 [224 kB]
    Get:6 http://mirror.yandex.ru/ubuntu focal/main amd64 libjson-perl all 4.02000-2 [80.9 kB]
    Get:7 http://mirror.yandex.ru/ubuntu focal/main amd64 ssl-cert all 1.0.39 [17.0 kB]
    Get:8 http://mirror.yandex.ru/ubuntu focal/main amd64 libcommon-sense-perl amd64 3.74-2build6 [20.1 kB]
    Get:9 http://mirror.yandex.ru/ubuntu focal/main amd64 libtypes-serialiser-perl all 1.0-1 [12.1 kB]
    Get:10 http://mirror.yandex.ru/ubuntu focal/main amd64 libjson-xs-perl amd64 4.020-1build1 [83.7 kB]
    Get:11 http://mirror.yandex.ru/ubuntu focal/main amd64 libllvm10 amd64 1:10.0.0-4ubuntu1 [15.3 MB]
    Get:12 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-common all 250.pgdg20.04+1 [239 kB]
    Get:13 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 libpq5 amd64 15.3-1.pgdg20.04+1 [184 kB]
    Get:14 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-client-15 amd64 15.3-1.pgdg20.04+1 [1,680 kB]
    Get:15 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 postgresql-15 amd64 15.3-1.pgdg20.04+1 [16.3 MB]
    Get:16 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors-config all 1:3.6.0-2ubuntu1.1 [6,052 B]
    Get:17 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libsensors5 amd64 1:3.6.0-2ubuntu1.1 [27.2 kB]
    Get:18 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 libxslt1.1 amd64 1.1.34-4ubuntu0.20.04.1 [151 kB]
    Get:19 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 sysstat amd64 12.2.0-2ubuntu0.2 [448 kB]
    Fetched 41.6 MB in 2s (17.9 MB/s)     
    Preconfiguring packages ...
    Selecting previously unselected package perl-modules-5.30.
    (Reading database ... 139026 files and directories currently installed.)
    Preparing to unpack .../00-perl-modules-5.30_5.30.0-9ubuntu0.3_all.deb ...
    Unpacking perl-modules-5.30 (5.30.0-9ubuntu0.3) ...
    Selecting previously unselected package libgdbm-compat4:amd64.
    Preparing to unpack .../01-libgdbm-compat4_1.18.1-5_amd64.deb ...
    Unpacking libgdbm-compat4:amd64 (1.18.1-5) ...
    Selecting previously unselected package libperl5.30:amd64.
    Preparing to unpack .../02-libperl5.30_5.30.0-9ubuntu0.3_amd64.deb ...
    Unpacking libperl5.30:amd64 (5.30.0-9ubuntu0.3) ...
    Selecting previously unselected package perl.
    Preparing to unpack .../03-perl_5.30.0-9ubuntu0.3_amd64.deb ...
    Unpacking perl (5.30.0-9ubuntu0.3) ...
    Selecting previously unselected package libjson-perl.
    Preparing to unpack .../04-libjson-perl_4.02000-2_all.deb ...
    Unpacking libjson-perl (4.02000-2) ...
    Selecting previously unselected package postgresql-client-common.
    Preparing to unpack .../05-postgresql-client-common_250.pgdg20.04+1_all.deb ...
    Unpacking postgresql-client-common (250.pgdg20.04+1) ...
    Selecting previously unselected package ssl-cert.
    Preparing to unpack .../06-ssl-cert_1.0.39_all.deb ...
    Unpacking ssl-cert (1.0.39) ...
    Selecting previously unselected package postgresql-common.
    Preparing to unpack .../07-postgresql-common_250.pgdg20.04+1_all.deb ...
    Adding 'diversion of /usr/bin/pg_config to /usr/bin/pg_config.libpq-dev by postgresql-common'
    Unpacking postgresql-common (250.pgdg20.04+1) ...
    Selecting previously unselected package libcommon-sense-perl.
    Preparing to unpack .../08-libcommon-sense-perl_3.74-2build6_amd64.deb ...
    Unpacking libcommon-sense-perl (3.74-2build6) ...
    Selecting previously unselected package libtypes-serialiser-perl.
    Preparing to unpack .../09-libtypes-serialiser-perl_1.0-1_all.deb ...
    Unpacking libtypes-serialiser-perl (1.0-1) ...
    Selecting previously unselected package libjson-xs-perl.
    Preparing to unpack .../10-libjson-xs-perl_4.020-1build1_amd64.deb ...
    Unpacking libjson-xs-perl (4.020-1build1) ...
    Selecting previously unselected package libllvm10:amd64.
    Preparing to unpack .../11-libllvm10_1%3a10.0.0-4ubuntu1_amd64.deb ...
    Unpacking libllvm10:amd64 (1:10.0.0-4ubuntu1) ...
    Selecting previously unselected package libpq5:amd64.
    Preparing to unpack .../12-libpq5_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking libpq5:amd64 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package libsensors-config.
    Preparing to unpack .../13-libsensors-config_1%3a3.6.0-2ubuntu1.1_all.deb ...
    Unpacking libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libsensors5:amd64.
    Preparing to unpack .../14-libsensors5_1%3a3.6.0-2ubuntu1.1_amd64.deb ...
    Unpacking libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Selecting previously unselected package libxslt1.1:amd64.
    Preparing to unpack .../15-libxslt1.1_1.1.34-4ubuntu0.20.04.1_amd64.deb ...
    Unpacking libxslt1.1:amd64 (1.1.34-4ubuntu0.20.04.1) ...
    Selecting previously unselected package postgresql-client-15.
    Preparing to unpack .../16-postgresql-client-15_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking postgresql-client-15 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package postgresql-15.
    Preparing to unpack .../17-postgresql-15_15.3-1.pgdg20.04+1_amd64.deb ...
    Unpacking postgresql-15 (15.3-1.pgdg20.04+1) ...
    Selecting previously unselected package sysstat.
    Preparing to unpack .../18-sysstat_12.2.0-2ubuntu0.2_amd64.deb ...
    Unpacking sysstat (12.2.0-2ubuntu0.2) ...
    Setting up perl-modules-5.30 (5.30.0-9ubuntu0.3) ...
    Setting up libsensors-config (1:3.6.0-2ubuntu1.1) ...
    Setting up libpq5:amd64 (15.3-1.pgdg20.04+1) ...
    Setting up libllvm10:amd64 (1:10.0.0-4ubuntu1) ...
    Setting up ssl-cert (1.0.39) ...
    Setting up libgdbm-compat4:amd64 (1.18.1-5) ...
    Setting up libsensors5:amd64 (1:3.6.0-2ubuntu1.1) ...
    Setting up libxslt1.1:amd64 (1.1.34-4ubuntu0.20.04.1) ...
    Setting up libperl5.30:amd64 (5.30.0-9ubuntu0.3) ...
    Setting up sysstat (12.2.0-2ubuntu0.2) ...

    Creating config file /etc/default/sysstat with new version
    update-alternatives: using /usr/bin/sar.sysstat to provide /usr/bin/sar (sar) in auto mode
    Created symlink /etc/systemd/system/multi-user.target.wants/sysstat.service → /lib/systemd/system/sysstat.service.
    Setting up perl (5.30.0-9ubuntu0.3) ...
    Setting up libjson-perl (4.02000-2) ...
    Setting up postgresql-client-common (250.pgdg20.04+1) ...
    Setting up libcommon-sense-perl (3.74-2build6) ...
    Setting up postgresql-client-15 (15.3-1.pgdg20.04+1) ...
    update-alternatives: using /usr/share/postgresql/15/man/man1/psql.1.gz to provide /usr/share/man/man1/psql.1.gz (psql.1.gz) in auto mode
    Setting up postgresql-common (250.pgdg20.04+1) ...
    Adding user postgres to group ssl-cert

    Creating config file /etc/postgresql-common/createcluster.conf with new version
    Building PostgreSQL dictionaries from installed myspell/hunspell packages...
    Removing obsolete dictionary files:
    '/etc/apt/trusted.gpg.d/apt.postgresql.org.gpg' -> '/usr/share/postgresql-common/pgdg/apt.postgresql.org.gpg'
    Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
    Setting up libtypes-serialiser-perl (1.0-1) ...
    Setting up postgresql-15 (15.3-1.pgdg20.04+1) ...
    Creating new PostgreSQL cluster 15/main ...
    /usr/lib/postgresql/15/bin/initdb -D /var/lib/postgresql/15/main --auth-local peer --auth-host scram-sha-256 --no-instructions
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /var/lib/postgresql/15/main ... ok
    creating subdirectories ... ok
    selecting dynamic shared memory implementation ... posix
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting default time zone ... Etc/UTC
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... ok
    syncing data to disk ... ok
    Setting up libjson-xs-perl (4.020-1build1) ...
    Processing triggers for systemd (245.4-4ubuntu3.21) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
    ubuntu@pg-srv3:~$ 
    ubuntu@pg-srv3:~$ pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
    ubuntu@pg-srv3:~$ 
    ```
***

> ### 6. Задачка под звездочкой: реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.
  * Text
    ```console
    ```
***

