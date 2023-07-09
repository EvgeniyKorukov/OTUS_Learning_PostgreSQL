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
    VALUES 	(1, 'Спички хозяйственные', .50),
    		(2, 'Автомобиль Toyota RAV-4', 185000000.01);
    
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

* Создаем функцию
    ```plpgsql
    CREATE OR REPLACE FUNCTION sum_good() RETURNS TRIGGER AS $$
    
    DECLARE
     v_qty integer;
     v_good_id integer;
    begin
    IF (TG_OP = 'INSERT') THEN
    	v_qty = NEW.sales_qty;
     	v_good_id = NEW.good_id; 
     ELSIF (TG_OP = 'UPDATE') THEN
    	 v_qty = NEW.sales_qty - OLD.sales_qty;
     	 v_good_id = OLD.good_id;
     ELSIF (TG_OP = 'DELETE') THEN
     	v_qty = 0 - OLD.sales_qty;
     	v_good_id = OLD.good_id;
    END IF;
    
    INSERT INTO good_sum_smart (good_name, sum_sale)
    SELECT good_name , good_price * v_qty
    FROM goods WHERE goods_id = v_good_id
    ON CONFLICT ON CONSTRAINT good_sum_smart_pkey
    DO UPDATE SET sum_sale = good_sum_smart.sum_sale + EXCLUDED.sum_sale
    WHERE good_sum_smart.good_name = EXCLUDED.good_name;
    RETURN NULL;
    END;
    $$ LANGUAGE plpgsql;
    ```

* Создаем триггер
    ```plpgsql
    CREATE TRIGGER sum_good
    AFTER INSERT OR UPDATE OR DELETE ON sales
    FOR EACH ROW EXECUTE FUNCTION sum_good();
    ```

* Для проверки работы триггера и функции
    * сначала очистим таблицу продаж
        ```sql
        truncate table public.sales;
        truncate table public.good_sum_smart;
        ``` 
    * а пототом заполним
        ```sql
        INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
        ```
    * Выполним SQL для отчета
        ```sql
        SELECT G.good_name, sum(G.good_price * S.sales_qty)
        FROM goods G
        INNER JOIN sales S ON S.good_id = G.goods_id
        GROUP BY G.good_name;
        ```
        ```console
                good_name         |     sum
        --------------------------+--------------
         Автомобиль Toyota RAV-4  | 185000000.01
         Спички хозяйственные     |        65.50
        (2 rows)
        ```
        
    * Проверим содержимое таблицы `public.good_sum_smart` и что тригер `sum_good` отработал
        ```sql
        select * from public.good_sum_smart;
        ```
        ```console
                good_name         |   sum_sale
        --------------------------+--------------
         Спички хозяйственные     |        65.50
         Автомобиль Toyota RAV-4  | 185000000.01
        (2 rows)
        ```

    * Сделаем продажу одной Toyota RAV-4
        ```sql
        INSERT INTO sales(good_id, sales_qty) VALUES (2, 1);
        ```

    * Проверим содержимое таблицы `public.good_sum_smart` и что тригер `sum_good` отработал
        ```sql
        select * from public.good_sum_smart;
        ```
        ```console
                good_name         |   sum_sale
        --------------------------+--------------
         Спички хозяйственные     |        65.50
         Автомобиль Toyota RAV-4  | 370000000.02
        (2 rows)
        ```
        
    * Выполним обновление продаж, как оказалось мы продали за один раз две штуки Toyota RAV-4 :-)
        ```sql
        update sales set sales_qty = 2 where good_id = 2 and sales_id = 15;
        ```
    
    * Проверим содержимое таблицы `public.good_sum_smart` и что тригер `sum_good` отработал
        ```sql
        select * from public.good_sum_smart;
        ```
        ```console
                good_name         |   sum_sale
        --------------------------+--------------
         Спички хозяйственные     |        65.50
         Автомобиль Toyota RAV-4  | 555000000.03
        (2 rows)
        ```

    * Удалим одну операцию продажи, клиенту не одобрили кредит в банке и продажа не состоялась :-(
        ```sql
        delete from sales where good_id = 2 and sales_id = 15;
        ```
    
    * Проверим содержимое таблицы `public.good_sum_smart` и что тригер `sum_good` отработал
        ```sql
        select * from public.good_sum_smart;
        ```
        ```console
                good_name         |   sum_sale
        --------------------------+--------------
         Спички хозяйственные     |        65.50
         Автомобиль Toyota RAV-4  | 185000000.01
        (2 rows)
        ```
