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
  * ❗️Файл конфигурации разный для всех 3х ВМ
    * :information_source: Меняем параметр с `listen_addr = localhost` на `listen_addr = *`
    * :information_source: Добавляем в раздел `[databases]` строку подключения ко всем БД на данном сервере `* = host=10.129.0.X port=5432 dbname=postgres`
    * :information_source: Разница только в:
      * `* = host=10.129.0.X port=5432 dbname=postgres`
    ```bash
    sudo vim /etc/pgbouncer/pgbouncer.ini
    ```
      * :pencil2: Файл конфигурации для ВМ №1 [`/etc/pgbouncer/pgbouncer.ini`](config_files/vm1_pgbouncer.ini)
      * :pencil2: Файл конфигурации для ВМ №2 [`/etc/pgbouncer/pgbouncer.ini`](config_files/vm2_pgbouncer.ini)
      * :pencil2: Файл конфигурации для ВМ №3 [`/etc/pgbouncer/pgbouncer.ini`](config_files/vm3_pgbouncer.ini)

  * Создаем файл пользователей [`/etc/pgbouncer/userlist.txt`](config_files/userlist.txt) на всех 3х ВМ
    ```bash
    sudo -u postgres psql -Atq -h 127.0.0.1 -p 5432 -U postgres -d postgres -c "SELECT concat('\"', usename, '\" \"', passwd, '\"') FROM pg_shadow" >> /tmp/userlist.txt && \
    sudo mv /tmp/userlist.txt /etc/pgbouncer/userlist.txt
    ```

  * Включаем автозагрузку `PgBouncer` в ОС на всех 3х ВМ
    ```bash  
    sudo systemctl enable pgbouncer
    ```

  * Запускаем `PgBouncer` на всех 3х ВМ
    ```bash  
    sudo systemctl start pgbouncer
    ```
      
  * Проверяем локальное подключение к `postgres` через `PgBouncer`
    ```bash  
    sudo -u postgres psql -p 6432 -h localhost -c "\l"
    ```
    ```console
    ubuntu@pg-srv1:~$ sudo -u postgres psql -p 6432 -h localhost -c "\l"
    Password for user postgres: 
                                                     List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
    -----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
    (3 rows)
    
    ubuntu@pg-srv1:~$ 
    ```
    ```console
    ubuntu@pg-srv2:~$ sudo -u postgres psql -p 6432 -h localhost -c "\l"
    Password for user postgres: 
                                                     List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
    -----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
    (3 rows)
    
    ubuntu@pg-srv2:~$ 
    ```
    ```console
    ubuntu@pg-srv3:~$ sudo -u postgres psql -p 6432 -h localhost -c "\l"
    Password for user postgres: 
                                                     List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
    -----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | 
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
               |          |          |             |             |            |                 | postgres=CTc/postgres
    (3 rows)
    
    ubuntu@pg-srv3:~$ 
    ```
    
***
### Полезные команды `PgBouncer`
  * Посмотреть логи `PgBouncer` в ОС
    ```bash
    tail -20f /var/log/postgresql/pgbouncer.log
    ```
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
***

### Какие возможности у нас появились на данном этапе
  | :electric_plug: Появилось на этом этапе | :electric_plug: Компонент | :hammer_and_wrench: Название ВМ | :link: ip-адрес | :magnet: порт | :memo: Описание возможности |
  |:--:|:---|:--------|:----------|------:|:------------------------------------------|
  |  | `Consul` | **`pg-srv1`** | `10.129.0.21` | 8500 | Веб-интерфейс (http://10.129.0.21:8500) |
  |  | `Consul` | **`pg-srv2`** | `10.129.0.22` | 8500 | Веб-интерфейс (http://10.129.0.22:8500) |
  |  | `Consul` | **`pg-srv3`** | `10.129.0.23` | 8500 | Веб-интерфейс (http://10.129.0.23:8500) |
  |  | `PostgreSQL` | **`pg-srv1`** | `10.129.0.21` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `PostgreSQL` | **`pg-srv2`** | `10.129.0.22` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `PostgreSQL` | **`pg-srv3`** | `10.129.0.23` | 5432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 5432) |
  |  | `Patroni` | **`pg-srv1`** | `10.129.0.21` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.21:8008) |
  |  | `Patroni` | **`pg-srv2`** | `10.129.0.22` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.22:8008) |
  |  | `Patroni` | **`pg-srv3`** | `10.129.0.23` | 8008 | Ответ Patroni о роли экземпляра PostgreSQL на данной ВМ (curl http://10.129.0.23:8008) |
  | :heavy_check_mark: | `PgBouncer` | **`pg-srv1`** | `10.129.0.21` | 6432 | Подключение через `PgBouncer` к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.21) |
  | :heavy_check_mark: | `PgBouncer` | **`pg-srv2`** | `10.129.0.22` | 6432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.22) |
  | :heavy_check_mark: | `PgBouncer` | **`pg-srv3`** | `10.129.0.23` | 6432 | Прямое подключение к экземпляру PostgreSQL на данной ВМ (sudo -u postgres psql -p 6432 -h 10.129.0.23) |


***
### :+1: `PgBouncer` установлен и настроен
