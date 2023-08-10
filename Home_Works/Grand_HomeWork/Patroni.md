<div align="center"><h2> 3. Настройка Patroni </h2></div>

***

> ### Немного теории:
  * Для работы в режиме высокой доступности `HA` (High Availability) нужно некоторое ПО, которое контроллирует кто сейчас `master` или `главный сервер`, а кто `secondary` или `реплика`
  * В PostgreSQL это очень важно чтобы не было `Split Brain` чтобы в кластере не появилось больше одного `master`
  * Подобные функции может выполнять следующее ПО:
    * [`Pacemaker and Corosync`](https://clusterlabs.org/)
    * [`Patroni`](https://patroni.readthedocs.io/en/latest/) ❗️Именно его мы и будет использовать
    * [`RepMgr`](https://www.repmgr.org/)
    * [`Stolon`](https://github.com/sorintlab/stolon)
    * [`Slony`](https://www.slony.info/)
   
*** 
### Установка и настройка `Patroni`
  * П
  * У нас будет стоять следующая версия
    ```bash
    ```
    ```console
    ```
***
###  Создаем конфигурацию для `Patroni`
  * ❗️Сгенерируем ключ для консула на любой из нод кластера
    ```bash
    consul keygen
    ```
    ```console
    ubuntu@pg-srv1:~$ consul keygen
    XvaUAIrls/9iIxtZfWdF0P1XDfW38m4jJHPK+yXXneE=
    ubuntu@pg-srv1:~$ 
    ```
***    
###  Создаем настраиваем службу в ОС для `Patroni`
  * Создаем службу [`/usr/lib/systemd/system/consul.service`](consul.service) в ОС на каждой из 3х ВМ
  * ❗️Разница только в `-node=pg-srv`
    ```bash
    sudo vim /usr/lib/systemd/system/consul.service
    ```
    ```service
    [Unit]
    Description=Consul Service Discovery Agent
    Documentation=https://www.consul.io/
    After=network-online.target
    Wants=network-online.target
    
    [Service]
    Type=simple
    User=consul
    Group=consul
    ExecStart=/usr/bin/consul agent \
        -node=pg-srv \
        -config-dir=/etc/consul.d
    ExecReload=/bin/kill -HUP $MAINPID
    KillSignal=SIGINT
    TimeoutStopSec=5
    Restart=on-failure
    SyslogIdentifier=consul
    
    [Install]
    WantedBy=multi-user.target
    ```

  * Перечитываем конфигурацию systemd
    ```bash
    sudo systemctl daemon-reload
    ```

  * Стартуем наш сервис
    ```bash
    sudo systemctl start patroni
    ```

  * Также разрешаем автоматический старт при запуске сервера
    ```bash
    sudo systemctl enable patroni
    ```
    ```console
    ```
    ```console
    ```
    ```console
    ```

  * Смотрим текущее состояние работы сервиса:
    ```bash
    sudo systemctl status patroni
    ```
    ```console
    ```
    ```console
    ```
    ```console
    ```
    

***
###  Полезные команды `Patroni`
  * Посмотреть список нод
    ```bash
    patronictl -c /etc/patroni/patroni.yml list
    ```

  * Изменим параметры с помощью команды
    ```bash
    patronictl -c /etc/patroni/patroni.yml edit-config
    ```

  * Рестарт одной ноды
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart postgres postgresql-patroni1
    ```

  * Рестарт всего кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml restart postgres
    ```

  * Рестарт reload кластера
    ```bash
    patronictl -c /etc/patroni/patroni.yml reload postgres
    ```

  * Плановое переключение
    ```bash
    patronictl -c /etc/patroni/patroni.yml switchover postgres
    ```

  * Реинициализации ноды
    ```bash
    patronictl -c /etc/patroni/patroni.yml reinit postgres postgresql-patroni2
    ```

***

### :+1: Patroni установлен и настроен
