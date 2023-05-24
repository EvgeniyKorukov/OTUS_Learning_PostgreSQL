<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Резервное копирование и восстановление" </h2></div>


***

> ### 1. Создаем ВМ/докер c ПГ.
  * Создаем ВМ в VirtualBox и создаем кластер баз данных PostgreSQL
    ```console
    user@srv-pg-ubuntu:~$ pg_lsclusters
    Ver Cluster Port Status Owner Data directory Log file
    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo pg_createcluster 15 main --start
    [sudo] password for user:
    Creating new PostgreSQL cluster 15/main ...
    /usr/lib/postgresql/15/bin/initdb -D /var/lib/postgresql/15/main --auth-local peer --auth-host scram-sha-256 --no-instructions
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "C.UTF-8".
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
    Ver Cluster Port Status Owner    Data directory              Log file
    15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
    user@srv-pg-ubuntu:~$
    ```
 ***

> ### 2. Создаем БД, схему и в ней таблицу.
* Тут все просто и логи ниже, но таблица создается с указанием схемы и список таблиц смотрим через указание схемы `\dt schm_backup.*`
    ```console
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -c 'create database db_backup;'
    CREATE DATABASE
    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -c '\l'
                                                 List of databases
       Name    |  Owner   | Encoding | Collate |  Ctype  | ICU Locale | Locale Provider |   Access privileges
    -----------+----------+----------+---------+---------+------------+-----------------+-----------------------
     db_backup | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            |
     postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            |
     template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +
               |          |          |         |         |            |                 | postgres=CTc/postgres
     template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |            | libc            | =c/postgres          +
               |          |          |         |         |            |                 | postgres=CTc/postgres
    (4 rows)

    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c 'create schema schm_backup;'
    CREATE SCHEMA
    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c '\dn'
             List of schemas
        Name     |       Owner
    -------------+-------------------
     public      | pg_database_owner
     schm_backup | postgres
    (2 rows)

    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c 'create table schm_backup.tbl1(c1 int);'
    CREATE TABLE
    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c '\dt'
    Did not find any relations.
    user@srv-pg-ubuntu:~$
    user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c '\dt schm_backup.*'
               List of relations
       Schema    | Name | Type  |  Owner
    -------------+------+-------+----------
     schm_backup | tbl1 | table | postgres
    (1 row)

    user@srv-pg-ubuntu:~$
    ```
 ***

> ### 3. Заполним таблицы автосгенерированными 100 записями.
  * Text
    ```console
    ```
 ***

> ### 4. Под линукс пользователем Postgres создадим каталог для бэкапов
  * Text
    ```console
    ```
 ***

> ### 5. Сделаем логический бэкап используя утилиту COPY
  * Text
    ```console
    ```
    
  ***

> ### 6. Восстановим в 2 таблицу данные из бэкапа.
  * Text
    ```console
    ```
    
    
  ***

> ### 7. Используя утилиту pg_dump создадим бэкап с оглавлением в кастомном сжатом формате 2 таблиц
  * Text
    ```console
    ```
    
    
 ***

> ### 8. Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
  * Text
    ```console
    ```
      
 ***      
