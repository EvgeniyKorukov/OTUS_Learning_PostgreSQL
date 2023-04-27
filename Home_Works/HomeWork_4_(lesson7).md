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
* **_sudo -u postgres psql -d testdb -U testread -h 127.0.0.1_**  
    * Если не указать имя хоста (-h 127.0.0.1) то попадаем на ошибку ниже в pg_hba.conf для peer. 
      * Текст ошибки в логах postgres
        * testread@testdb LOG:  provided user name (testread) and authenticated user name (postgres) do not match
        * testread@testdb FATAL:  Peer authentication failed for user "testread"
        * testread@testdb DETAIL:  Connection matched pg_hba.conf line 95: "local   all             all                                     peer"
    * А у пользователя testread пароль хранится в scram-sha-256, поэтому наше рабочее правило в pg_hba.conf
      * IPv4 local connections:
      * host    all             all             127.0.0.1/32            scram-sha-256
    * Очень надеюсь, что более-менее понятно объяснил :-) 
 
 
> сделайте select * from t1;
* **_Сделал_**


> получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
* **_Нет_**  


> напишите что именно произошло в тексте домашнего задания
* **_Получили ошибку_**  
    * ERROR:  permission denied for table t1

> у вас есть идеи почему? ведь права то дали?
* **_Мы выдавали роли readonly права на выборку всех таблиц в схеме testnm, а таблица t1 находится в схеме public. В данном случае пользователь testread является частью роли(группы) readonly. Но у нас нет наследования прав_**  


> посмотрите на список таблиц
* **_Таблица t1 находится в схеме public_**  


> подсказка в шпаргалке под пунктом 20
* **_Не понимаю про какую подсказку идет речь. Но, можно создавать объекты с явным указание имени схемы. Или выдать пользователю testread (или группе readonly) права на выборку всех таблиц в схеме public_**  


> а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
* **_Могли явно указать имя схемы при создании таблицы_**  


> вернитесь в базу данных testdb под пользователем postgres
* **_sudo -u postgres psql -d testdb -U postgres_**  


> удалите таблицу t1
* **_drop table t1;_**  


> создайте ее заново но уже с явным указанием имени схемы testnm
* **_create table testnm.t1(c1 int);_**  


> вставьте строку со значением c1=1
* **_insert into testnm.t1(c1) values(1);_**  


> зайдите под пользователем testread в базу данных testdb
* **_sudo -u postgres psql -d testdb -U testread -h 127.0.0.1_**  


> сделайте select * from testnm.t1;
* **_Сделал_**  


> получилось?
* **_Нет, ошибка ниже_**  
    * ERROR:  permission denied for table t1


> есть идеи почему? если нет - смотрите шпаргалку
* **_Идеи есть и самая простая-выдать явные права на выборку таблицы testnm.t1 пользователю testread_**  
    * grant select on testnm.t1 to testread;


> как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
* **_Выдать пользователю testread или роли(группе) readonly права на выборку всех таблиц в схеме testnm. Или же заняться вопросом наследования прав_**  
    * grant select on all tables in schema testnm to testread;
    * или
    * grant select on all tables in schema testnm to readonly;


> сделайте select * from testnm.t1;
* **_Сделал_**  


> получилось?
* **_Да_**  


> есть идеи почему? если нет - смотрите шпаргалку
* **_Да уже все описал выше)_**  

> сделайте select * from testnm.t1;
* **_Сделал_**  


> получилось?
* **_Да_**  


> ура!
* **_Кто бы сомневался)_**  


> теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
* **_Сделал_**  


> а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
* **_Так нам никто и не запрещал создавать объекты в схеме public. А раз создали таблицу, то мы становимся ее владельцем и имеем полные права на нее_**  


> есть идеи как убрать эти права? если нет - смотрите шпаргалку
* **_Есть_**  
    * REVOKE ALL ON SCHEMA public FROM readonly,testread;
    * REVOKE ALL ON ALL TABLES IN SCHEMA public FROM testread, readonly;

 
> если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
* **_Все описал чуть выше)_**  


> теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
* **_Попробовал_**  


> расскажите что получилось и почему
* **_Таблица создается, но не вставляются данные в t2 т.к. прав нет, мы их забрали_**  

