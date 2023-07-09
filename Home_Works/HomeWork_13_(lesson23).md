<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Хранимые функции и процедуры." </h2></div>


> ### Цель: Создать триггер для поддержки витрины в актуальном состоянии.
> ### Описание/Пошаговая инструкция выполнения домашнего задания:
> ### Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ
> ### В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).
> ### Есть запрос для генерации отчета – сумма продаж по каждому товару.
> ### БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.
> ### Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)
> ### Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE

* Скрипт и развернутое описание задачи – в [hw_triggers.sql](https://disk.yandex.ru/d/l70AvknAepIJXQ)
    ```sql
    -- ДЗ тема: триггеры, поддержка заполнения витрин
    
    DROP SCHEMA IF EXISTS pract_functions CASCADE;
    CREATE SCHEMA pract_functions;
    
    SET search_path = pract_functions, publ
    
    -- товары:
    CREATE TABLE goods
    (
        goods_id    integer PRIMARY KEY,
        good_name   varchar(63) NOT NULL,
        good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
    );
    INSERT INTO goods (goods_id, good_name, good_price)
    VALUES 	(1, 'Спички хозайственные', .50),
    		(2, 'Автомобиль Ferrari FXX K', 185000000.01);
    
    -- Продажи
    CREATE TABLE sales
    (
        sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        good_id     integer REFERENCES goods (goods_id),
        sales_time  timestamp with time zone DEFAULT now(),
        sales_qty   integer CHECK (sales_qty > 0)
    );
    
    INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
    
    -- отчет:
    SELECT G.good_name, sum(G.good_price * S.sales_qty)
    FROM goods G
    INNER JOIN sales S ON S.good_id = G.goods_id
    GROUP BY G.good_name;
    
    -- с увеличением объёма данных отчет стал создаваться медленно
    -- Принято решение денормализовать БД, создать таблицу
    CREATE TABLE good_sum_mart
    (
    	good_name   varchar(63) NOT NULL,
    	sum_sale	numeric(16, 2)NOT NULL
    );
    
    -- Создать триггер (на таблице sales) для поддержки.
    -- Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE
    
    -- Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
    -- Подсказка: В реальной жизни возможны изменения цен.
    ```

***

* Зачищаем свои объекты БД, перед началом
    ```sql
    drop function sum_good() 
    DROP TRIGGER sum_good ON public.sales;
    drop table public.good_sum_smart
    drop table public.goods
    drop table public.sales
    select * from public.good_sum_smart;
    select * from public.goods;
    select * from public.sales;
    ```
