<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Работа с индексами, join'ами, статистикой" </h2></div>

***
> ### 1 вариант:
***
> Создать индексы на БД, которые ускорят доступ к данным. В данном задании тренируются навыки:
> * определения узких мест
> * написания запросов для создания индекса
> * оптимизации
>
> Необходимо:
> 1. Создать индекс к какой-либо из таблиц вашей БД
  ```sql
  postgres=# create table tbl1 as
  select generate_series as pkey
          , generate_series::text || (random() * 10)::text as cnumber
      , (array['Masha', 'Dasha', 'Sasha', 'Dima', 'Tolya'])[floor(random() * 3 + 1)] as cname
  from generate_series(1, 100000);
  SELECT 100000
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where cname = 'Dasha';
                                                  QUERY PLAN                                                 
  -----------------------------------------------------------------------------------------------------------
   Seq Scan on tbl1  (cost=0.00..2072.00 rows=32943 width=33) (actual time=0.021..42.989 rows=33087 loops=1)
     Filter: (cname = 'Dasha'::text)
     Rows Removed by Filter: 66913
   Planning Time: 0.824 ms
   Execution Time: 47.361 ms
  (5 rows)
  
  postgres=#
  postgres=# create index tbl1_pkey_idx on tbl1(pkey);                    
  CREATE INDEX
  postgres=# 
  ```
> 2. Прислать текстом результат команды explain, в которой используется данный индекс
  ```sql
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where pkey = 3;
                                                       QUERY PLAN                                                      
  ---------------------------------------------------------------------------------------------------------------------
   Index Scan using tbl1_pkey_idx on tbl1  (cost=0.29..8.31 rows=1 width=33) (actual time=0.083..0.086 rows=1 loops=1)
     Index Cond: (pkey = 3)
   Planning Time: 0.510 ms
   Execution Time: 0.122 ms
  (4 rows)
  
  postgres=# 
  ```

> 3. Реализовать индекс для полнотекстового поиска
  ```sql

  ```

> 4. Реализовать индекс на часть таблицы или индекс на поле с функцией
  ```sql
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where cname = 'Dasha';
                                                  QUERY PLAN                                                 
  -----------------------------------------------------------------------------------------------------------
   Seq Scan on tbl1  (cost=0.00..2072.00 rows=32857 width=33) (actual time=0.021..51.959 rows=33087 loops=1)
     Filter: (cname = 'Dasha'::text)
     Rows Removed by Filter: 66913
   Planning Time: 0.637 ms
   Execution Time: 56.788 ms
  (5 rows)
  
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where upper(cname) = upper('Dasha');
                                                 QUERY PLAN                                                
  ---------------------------------------------------------------------------------------------------------
   Seq Scan on tbl1  (cost=0.00..2322.00 rows=500 width=33) (actual time=0.030..95.340 rows=33087 loops=1)
     Filter: (upper(cname) = 'DASHA'::text)
     Rows Removed by Filter: 66913
   Planning Time: 0.144 ms
   Execution Time: 98.421 ms
  (5 rows)
  
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where upper(cname) = 'DASHA';
                                                  QUERY PLAN                                                
  ----------------------------------------------------------------------------------------------------------
   Seq Scan on tbl1  (cost=0.00..2322.00 rows=500 width=33) (actual time=0.043..100.245 rows=33087 loops=1)
     Filter: (upper(cname) = 'DASHA'::text)
     Rows Removed by Filter: 66913
   Planning Time: 0.249 ms
   Execution Time: 103.751 ms
  (5 rows)
  
  postgres=# 
  postgres=# create index tbl1_idx_func on tbl1(upper(cname));
  CREATE INDEX
  postgres=# 
  postgres=# explain analyse                                  
  select * from tbl1 where cname = 'Dasha';
                                                  QUERY PLAN                                                 
  -----------------------------------------------------------------------------------------------------------
   Seq Scan on tbl1  (cost=0.00..2072.00 rows=32857 width=33) (actual time=0.011..33.174 rows=33087 loops=1)
     Filter: (cname = 'Dasha'::text)
     Rows Removed by Filter: 66913
   Planning Time: 1.535 ms
   Execution Time: 36.369 ms
  (5 rows)
  
  postgres=# 
  postgres=# explain analyse                                  
  select * from tbl1 where upper(cname) = upper('Dasha');
                                                           QUERY PLAN                                                         
  ----------------------------------------------------------------------------------------------------------------------------
   Bitmap Heap Scan on tbl1  (cost=8.17..764.29 rows=500 width=33) (actual time=5.277..29.704 rows=33087 loops=1)
     Recheck Cond: (upper(cname) = 'DASHA'::text)
     Heap Blocks: exact=822
     ->  Bitmap Index Scan on tbl1_idx_func  (cost=0.00..8.04 rows=500 width=0) (actual time=4.620..4.621 rows=33087 loops=1)
           Index Cond: (upper(cname) = 'DASHA'::text)
   Planning Time: 0.249 ms
   Execution Time: 37.834 ms
  (7 rows)
  
  postgres=# 
  postgres=# explain analyse                                  
  select * from tbl1 where upper(cname) = 'DASHA';
                                                           QUERY PLAN                                                         
  ----------------------------------------------------------------------------------------------------------------------------
   Bitmap Heap Scan on tbl1  (cost=8.17..764.29 rows=500 width=33) (actual time=5.042..28.940 rows=33087 loops=1)
     Recheck Cond: (upper(cname) = 'DASHA'::text)
     Heap Blocks: exact=822
     ->  Bitmap Index Scan on tbl1_idx_func  (cost=0.00..8.04 rows=500 width=0) (actual time=4.385..4.386 rows=33087 loops=1)
           Index Cond: (upper(cname) = 'DASHA'::text)
   Planning Time: 0.163 ms
   Execution Time: 36.323 ms
  (7 rows)
  
  postgres=# 
  ```
> 5. Создать индекс на несколько полей
  ```sql
  postgres=# 
  postgres=# create index tbl1_idx2 on tbl1(pkey,cname);
  CREATE INDEX
  postgres=# 
  postgres=# explain analyse
  select * from tbl1 where pkey=10 and cname = 'Dasha';
                                                     QUERY PLAN                                                    
  -----------------------------------------------------------------------------------------------------------------
   Index Scan using tbl1_idx2 on tbl1  (cost=0.42..8.44 rows=1 width=33) (actual time=0.059..0.062 rows=1 loops=1)
     Index Cond: ((pkey = 10) AND (cname = 'Dasha'::text))
   Planning Time: 0.651 ms
   Execution Time: 0.101 ms
  (4 rows)
  
  postgres=# 
  ```

> 6. Написать комментарии к каждому из индексов
  ```sql

  ```

> 7. Описать что и как делали и с какими проблемами столкнулись
  ```sql

  ```
