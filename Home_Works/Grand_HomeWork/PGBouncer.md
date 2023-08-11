<div align="center"><h2> 4. Настройка PgBouncer </h2></div>

***
### Немного теории:
  * В `PostgreSQL` нет встроенного диспетчера/менеджера подключений и поэтому сессии могут съесть всю оперативную память и уронить экземпляр `PostgreSQL`
  * В качестве диспетчера подключений может выступать следующее ПО:
      * [`PgBouncer`](https://www.pgbouncer.org/) ❗️Именно его мы и будет использовать
      * [`PgPool`](https://www.pgpool.net/mediawiki/index.php/Main_Page)
      * [`Odyssey`](https://github.com/yandex/odyssey)

***


> ### 1. Text
   * Text
       :hammer_and_wrench: Параметр | :memo: Значение |
