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
```console
user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c 'INSERT INTO schm_backup.tbl1(c1) SELECT generate_series(1,100);'
INSERT 0 100
user@srv-pg-ubuntu:~$
user@srv-pg-ubuntu:~$ sudo -u postgres psql -d db_backup -c 'SELECT count(*) from schm_backup.tbl1;'
 count 
-------
   100
(1 row)

user@srv-pg-ubuntu:~$ 
```
***

> ### 4. Под линукс пользователем Postgres создадим каталог для бэкапов
```console
user@srv-pg-ubuntu:~$ sudo su - postgres
postgres@srv-pg-ubuntu:~$
postgres@srv-pg-ubuntu:~$ cd ~
postgres@srv-pg-ubuntu:~$
postgres@srv-pg-ubuntu:~$ pwd
/var/lib/postgresql
postgres@srv-pg-ubuntu:~$ mkdir backups
postgres@srv-pg-ubuntu:~$ ls
15  backups
postgres@srv-pg-ubuntu:~$
```
***

> ### 5. Сделаем логический бэкап используя утилиту COPY
```sql
postgres@srv-pg-ubuntu:~$ psql -d db_backup
psql (15.3 (Ubuntu 15.3-1.pgdg22.04+1))
Type "help" for help.

db_backup=# \copy schm_backup.tbl1 to '/var/lib/postgresql/backups/tbl1.sql';
COPY 100
db_backup=#
```
    
***

> ### 6. Восстановим в 2 таблицу данные из бэкапа.
```sql
db_backup=#
db_backup=# create table schm_backup.tbl2(c1 int);
CREATE TABLE
db_backup=#
db_backup=# \copy schm_backup.tbl2 from '/var/lib/postgresql/backups/tbl1.sql';
COPY 100
db_backup=#
db_backup=# select count(*) from schm_backup.tbl2;
 count
-------
   100
(1 row)

db_backup=#
```
    
    
***

> ### 7. Используя утилиту pg_dump создадим бэкап с оглавлением в кастомном сжатом формате 2 таблиц
* ❗️Если я правильно понял термин "бэкап с оглавлением", то это про опцию `--schema-only` т.е. бэкап со структурой, но без данных
  ```console
  postgres@srv-pg-ubuntu:~/backups$ pwd
  /var/lib/postgresql/backups
  postgres@srv-pg-ubuntu:~/backups$ pg_dump -d db_backup --schema-only -Fc > db_backup_all.gz
  postgres@srv-pg-ubuntu:~/backups$
  postgres@srv-pg-ubuntu:~/backups$ ls -ll
  total 8
  -rw-rw-r-- 1 postgres postgres 1500 May 24 22:18 db_backup_all.gz
  -rw-rw-r-- 1 postgres postgres  292 May 24 21:51 tbl1.sql
  postgres@srv-pg-ubuntu:~/backups$
  ```
   
    
***

> ### 8. Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
```console
postgres@srv-pg-ubuntu:~/backups$ echo "CREATE DATABASE otus;" | psql -U postgres
CREATE DATABASE
postgres@srv-pg-ubuntu:~/backups$ echo "CREATE SCHEMA schm_backup;" | psql -U postgres -d otus
CREATE SCHEMA
postgres@srv-pg-ubuntu:~/backups$
postgres@srv-pg-ubuntu:~/backups$ pg_restore -d otus -U postgres -n schm_backup -t tbl2  '/var/lib/postgresql/backups/db_backup_all.gz'
postgres@srv-pg-ubuntu:~/backups$
postgres@srv-pg-ubuntu:~/backups$ psql -d otus -c '\dt schm_backup.*'
           List of relations
   Schema    | Name | Type  |  Owner
-------------+------+-------+----------
 schm_backup | tbl2 | table | postgres
(1 row)

postgres@srv-pg-ubuntu:~/backups$ psql -d otus -c 'select * from schm_backup.tbl2;'
 c1
----
(0 rows)

postgres@srv-pg-ubuntu:~/backups$
```
      
***      
