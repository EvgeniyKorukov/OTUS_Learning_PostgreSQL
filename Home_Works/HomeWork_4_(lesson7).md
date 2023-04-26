#### *Отчет о выполнении домашнего задания:*


> создайте новый кластер PostgresSQL 14
* **_sudo pg_createcluster 14 main_**  
* **_sudo pg_ctlcluster start 14 main_**  
   * Это надо т.к. после создания кластера баз данных, они автоматом не запускаются, судя по выводу 
      * sudo pg_lsclusters
         * Ver Cluster Port Status Owner    Data directory              Log file
         * 14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log


> зайдите в созданный кластер под пользователем postgres
* **_sudo -u postgres psql или sudo -u postgres psql -U postgres_**  
* **_или_**  
* **_sudo -u postgres psql -U postgres_**  
    * Информацию о текущем подключении можно посмотреть через команду \conninfo в psql
    * Информацию о текущем пользователе можно получить через:
      * Запрос select current_user;
      * Запрос select user;


> создайте новую базу данных testdb
* **_create database testdb;_**  
    * Список БД можно посмотреть через:
      * Команду \l в psql
      * Запрос select * from pg_database;


> зайдите в созданную базу данных под пользователем postgres
* **_sudo -u postgres psql -d testdb_**  
* **_или_**  
* **_sudo -u postgres psql -d testdb -U postgres_**  
    * Информацию о текущем подключении можно посмотреть через команду \conninfo в psql
    * Информацию о текущем пользователе можно получить через:
      * Запрос select current_user;
      * Запрос select user;
    * Информацию о текущей БД можно получить через:
      * Запрос select current_database();


> создайте новую схему testnm
* **_create schema testnm;_**  
    * Список схем можно посмотреть через:
      * Команду \dn в psql
      * Запрос select * from pg_namespace;


> создайте новую таблицу t1 с одной колонкой c1 типа integer
* **_create table t1(c1 int);_**  
    * Список таблиц можно посмотреть через: 
      * Команду \dt в psql
      * Запрос select * from pg_tables;


> вставьте строку со значением c1=1
* **_insert into t1(c1) values(1);_**  


> создайте новую роль readonly
* **_create role readonly;_**  
    * Список ролей можно посмотреть через: 
      * Команду \dg в psql
      * Команду \du в psql (список ролей и пользователей)
      * Запрос select * from pg_roles;


> дайте новой роли право на подключение к базе данных testdb
* **_grant connect on database testdb to readonly;_**  
    * Список выданных прав для БД testdb можно посмотреть через: 
      * Команду \l+ testdb в psql
      * Запрос select * from pg_database where datname='testdb';


> дайте новой роли право на использование схемы testnm
* **_grant usage on schema testnm to readonly;_**  
    * Список выданных прав для схемы testnm можно посмотреть через: 
      * Команду \dn+ testnm в psql
      * select * from pg_namespace where nspname='testnm';


> дайте новой роли право на select для всех таблиц схемы testnm
* **_grant select on all tables in schema testnm to readonly;_**  
    * Явный смысл не понятен т.к. в схеме testnm еще нет ни одной таблицы. Видимо это для дальнейшей части эксперимента или для разбора с with admin option или with grant option


> создайте пользователя testread с паролем test123
* **_create user testread with password 'test123';_**  
    * Список пользователей можно посмотреть через: 
      * Команду \du в psql
      * Запрос select * from pg_user;


> дайте роль readonly пользователю testread
* **_grant readonly to testread;_**  
    * Список выданных прав для пользователя testread можно посмотреть через: 
      * Команду \du+ testread в psql
      * Запрос select roleid::regrole, member::regrole, grantor::regrole from pg_auth_members where member::regrole::text='testread';


> зайдите под пользователем testread в базу данных testdb
* **_!????sudo -u postgres psql -U testread -d testdb_**  
    * Text

 
> сделайте select * from t1;
* **_ToDo_**  
    * Text


> получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
* **_ToDo_**  
    * Text


> напишите что именно произошло в тексте домашнего задания
* **_ToDo_**  
    * Text
    * 
> у вас есть идеи почему? ведь права то дали?
* **_ToDo_**  
    * Text
    * 
> посмотрите на список таблиц
* **_ToDo_**  
    * Text
    * 
> подсказка в шпаргалке под пунктом 20
* **_ToDo_**  
    * Text
    * 
> а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
* **_ToDo_**  
    * Text
    * 
> вернитесь в базу данных testdb под пользователем postgres
* **_ToDo_**  
    * Text
    * 
> удалите таблицу t1
* **_ToDo_**  
    * Text
    * 
> создайте ее заново но уже с явным указанием имени схемы testnm
* **_ToDo_**  
    * Text
    * 
> вставьте строку со значением c1=1
* **_ToDo_**  
    * Text
    * 
> зайдите под пользователем testread в базу данных testdb
* **_ToDo_**  
    * Text
    * 
> сделайте select * from testnm.t1;
* **_ToDo_**  
    * Text
    * 
> получилось?
* **_ToDo_**  
    * Text
    * 
> есть идеи почему? если нет - смотрите шпаргалку
* **_ToDo_**  
    * Text
    * 
> как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
* **_ToDo_**  
    * Text
    * 
> сделайте select * from testnm.t1;
* **_ToDo_**  
    * Text
    * 
> получилось?
* **_ToDo_**  
    * Text
    * 
> есть идеи почему? если нет - смотрите шпаргалку
* **_ToDo_**  
    * Text
    * 
> сделайте select * from testnm.t1;
* **_ToDo_**  
    * Text
    * 
> получилось?
* **_ToDo_**  
    * Text
    * 
> ура!
* **_ToDo_**  
    * Text
    * 
> теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
* **_ToDo_**  
    * Text
    * 
> а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
* **_ToDo_**  
    * Text
    * 
> есть идеи как убрать эти права? если нет - смотрите шпаргалку
* **_ToDo_**  
    * Text
    * 
> если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
* **_ToDo_**  
    * Text
    * 
> теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
* **_ToDo_**  
    * Text
    * 
> расскажите что получилось и почему
* **_ToDo_**  
    * Text
