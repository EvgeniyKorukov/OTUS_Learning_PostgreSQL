## **SQL и реляционные СУБД. Введение в PostgreSQL** ##


#### *Цели занятия:*
* объяснить основу реляционной модели данных;
* объяснить назначение языка SQL и его основные конструкции;
* иметь представление об основных реляционных СУБД;
* рассмотреть разницу в уровнях изоляции транзакций


#### *Краткое содержание:*
* реляционная модель и SQL;
* OLTP, ACID, MVCC, ARIES;
* уровни изоляции транзакций;
* современные РСУБД;
* введение в PostgreSQL и практика.


#### *Результаты:*
* будет говорит на одном языке с разработчиками и архитекторами БД;
* создаст собственный проект в GCP;
* проверит работу различных уровней изоляции транзакций в своем проекте


#### *Преподаватель:*
  * Евгений Аристов


#### *Компетенции:*
* владение базовыми навыками работы в SQL
    * основа реляционной модели данных
    * назначение языка SQL


#### *Дата и время:*
* 6 апреля, четверг в 20:00
* Длительность занятия: 90 минут


#### *Материалы:*
* [Создание_инстанса_в_GCP](https://cdn.otus.ru/media/private/e5/67/%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5_%D0%B8%D0%BD%D1%81%D1%82%D0%B0%D0%BD%D1%81%D0%B0_%D0%B2_GCP-25239-e56756.pdf?hash=fOP8UucOeQUqtkJqMxLjfQ&expires=1681021187)
* [#RuPostgres – о PostgreSQL на русском](https://www.youtube.com/channel/UC0SBGSNmBLrTZIkbN-lJHnw)
* [SQL exercises](https://sql-ex.ru/)
* [Software versioning - Wikipedia](https://en.wikipedia.org/wiki/Software_versioning)
* [Benchmark: MariaDB vs MySQL on Commodity Cloud Hardware | MariaDB](https://mariadb.com/resources/blog/benchmark-mariadb-vs-mysql-on-commodity-cloud-hardware/)
* [Что такое транзакция](https://habr.com/ru/post/537594/)
* [Хабр](https://habr.com/ru/company/miro/blog/540500/)
* [18 советов по использованию команд APT на Linux](https://gitjournal.tech/ispolzovanie-apt-v-linux-komandy-apt-i-apt-get/#i)
* [PostgreSQL: Versioning Policy](https://www.postgresql.org/support/versioning/)
* [02 PG SQL и реляционные СУБД. Введение в PostgreSQL](https://cdn.otus.ru/media/public/02/71/02_PG_SQL_%D0%B8_%D1%80%D0%B5%D0%BB%D1%8F%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D1%8B%D0%B5_%D0%A1%D0%A3%D0%91%D0%94._%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B2_PostgreSQL-25239-02714a.pdf)
* [02 PG intro](https://cdn.otus.ru/media/public/1f/60/02_PG_intro-25239-1f60e5.txt)


#### *Домашнее задание:*
Работа с уровнями изоляции транзакции в PostgreSQL
* Цель:
    * научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)
    * научиться управлять уровнем изолции транзации в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

#### *Описание/Пошаговая инструкция выполнения домашнего задания:*
* создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
* далее создать инстанс виртуальной машины с дефолтными параметрами
* добавить свой ssh ключ в metadata ВМ
* зайти удаленным ssh (первая сессия), не забывайте про ssh-add
* поставить PostgreSQL
* зайти вторым ssh (вторая сессия)
* запустить везде psql из под пользователя postgres
* выключить auto commit
* сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
* посмотреть текущий уровень изоляции: show transaction isolation level
* начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
* в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
* сделать select * from persons во второй сессии
* видите ли вы новую запись и если да то почему?
* завершить первую транзакцию - commit;
* сделать select * from persons во второй сессии
* видите ли вы новую запись и если да то почему?
* завершите транзакцию во второй сессии
* начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
* в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
* сделать select * from persons во второй сессии
* видите ли вы новую запись и если да то почему?
* завершить первую транзакцию - commit;
* сделать select * from persons во второй сессии
* видите ли вы новую запись и если да то почему?
* завершить вторую транзакцию
* сделать select * from persons во второй сессии
* видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите

