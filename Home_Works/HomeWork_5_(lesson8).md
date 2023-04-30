#### *Отчет о выполнении домашнего задания:*


> Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
  * **_Создал виртуальную машину на домашнем Proxmox с указанными параметрами_** 


> Установить на него PostgreSQL 15 с дефолтными настройками
  * **_Устанавливаем Postgres 15, создаем кластер баз данных и запускаем его_**  
    * sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
    * sudo pg_createcluster 15 main
    * sudo pg_ctlcluster start 15 main


> Создать БД для тестов: выполнить pgbench -i postgres
  * **_sudo -u postgres pgbench -i postgres_**


> Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
  * **_sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres_**
    * pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
    * starting vacuum...end.
    * progress: 6.0 s, 244.2 tps, lat 32.155 ms stddev 15.415, 0 failed
    * progress: 12.0 s, 256.7 tps, lat 31.141 ms stddev 14.846, 0 failed
    * progress: 18.0 s, 256.2 tps, lat 31.244 ms stddev 15.769, 0 failed
    * progress: 24.0 s, 258.0 tps, lat 30.995 ms stddev 14.555, 0 failed
    * progress: 30.0 s, 258.0 tps, lat 31.010 ms stddev 14.957, 0 failed
    * progress: 36.0 s, 257.5 tps, lat 31.035 ms stddev 14.797, 0 failed
    * progress: 42.0 s, 256.8 tps, lat 31.198 ms stddev 15.005, 0 failed
    * progress: 48.0 s, 249.7 tps, lat 32.038 ms stddev 15.395, 0 failed
    * progress: 54.0 s, 243.2 tps, lat 32.869 ms stddev 18.536, 0 failed
    * progress: 60.0 s, 256.7 tps, lat 31.178 ms stddev 15.165, 0 failed
    * transaction type: <builtin: TPC-B (sort of)>
    * scaling factor: 1
    * query mode: simple
    * number of clients: 8
    * number of threads: 1
    * maximum number of tries: 1
    * duration: 60 s
    * number of transactions actually processed: 15229
    * number of failed transactions: 0 (0.000%)
    * latency average = 31.480 ms
    * latency stddev = 15.478 ms
    * initial connection time = 92.608 ms
    * tps = 253.976325 (without initial connection time)


> Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
  * **_sudo -u postgres psql_**
    * alter system set max_connections = 40;
    * alter system set shared_buffers = '1GB';
    * alter system set effective_cache_size = '3GB';
    * alter system set maintenance_work_mem = '512MB';
    * alter system set checkpoint_completion_target = 0.9;
    * alter system set wal_buffers = '16MB';
    * alter system set default_statistics_target = 500;
    * alter system set random_page_cost = 4;
    * alter system set effective_io_concurrency = 2;
    * alter system set work_mem = '6553kB';
    * alter system set min_wal_size = '4GB';
    * alter system set max_wal_size = '16GB';
    * Перезагружаем кластер баз данных т.к. это надо для применения части параметров (context=postmaster в pg_settings)
      * sudo pg_ctlcluster restart 15 main
    * Параметры СУБД менются через alter system set. Здесь речь идет именно о параметрах, которые менялись т.к. они попадают в файл postgresql.auto.conf, и посмотреть их можно в: 
      * Файле параметров /var/lib/postgresql/15/main/postgresql.auto.conf
      * Запросом select * from pg_file_settings where sourcefile='/var/lib/postgresql/15/main/postgresql.auto.conf' order by name;


> Протестировать заново
  * **_sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres_**
    * pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
    * starting vacuum...end.
    * progress: 6.0 s, 252.2 tps, lat 31.042 ms stddev 14.160, 0 failed
    * progress: 12.0 s, 257.0 tps, lat 31.107 ms stddev 14.086, 0 failed
    * progress: 18.0 s, 265.3 tps, lat 30.144 ms stddev 14.743, 0 failed
    * progress: 24.0 s, 249.3 tps, lat 32.075 ms stddev 15.228, 0 failed
    * progress: 30.0 s, 262.5 tps, lat 30.461 ms stddev 15.467, 0 failed
    * progress: 36.0 s, 259.3 tps, lat 30.877 ms stddev 14.711, 0 failed
    * progress: 42.0 s, 266.5 tps, lat 30.019 ms stddev 13.657, 0 failed
    * progress: 48.0 s, 265.7 tps, lat 30.108 ms stddev 14.494, 0 failed
    * progress: 54.0 s, 260.7 tps, lat 30.681 ms stddev 14.189, 0 failed
    * progress: 60.0 s, 257.7 tps, lat 31.036 ms stddev 14.928, 0 failed
    * transaction type: <builtin: TPC-B (sort of)>
    * scaling factor: 1
    * query mode: simple
    * number of clients: 8
    * number of threads: 1
    * maximum number of tries: 1
    * duration: 60 s
    * number of transactions actually processed: 15585
    * number of failed transactions: 0 (0.000%)
    * latency average = 30.746 ms
    * latency stddev = 14.583 ms
    * initial connection time = 112.373 ms
    * tps = 260.079074 (without initial connection time)


> Что изменилось и почему?
 * Ниже наблюдения по работе. С точки зрения скорости работы СУБД-она стала быстрее работать, но не значительно. С учетом темы урока-не понял на чем конкретн в этом задании надо было сфокусировать свое внимание.
   * Увеличилось транзакций в секунду (tps) 
   * Увеличилось количество обработанных транзакций (number of transactions actually processed)
   * Уменьшилась средняя латентность (latency average)
   * Уменьшились заддержки(латентность) ввода/вывода (latency stddev) 
   * Увеличилось время подключения (initial connection time) 


> Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
  * **_sudo -u postgres psql_**
    * create table tab1(c1 text);
    * INSERT INTO tab1(c1) SELECT 'noname' FROM generate_series(1,1000000);


> Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('tab1'));_**
   * 35 MB 


> 5 раз обновить все строчки и добавить к каждой строчке любой символ
 * update tab1 set c1=c1||'*';
 * update tab1 set c1=c1||'#';
 * update tab1 set c1=c1||'1';
 * update tab1 set c1=c1||'*';
 * update tab1 set c1=c1||'7';


> Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
 * **_SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'tab1';_**
   * Мертвых строчек n_dead_tup нет т.к. avtovacuum быстрее нас :-) Но должно было быть примерно 5 млн. мертвых строк.
   * Автовакуум пришел в "2023-04-30 00:25:05.883613+00"
     * relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
     * ---------+------------+------------+--------+-------------------------------
     * tab1    |    1000000 |          0 |      0 | 2023-04-30 00:25:05.883613+00


> Подождать некоторое время, проверяя, пришел ли автовакуум
 * **_Он меня опередил, как писал выше_**


> 5 раз обновить все строчки и добавить к каждой строчке любой символ
 * update tab1 set c1=c1||'9';
 * update tab1 set c1=c1||'8';
 * update tab1 set c1=c1||'7';
 * update tab1 set c1=c1||'6';
 * update tab1 set c1=c1||'5';


> Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('tab1'));_**
   * 253 MB 


> Отключить Автовакуум на конкретной таблице
 * **_alter table tab1 set (autovacuum_enabled = off);_**


> 10 раз обновить все строчки и добавить к каждой строчке любой символ
 * update tab1 set c1=c1||'1';
 * update tab1 set c1=c1||'2';
 * update tab1 set c1=c1||'3';
 * update tab1 set c1=c1||'4';
 * update tab1 set c1=c1||'5';
 * update tab1 set c1=c1||'6';
 * update tab1 set c1=c1||'7';
 * update tab1 set c1=c1||'8';
 * update tab1 set c1=c1||'9';
 * update tab1 set c1=c1||'*';


> Посмотреть размер файла с таблицей
 * **_select pg_size_pretty(pg_table_size('tab1'));_**
   * 555 MB 


> Объясните полученный результат
 * **_Автовакуум выключен и мертвые строчки не удаляются. В итоге, у нас 10млн. мертвых строк. Но чтобы сжать файл на уровне ОС-надо выполнить vacuum full_**


> Не забудьте включить автовакуум)
 * **_alter table tab1 set (autovacuum_enabled = on);_**


> Задание со *:
> Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
> Не забыть вывести номер шага цикла.
 * _DO_
 * _$do$_
 * _BEGIN_
   * _FOR i IN 1..10 LOOP_
     * _RAISE NOTICE 'Step = %', i;_
     * _update tab1 set c1=c1||i;_
   * _END LOOP;_
 * _END_
 * _$do$;_


