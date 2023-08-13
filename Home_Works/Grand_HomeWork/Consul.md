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
### Список ВМ с которыми идет взаимодействие на данном этапе
  :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
  |--------------:|---------------|
  | **`pg-srv1`** | `10.129.0.21` |
  | **`pg-srv2`** | `10.129.0.22` |      
  | **`pg-srv3`** | `10.129.0.23` |
  
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

  * Создаем конфигурационный файл для консула [`/etc/consul.d/config.json`](config_files/config.json) на всех 3х ВМ
    ```bash
    sudo vim /etc/consul.d/config.json
    ```
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
    ubuntu@pg-srv1:~$ consul validate /etc/consul.d/config.json
    The 'start_join' field is deprecated. Use the 'retry_join' field instead.
    bootstrap_expect > 0: expecting 3 servers
    using enable-script-checks without ACLs and without allow_write_http_from is DANGEROUS, use enable-local-script-checks instead, see https://www.hashicorp.com/blog/protecting-consul-from-rce-risk-in-specific-configurations/
    Configuration is valid!
    ubuntu@pg-srv1:~$ 
    ```
    ```console
    ubuntu@pg-srv2:~$ consul validate /etc/consul.d/config.json
    The 'start_join' field is deprecated. Use the 'retry_join' field instead.
    bootstrap_expect > 0: expecting 3 servers
    using enable-script-checks without ACLs and without allow_write_http_from is DANGEROUS, use enable-local-script-checks instead, see https://www.hashicorp.com/blog/protecting-consul-from-rce-risk-in-specific-configurations/
    Configuration is valid!
    ubuntu@pg-srv2:~$ 
    ```
    ```console
    ubuntu@pg-srv3:~$ consul validate /etc/consul.d/config.json
    The 'start_join' field is deprecated. Use the 'retry_join' field instead.
    bootstrap_expect > 0: expecting 3 servers
    using enable-script-checks without ACLs and without allow_write_http_from is DANGEROUS, use enable-local-script-checks instead, see https://www.hashicorp.com/blog/protecting-consul-from-rce-risk-in-specific-configurations/
    Configuration is valid!
    ubuntu@pg-srv3:~$ 
    ```

***

###  Создаем настраиваем службу в ОС для `Consul`
  * Создаем службу [`/usr/lib/systemd/system/consul.service`](config_files/consul.service) в ОС на каждой из 3х ВМ
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
    sudo systemctl start consul
    ```

  * Также разрешаем автоматический старт при запуске сервера
    ```bash
    sudo systemctl enable consul
    ```
    ```console
    ubuntu@pg-srv1:~$ sudo systemctl enable consul
    Created symlink /etc/systemd/system/multi-user.target.wants/consul.service → /lib/systemd/system/consul.service.
    ubuntu@pg-srv1:~$ 
    ```
    ```console
    ubuntu@pg-srv2:~$ sudo systemctl enable consul
    Created symlink /etc/systemd/system/multi-user.target.wants/consul.service → /lib/systemd/system/consul.service.
    ubuntu@pg-srv2:~$ 
    ```
    ```console
    ubuntu@pg-srv3:~$ sudo systemctl enable consul
    Created symlink /etc/systemd/system/multi-user.target.wants/consul.service → /lib/systemd/system/consul.service.
    ubuntu@pg-srv3:~$ 
    ```

  * Смотрим текущее состояние работы сервиса:
    ```bash
    sudo systemctl status consul
    ```
    ```console
    ubuntu@pg-srv1:~$ sudo systemctl status consul
    ● consul.service - Consul Service Discovery Agent
         Loaded: loaded (/lib/systemd/system/consul.service; enabled; vendor preset: enabled)
         Active: active (running) since Thu 2023-08-10 21:36:38 UTC; 5min ago
           Docs: https://www.consul.io/
       Main PID: 14398 (consul)
          Tasks: 8 (limit: 4631)
         Memory: 34.0M
         CGroup: /system.slice/consul.service
                 └─14398 /usr/bin/consul agent -node=pg-srv1 -config-dir=/etc/consul.d
    
    Aug 10 21:37:11 pg-srv1 consul[14398]: 2023-08-10T21:37:11.272Z [INFO]  agent.leader: stopped r>
    Aug 10 21:37:11 pg-srv1 consul[14398]: agent.leader: stopped routine: routine="virtual IP versi>
    Aug 10 21:37:11 pg-srv1 consul[14398]: 2023-08-10T21:37:11.277Z [INFO]  agent.server: member jo>
    Aug 10 21:37:11 pg-srv1 consul[14398]: agent.server: member joined, marking health alive: membe>
    Aug 10 21:37:11 pg-srv1 consul[14398]: 2023-08-10T21:37:11.303Z [INFO]  agent.server: member jo>
    Aug 10 21:37:11 pg-srv1 consul[14398]: agent.server: member joined, marking health alive: membe>
    Aug 10 21:37:11 pg-srv1 consul[14398]: 2023-08-10T21:37:11.607Z [INFO]  agent.server: federatio>
    Aug 10 21:37:11 pg-srv1 consul[14398]: agent.server: federation state anti-entropy synced
    Aug 10 21:37:15 pg-srv1 consul[14398]: 2023-08-10T21:37:15.491Z [INFO]  agent: Synced node info
    Aug 10 21:37:15 pg-srv1 consul[14398]: agent: Synced node info
    lines 1-20/20 (END)
    ```
    ```console
    ubuntu@pg-srv2:~$ sudo systemctl status consul
    ● consul.service - Consul Service Discovery Agent
         Loaded: loaded (/lib/systemd/system/consul.service; enabled; vendor preset: enabled)
         Active: active (running) since Thu 2023-08-10 21:37:07 UTC; 3min 15s ago
           Docs: https://www.consul.io/
       Main PID: 13761 (consul)
          Tasks: 8 (limit: 4631)
         Memory: 32.4M
         CGroup: /system.slice/consul.service
                 └─13761 /usr/bin/consul agent -node=pg-srv2 -config-dir=/etc/consul.d
    
    Aug 10 21:37:08 pg-srv2 consul[13761]: 2023-08-10T21:37:08.611Z [INFO]  agent.server: Found exp>
    Aug 10 21:37:08 pg-srv2 consul[13761]: agent.server: Found expected number of peers, attempting>
    Aug 10 21:37:08 pg-srv2 consul[13761]: 2023-08-10T21:37:08.619Z [INFO]  agent.server.serf.wan: >
    Aug 10 21:37:08 pg-srv2 consul[13761]: agent.server.serf.wan: serf: EventMemberJoin: pg-srv3.dc>
    Aug 10 21:37:08 pg-srv2 consul[13761]: 2023-08-10T21:37:08.619Z [INFO]  agent.server: Handled e>
    Aug 10 21:37:08 pg-srv2 consul[13761]: agent.server: Handled event for server in area: event=me>
    Aug 10 21:37:11 pg-srv2 consul[13761]: 2023-08-10T21:37:11.271Z [INFO]  agent.server: New leade>
    Aug 10 21:37:11 pg-srv2 consul[13761]: agent.server: New leader elected: payload=pg-srv1
    Aug 10 21:37:16 pg-srv2 consul[13761]: 2023-08-10T21:37:16.418Z [INFO]  agent: Synced node info
    Aug 10 21:37:16 pg-srv2 consul[13761]: agent: Synced node info
    lines 1-20/20 (END)
    ```
    ```console
    ubuntu@pg-srv3:~$ sudo systemctl status consul
    ● consul.service - Consul Service Discovery Agent
         Loaded: loaded (/lib/systemd/system/consul.service; enabled; vendor preset: enabled)
         Active: active (running) since Thu 2023-08-10 21:37:08 UTC; 4min 15s ago
           Docs: https://www.consul.io/
       Main PID: 14126 (consul)
          Tasks: 8 (limit: 4631)
         Memory: 32.4M
         CGroup: /system.slice/consul.service
                 └─14126 /usr/bin/consul agent -node=pg-srv3 -config-dir=/etc/consul.d
    
    Aug 10 21:37:08 pg-srv3 consul[14126]: agent.server: Handled event for server in area: event=me>
    Aug 10 21:37:08 pg-srv3 consul[14126]: agent.server: Handled event for server in area: event=me>
    Aug 10 21:37:08 pg-srv3 consul[14126]: 2023-08-10T21:37:08.624Z [INFO]  agent: (LAN) joined: nu>
    Aug 10 21:37:08 pg-srv3 consul[14126]: 2023-08-10T21:37:08.624Z [INFO]  agent: Join cluster com>
    Aug 10 21:37:08 pg-srv3 consul[14126]: agent: (LAN) joined: number_of_nodes=6
    Aug 10 21:37:08 pg-srv3 consul[14126]: agent: Join cluster completed. Synced with initial agent>
    Aug 10 21:37:11 pg-srv3 consul[14126]: 2023-08-10T21:37:11.272Z [INFO]  agent.server: New leade>
    Aug 10 21:37:11 pg-srv3 consul[14126]: agent.server: New leader elected: payload=pg-srv1
    Aug 10 21:37:12 pg-srv3 consul[14126]: 2023-08-10T21:37:12.092Z [INFO]  agent: Synced node info
    Aug 10 21:37:12 pg-srv3 consul[14126]: agent: Synced node info
    lines 1-20/20 (END)
    ```
    
***
###  Полезные команды `Consul`
  * Состояние нод кластера мы можем посмотреть командой
    ```bash
    consul members
    ```
    ```console
    ubuntu@pg-srv1:~$ consul members
    Node     Address           Status  Type    Build   Protocol  DC   Partition  Segment
    pg-srv1  10.129.0.21:8301  alive   server  1.16.1  2         dc1  default    <all>
    pg-srv2  10.129.0.22:8301  alive   server  1.16.1  2         dc1  default    <all>
    pg-srv3  10.129.0.23:8301  alive   server  1.16.1  2         dc1  default    <all>
    ubuntu@pg-srv1:~$ 
    ```

  * А данной командой можно увидеть дополнительную информацию
    ```bash
    consul members -detailed
    ```
    ```console
    ubuntu@pg-srv3:~$ consul members -detailed
    Node     Address           Status  Tags
    pg-srv1  10.129.0.21:8301  alive   acls=0,ap=default,build=1.16.1:e0ab4d29,dc=dc1,expect=3,ft_fs=1,ft_si=1,grpc_tls_port=8503,id=98445677-186f-8ab3-fd11-0ab2979f799c,port=8300,raft_vsn=3,role=consul,segment=<all>,vsn=2,vsn_max=3,vsn_min=2,wan_join_port=8302
    pg-srv2  10.129.0.22:8301  alive   acls=0,ap=default,build=1.16.1:e0ab4d29,dc=dc1,expect=3,ft_fs=1,ft_si=1,grpc_tls_port=8503,id=8d19d22b-016a-e803-9be4-d71b8afb27b8,port=8300,raft_vsn=3,role=consul,segment=<all>,vsn=2,vsn_max=3,vsn_min=2,wan_join_port=8302
    pg-srv3  10.129.0.23:8301  alive   acls=0,ap=default,build=1.16.1:e0ab4d29,dc=dc1,expect=3,ft_fs=1,ft_si=1,grpc_tls_port=8503,id=e325f19b-c64b-54e1-b64e-267dcc1f7984,port=8300,raft_vsn=3,role=consul,segment=<all>,vsn=2,vsn_max=3,vsn_min=2,wan_join_port=8302
    ubuntu@pg-srv3:~$ 
    ```

  * Также у нас должен быть доступен веб-интерфейс по адресу:
    ```bash
    http://<IP-адрес любого сервера консул>:8500  
    ```
  * [Все порты `Consul`](https://developer.hashicorp.com/consul/docs/agent/config/config-files)
     

***

### Какие возможности у нас появились на данном этапе
  | :electric_plug: Появилось на этом этапе | :electric_plug: Компонент | :hammer_and_wrench: Название ВМ | :link: ip-адрес | :magnet: порт | :memo: Описание возможности |
  |:--:|:---|:--------|:----------|------:|:------------------------------------------|
  | :heavy_check_mark: | `Consul` | **`pg-srv1`** | `10.129.0.21` | 8500 | Web GUI |
  | :heavy_check_mark: | `Consul` | **`pg-srv2`** | `10.129.0.22` | 8500 | Web GUI |
  | :heavy_check_mark: | `Consul` | **`pg-srv3`** | `10.129.0.23` | 8500 | Web GUI |

***
### :+1: `Consul` установлен и настроен
