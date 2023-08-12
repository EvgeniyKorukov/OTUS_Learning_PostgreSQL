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

***
### Конфигурируем `PgBouncer`
  * Создаем файл конфигурации [`/etc/pgbouncer/pgbouncer.ini`](config_files/pgbouncer.ini) на всех 3х ВМ
  * ❗️Сохраняем оригинальный файл перед правкой
    ```bash
    sudo cp /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.orig
    ```
  * Правим файл конфигурации
    * Меняем параметр с `listen_addr = localhost` на `listen_addr = *`
    * Добавляем в раздел `[databases]` строку подключения ко всем БД `* = host=10.129.0.21 port=5432 dbname=postgres` на `ip` ❗️текущего сервера т.е. разница в `ip` ВМ 
    ```bash
    sudo vim /etc/pgbouncer/pgbouncer.ini
    ```

  * Создаем файл пользователей [`/etc/pgbouncer/userlist.txt`](config_files/userlist.txt) на всех 3х ВМ
    ```bash
    sudo -u postgres psql -Atq -h 127.0.0.1 -p 5432 -U postgres -d postgres -c "SELECT concat('\"', usename, '\" \"', passwd, '\"') FROM pg_shadow" >> /tmp/userlist.txt && \
    sudo mv /tmp/userlist.txt /etc/pgbouncer/userlist.txt
    ```

  * Запускаем `PgBouncer` на всех 3х ВМ
    ```bash  
    sudo systemctl start pgbouncer
    ```
      
  * Проверяем локальное подключение к `postgres` через `PgBouncer`
    ```bash  
    sudo -u postgres psql -p 6432 -h localhost
    ```
    
***
### Полезные команды `PgBouncer`
  * Запуск
    ```bash
    pgbouncer -d /etc/pgbouncer/pgbouncer.ini
    или
    sudo systemctl start pgbouncer
    ```
  * Перезапуск
    ```bash
    pgbouncer -R /etc/pgbouncer/pgbouncer.ini
    или
    sudo systemctl restart pgbouncer
    ```
  * Админка
    ```bash
    sudo -u postgres psql -p 6432 -U pgbouncer pgbouncer
    ```
  * Подключение через `PgBouncer`
    ```bash
    sudo -u postgres psql -p 6432 -h localhost
    ``` 
