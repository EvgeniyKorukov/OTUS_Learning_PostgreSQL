<div align="center"><h2> 2. Настройка Consul </h2></div>

***
### Немного теории:
  * В концепции кластера все значения/параметры/статусы/прочее должны храниться в единном месте - `DCS (Distributed Config Store)` и должны быть одинаковыми для всех участников кластера. Другими словами - это единное хранилище вида "Ключ-Значение". 
  * PostgreSQL не умеет напрямую работать с `DCS (Distributed Config Store)` поэтому ему нужно некоторое ПО, в нашем случае - это [`Patroni`](https://github.com/zalando/patroni). Но о нем позже.
  * В качестве `DCS (Distributed Config Store)` для [`Patroni`](https://github.com/zalando/patroni) могут выступать:
    * [`Consul`](https://www.consul.io/). ❗️Именно его мы и будет использовать
    * [`etcd`](https://github.com/etcd-io/etcd)
    * [`ZooKeeper`](https://zookeeper.apache.org/)
  * `Consul` не чувствителен к вводу/выводу в отличие от `etcd`. Хотя и его правильно размещать на отдельных серверах.
  
***
### Установка и настройка `Consul`
  * Поскольку он закрыт для России, то скачиваем его обходными путями.
  * У нас будет стоять следующая версия
    ```bash
    ./consul version
    ```
    ```console
    ./consul version
    Consul v1.16.1
    Revision e0ab4d29
    Build Date 2023-08-05T21:56:29Z
    Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
    ```

  * Копируем бинарник с локальной машины на все 3 ВМ
    ```bash
    scp consul ubuntu@pg-srv1:/tmp/
    ```
    ```console
    scp consul ubuntu@pg-srv1:/tmp/
    consul                                                        100%  153MB   9.2MB/s   00:16    
    ```
    ```bash
    scp consul ubuntu@pg-srv2:/tmp/
    ```
    ```console
    scp consul ubuntu@pg-srv2:/tmp/
    consul                                                        100%  153MB   9.3MB/s   00:16    
    ```
    ```bash
    scp consul ubuntu@pg-srv3:/tmp/
    ```
    ```console
    scp consul ubuntu@pg-srv3:/tmp/
    consul                                                        100%  153MB  10.4MB/s   00:14    
    ```

  * На каждой из 3х ВМ создаем нужные каталоги и права, и делаем правильное размещение бинарника
    ```bash
    sudo mv /tmp/consul /usr/bin/
    sudo chmod +x /usr/bin/consul
    sudo useradd -r -c 'Consul DCS service' consul
    sudo mkdir -p /var/lib/consul /etc/consul.d
    sudo chown consul:consul /var/lib/consul /etc/consul.d
    sudo chmod 775 /var/lib/consul /etc/consul.d
    ```

***
###  Создаем конфигурацию для `Consul`
  * ❗️Сгенерируем ключ для консула на любой из нод кластера
    ```bash
    consul keygen
    ```
    ```console
    ubuntu@pg-srv1:~$ consul keygen
    XvaUAIrls/9iIxtZfWdF0P1XDfW38m4jJHPK+yXXneE=
    ubuntu@pg-srv1:~$ 
    ```

  * Создаем конфигурационный файл для консула [`/etc/consul.d/config.json`](config.json) на всех 3х ВМ
    ```json
    {
        "bind_addr": "0.0.0.0",
        "bootstrap_expect": 3,
        "client_addr": "0.0.0.0",
        "data_dir": "/var/lib/consul",
        "enable_script_checks": true,
        "dns_config": {
            "enable_truncate": true,
            "only_passing": true
        },
        "enable_syslog": true,
        "encrypt": "XvaUAIrls/9iIxtZfWdF0P1XDfW38m4jJHPK+yXXneE=",
        "leave_on_terminate": true,
        "log_level": "INFO",
        "rejoin_after_leave": true,
        "retry_join": [
            "pg-srv1",
            "pg-srv2",
            "pg-srv3"
        ],
        "server": true,
        "start_join": [
            "pg-srv1",
            "pg-srv2",
            "pg-srv3"
        ],
       "ui_config": { "enabled": true }
    }
    ```
  * Параметры конфига
    * `bind_addr` — адрес, на котором будет слушать наш сервер консул. Это может быть IP любого из наших сетевых интерфейсов или, как в данном примере, все.
    * `bootstrap_expect` — ожидаемое количество серверов в кластере.
    * `client_addr` — адрес, к которому будут привязаны клиентские интерфейсы.
    * `datacenter` — привязка сервера к конкретному датацентру. Нужен для логического разделения. Серверы с одинаковым датацентром должны находиться в одной локальной сети.
    * `data_dir` — каталог для хранения данных.
    * `domain` — домен, в котором будет зарегистрирован сервис.
    * `enable_script_checks` — разрешает на агенте проверку работоспособности.
    * `dns_config` — параметры для настройки DNS.
    * `enable_syslog` — разрешение на ведение лога.
    * `encrypt` — ключ для шифрования сетевого трафика. В качестве значения используем сгенерированный ранее.
    * `leave_on_terminate` — при получении сигнала на остановку процесса консула, корректно отключать ноду от кластера.
    * `log_level` — минимальный уровень события для отображения в логе. Возможны варианты "trace", "debug", "info", "warn", and "err".
    * `rejoin_after_leave` — по умолчанию, нода покидающая кластер не присоединяется к нему автоматически. Данная опция позволяет управлять данным поведением.
    * `retry_join` — перечисляем узлы, к которым можно присоединять кластер. Процесс будет повторяться, пока не завершиться успешно.
    * `server` — режим работы сервера.
    * `start_join` — список узлов кластера, к которым пробуем присоединиться при загрузке сервера.
    * `ui_config` — конфигурация для графического веб-интерфейса.

  * Проверяем корректность конфигурационного файла.
    ```bash
    consul validate /etc/consul.d/config.json
    ```
    ```console
  
    ```

***
###  Создаем настраиваем службу в ОС для `Consul`
  * В завершение настройки создадим юнит в systemd для возможности автоматического запуска сервиса
    ```bash
  
    ```
    ```console
  
    ```

  * Перечитываем конфигурацию systemd
    ```bash
    systemctl daemon-reload
    ```

  * Стартуем наш сервис
    ```bash
    systemctl start consul
    ```
    ```console
  
    ```            


  * Также разрешаем автоматический старт при запуске сервера
    ```bash
    systemctl enable consul
    ```
    ```console
  
    ```



  * Смотрим текущее состояние работы сервиса:
    ```bash
    systemctl status consul
    ```
    ```console
  
    ```



  * Состояние нод кластера мы можем посмотреть командой
    ```bash
    consul members
    ```
    ```console
  
    ```



  * А данной командой можно увидеть дополнительную информацию
    ```bash
    consul members -detailed
    ```
    ```console
  
    ```

  * Также у нас должен быть доступен веб-интерфейс по адресу:
    ```bash
  
    ```
    ```console
  
    ```



  * Консул установлен
    ```bash
  
    ```
    ```console
  
    ```


  * 
    ```bash
  
    ```
    ```console
  
    ```



  * 
    ```bash
  
    ```
    ```console
  
    ```
  * 
    ```bash
  
    ```
    ```console
  
    ```



  * 
    ```bash
  
    ```
    ```console
  
    ```      
