<div align="center"><h2> 4. Настройка PgBouncer </h2></div>

***
### Немного теории:
  * В `PostgreSQL` нет встроенного диспетчера/менеджера подключений и поэтому сессии могут съесть всю оперативную память и уронить экземпляр `PostgreSQL`
  * В качестве диспетчера подключений может выступать следующее ПО:
      * [`PgBouncer`](https://www.pgbouncer.org/) ❗️Именно его мы и будет использовать
        * Легковесный - 2 Кб на соединение
        * Можно выбрать тип соединения: на сессию, транзакцию или каждую операцию
        * Онлайн-реконфигурация без сброса подключений
      * [`PgPool`](https://www.pgpool.net/mediawiki/index.php/Main_Page)
        * Балансировка нагрузки (может заменить `HA Proxy`)
      * [`Odyssey`](https://github.com/yandex/odyssey)
        * Поделка от яндекса, говорят начала более-менее нормально работать.

*** 
### Список ВМ с которыми идет взаимодействие на данном этапе
   :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
   |--------------:|---------------|
   | **`pg-srv1`** | `10.129.0.21` |
   | **`pg-srv2`** | `10.129.0.22` |      
   | **`pg-srv3`** | `10.129.0.23` |
  
***
### Установка `PgBouncer`
  * Выполняем устаноку на всех 3х ВМ
    ```bash
    sudo apt -y install pgbouncer
    ```
  * Останавливаем службу/сервис в ОС на всех 3х ВМ
    ```bash
    sudo systemctl stop pgbouncer
    ```
  * ????Отключаем автозагрузку службы/сервиса в ОС на всех 3х ВМ
    ```bash
    sudo systemctl disable pgbouncer
    ```

***

### Конфигурируем `PgBouncer`
  * Создаем файл конфигурации [`/etc/pgbouncer/pgbouncer.ini`](pgbouncer.ini) на всех 3х ВМ
  * ❗️Сохраняем оригинальный файл перед правкой
    ```bash
    sudo cp /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.orig
    ```
    ```bash
    
    ```
    ```console

    ```
  * Создаем файл пользователей [`/etc/pgbouncer/userlist.txt`](userlist.txt) на всех 3х ВМ
    ```bash
    
    ```
    ```console

    ```
    
***
