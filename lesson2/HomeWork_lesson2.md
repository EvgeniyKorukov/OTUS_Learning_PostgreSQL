#### *Отчет о выполнении домашнего задания:*

> создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
> далее создать инстанс виртуальной машины с дефолтными параметрами
> добавить свой ssh ключ в metadata ВМ
> зайти удаленным ssh (первая сессия), не забывайте про ssh-add

* **_Установлен Ubuntu server 22.04.2 на виртуальной машине в VirtualBox_**  

>  поставить PostgreSQL

* **_Установлен Postgres 15_**  
* **sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15**
  

> зайти вторым ssh (вторая сессия)
> запустить везде psql из под пользователя postgres
* **_X_**

> выключить auto commit
* **_X_**

>  сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
>  посмотреть текущий уровень изоляции: show transaction isolation level
* **_X_**

>  начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
>  в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
>  сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему?
> завершить первую транзакцию - commit;
> сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему?
> завершите транзакцию во второй сессии
> начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
> в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
> сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему?
> завершить первую транзакцию - commit;
> сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему?
> завершить вторую транзакцию
> сделать select * from persons во второй сессии
> видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите

