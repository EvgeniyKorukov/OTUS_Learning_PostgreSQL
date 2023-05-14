<div align="center"><h2> Отчет о выполнении домашнего задания по теме: Журналы</h2></div>


***

> ### Настройте выполнение контрольной точки раз в 30 секунд
* Поскольку context=sighup, то чтобы применить параметр, будет достаточно SELECT pg_reload_conf()
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





3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой
объем приходится в среднем на одну контрольную точку.
pg_current_wal_lsn()
а вот как среднюю посчитать ?

select blks_hit-:blks_hit"blk hit",blks_read-:blks_read"blk read",tup_inserted-:tup_inserted"ins",tup_updated-:tup_updated"upd",tup_deleted-:tup_deleted"del",tup_returned-:tup_returned"tup ret",tup_fetched-:tup_fetched"tup fch",xact_commit-:xact_commit"commit",xact_rollback-:xact_rollback"rbk",pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(),:'pg_current_wal_lsn')) "WAL",pg_size_pretty(temp_bytes-:temp_bytes)"temp" from pg_stat_database where datname=current_database();

pg_wal_lsn_diff

SELECT pg_walfile_name(pg_current_wal_lsn());

-- посмотрим какой у нас wal file
SELECT pg_current_wal_insert_lsn();	
SELECT pg_walfile_name('0/1670928');	-- Выдаёт для заданной позиции в журнале предзаписи имя соответствующего файла WAL

SELECT '0/1672DB8'::pg_lsn - '0/1670928'::pg_lsn;
sudo /usr/lib/postgresql/13/bin/pg_waldump -p /var/lib/postgresql/13/main/pg_wal -s 0/1670928 -e 0/1672DB8 000000010000000000000001



4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию.
Почему так произошло?
?Это смотеть в логе?

--Checkpoint----
-- посмотрим информацию о кластере
sudo /usr/lib/postgresql/13/bin/pg_controldata /var/lib/postgresql/13/main/
SELECT pg_current_wal_insert_lsn();
CHECKPOINT;
SELECT pg_current_wal_insert_lsn();
sudo /usr/lib/postgresql/13/bin/pg_waldump -p /var/lib/postgresql/13/main/pg_wal -s 0/173B0D0 -e 0/173D2B0 000000010000000000000001
sudo /usr/lib/postgresql/13/bin/pg_waldump -p /var/lib/postgresql/13/main/pg_wal -s 2/E0048070 -e 2/E00481B8 0000000100000002000000E0


6. Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу.
Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите
кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и
продолжить работу?
Найти параметр игнорирующих поврежденные данные
=> SET ignore_checksum_failure = on;
=> SELECT * FROM wallevel;


\c buffer_temp
SELECT pg_relation_filepath('test_text');
-- Остановим сервер и поменяем несколько байтов в странице (сотрем из заголовка LSN последней журнальной записи)
dd if=/dev/zero of=/var/lib/postgresql/13/main/base/16384/16410 oflag=dsync conv=notrunc bs=1 count=8
-- запустим сервер и попробуем сделать выборку из таблицы


select count(*) as all, 
    count(*) filter (where filename > pg_walfile_name(pg_current_wal_lsn())) as future 
from pg_ls_dir('pg_wal') as t(filename);


https://dba.stackexchange.com/questions/192564/is-there-a-query-to-check-the-current-wal-size-in-postgresql
https://franckpachot.medium.com/postgresql-measuring-query-activity-wal-size-generated-shared-buffer-reads-filesystem-reads-15d2f9b4ca1f
https://dba.stackexchange.com/questions/192564/is-there-a-query-to-check-the-current-wal-size-in-postgresql
https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-WAL-VIEW
https://www.ktexperts.com/wal-internals-in-postgresql/



https://dataegret.com/2017/03/deep-dive-into-postgres-stats-pg_stat_bgwriter/
https://habr.com/ru/articles/505440/
https://sysadminium.ru/statistika_raboty_postgresql/
https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW
https://www.postgresql.org/docs/current/monitoring-stats.html
https://www.2ndquadrant.com/en/blog/measuring_postgresql_checkpoin/
https://sysadminium.ru/bufernyj_kesh_i_zhurnal_wal_v_postgresql/
https://habr.com/ru/companies/postgrespro/articles/496150/

---
pg_current_wal_lsn()


---
Контрольная точка
checkpoint_timeout = 5min
checkpoint_completion_target = 0.8

Настройка частоты срабатывания:
• checkpoint_timeout = 5min
• max_wal_size = 1GB
Сервер хранит журнальные файлы необходимые для восстановления:
• (2 (1 с 12 версии) + checkpoint_completion_target) * max_wal_size
• еще не прочитанные через слоты репликации
• еще не записанные в архив, если настроена непрерывная архивация
• не превышающие по объему минимальной отметки
Настройки
• max_wal_size = 1GB
• min_wal_size = 100MB
•walkeepsegments=0

Контрольная точка. Процесс фоновой записи
Настройки
• bgwriter_delay = 200ms
• bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
Алгоритм
• уснуть на bgwriter_delay
• если в среднем за цикл запрашивается N буферов, то записать
N * bgwriter_lru_multiplier ≤ bgwriter_lru_maxpages грязных буферов

---
Синхронизация с диском
данные должны дойти до энергонезависимого хранилища через многочисленные
кэши
СУБД сообщает операционной системе способом, указанным в wal_sync_method
надо учитывать аппаратное кэширование
Настройки
fsync = on
show fsync;
show wal_sync_method;
утилита pg_test_fsync помогает выбрать оптимальный способ


Режим синхронной записи
Алгоритм
● при фиксации изменений сбрасывает накопившиеся записи, включая
запись о фиксации
● ждет commit_delay, если активно не менее commit_siblings транзакций
Характеристики
● гарантируется долговечность
● увеличивается время отклика
Настройки
● synchronous_commit = on
● commit_delay = 0
● commit_siblings = 5
5
9
Режим асинхронной записи
Алгоритм
● циклы записи через wal_writer_delay
● записывает только целиком заполненные страницы
● но если новых полных страниц нет, то записывает последнюю до конца
Характеристики
● гарантируется согласованность, но не долговечность
● зафиксированные изменения могут пропасть (3 × wal_writer_delay)
● уменьшается время отклика
Настройки
● synchronous_commit = off (можно изменять на уровне транзакции)
● wal_writer_delay = 200ms

---


-- Попробуем нагрузочное тестирование в синхронном и асинхронном режиме
pgbench -i buffer_temp
pgbench -P 1 -T 10 buffer_temp

ALTER SYSTEM SET synchronous_commit = off;

pgbench -P 1 -T 10 buffer_temp
-- почему не увидели разницы???


SELECT pg_reload_conf(); -- конфигурацию-то не перечитали %)
sudo pg_ctlcluster 13 main reload
-- на простых старых hdd разница до 30 раз 



*__Домашнее задание
Работа с журналами
Цель:
уметь работать с журналами и контрольными точками
уметь настраивать параметры журналов
Описание/Пошаговая инструкция выполнения домашнего задания:
Настройте выполнение контрольной точки раз в 30 секунд.
10 минут c помощью утилиты pgbench подавайте нагрузку.
Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?__*
