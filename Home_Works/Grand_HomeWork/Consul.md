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



