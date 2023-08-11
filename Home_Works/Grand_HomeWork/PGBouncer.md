<div align="center"><h2> 4. Настройка PgBouncer </h2></div>

***
### Немного теории:
  * В `PostgreSQL` нет встроенного диспетчера/менеджера подключений и поэтому сессии могут съесть всю оперативную память и уронить экземпляр `PostgreSQL`
  * В качестве диспетчера подключений может выступать следующее ПО:
      * [`PgBouncer`](https://www.pgbouncer.org/) ❗️Именно его мы и будет использовать
      * [`PgPool`](https://www.pgpool.net/mediawiki/index.php/Main_Page)
      * [`Odyssey`](https://github.com/yandex/odyssey)

*** 
### Список ВМ с которыми идет взаимодействие на данном этапе
  :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
  |--------------:|---------------|
  | **`pg-srv1`** | `10.129.0.21` |
  | **`pg-srv2`** | `10.129.0.22` |      
  | **`pg-srv3`** | `10.129.0.23` |
  
***



> ### 1. Text
   * Text
       :hammer_and_wrench: Параметр | :memo: Значение |
