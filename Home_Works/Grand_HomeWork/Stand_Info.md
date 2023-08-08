
<div align="center"><h2> Описание стенда и функциональности </h2></div>

***

> ### 1. Создаем 3 ВМ для Patroni+Consul+PGBouncer в YandexCloud.
   * Общие параметры для всех 3х ВМ для Patroni+Consul+PGBouncer с фиксированным внутренним IPv4.
       :hammer_and_wrench: Параметр | :memo: Значение |
      --------------:|---------------| 
      | Операционная система | `Ubuntu 20.04 LTS` |
      | Зона доступности | `ru-central1-b` |
      | Платформа | `Intel Cascade Lake	` |
      | vCPU | `2` |
      | Гарантированная доля vCPU | `20%` |
      | RAM | `4 ГБ` |
      | Тип диска | `SSD` | 
      | Объём дискового пространства | `10 ГБ` |
      | Макс. IOPS (чтение / запись) | `1000 / 1000` |
      | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |
      | Прерываемая | :ballot_box_with_check: |
        
   * Индивидуальные параметры каждой ВМ
      :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
      --------------:|---------------|
      | **`pg-srv1`** | `10.129.0.21` |
      | **`pg-srv2`** | `10.129.0.22` |      
      | **`pg-srv3`** | `10.129.0.23` |

   * На каждой ВМ устанавливаем PostgreSQL 15 и проверяем, что экземпляры запустились (вывод установки приводить не стал т.к. в этом нет особого смысла)
         ```bash
         sudo apt update && sudo apt upgrade -y -q && echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee -a /etc/apt/sources.list.d/pgdg.list && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15

         pg_lsclusters
         ```
       * Вывод pg_lsclusters с каждой ВМ:
         ```console
         ubuntu@pg-srv1:~$ hostname; pg_lsclusters
         pg-srv1
         Ver Cluster Port Status Owner    Data directory              Log file
         15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
         ubuntu@pg-srv1:~$        
         ```
         ```console
         ubuntu@pg-srv2:~$ hostname; pg_lsclusters
         pg-srv2
         Ver Cluster Port Status Owner    Data directory              Log file
         15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
         ubuntu@pg-srv2:~$ 
         ```
         ```console
         ubuntu@pg-srv3:~$ pg_lsclusters
         Ver Cluster Port Status Owner    Data directory              Log file
         15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
         ubuntu@pg-srv3:~$ 
         ```  


> ### 2. Создаем 2 ВМ для HAProxy+KeepAlived в YandexCloud.
  * Общие параметры для всех 2х ВМ для HAProxy+KeepAlived с фиксированным внутренним IPv4. Команды по созданию ВМ прилагать не буду т.к. они типовые
       :hammer_and_wrench: Параметр | :memo: Значение |
      --------------:|---------------| 
      | Операционная система | `Ubuntu 20.04 LTS` |
      | Зона доступности | `ru-central1-b` |
      | Платформа | `Intel Cascade Lake	` |
      | vCPU | `2` |
      | Гарантированная доля vCPU | `20%` |
      | RAM | `2 ГБ` |
      | Тип диска | `SSD` | 
      | Объём дискового пространства | `5 ГБ` |
      | Макс. IOPS (чтение / запись) | `1000 / 1000` |
      | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |
      | Прерываемая | :ballot_box_with_check: |
  * Индивидуальные параметры каждой ВМ
      :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
      --------------:|---------------|
      | **`hap1`** | `10.129.0.11` |
      | **`hap2`** | `10.129.0.12` |      

***
* На каждой ВМ устанавливаем etcd и останавливаем etcd, чтобы изменить конфигурацию
       ```bash
       sudo apt update && sudo apt upgrade -y && sudo apt install -y etcd
       sudo systemctl stop etcd
       ```
     * На каждой ВМ с etcd:
       * Формируем конфигурацию для etcd
       * Делаем сервис etcd автозапускаемым
       * Запускаем etcd
       * **❗️используем FQDM, полное имя хоста (сервера) получаем командой `hostname -f`**
         ```bash
         cat > temp.cfg << EOF 
         ETCD_NAME="$(hostname)"
         ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
         ETCD_ADVERTISE_CLIENT_URLS="http://$(hostname -f):2379"
         ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
         ETCD_INITIAL_ADVERTISE_PEER_URLS="http://$(hostname -f):2380"
         ETCD_INITIAL_CLUSTER_TOKEN="PatroniCluster"
         ETCD_INITIAL_CLUSTER="etcd1=http://etcd1.ru-central1.internal:2380,etcd2=http://etcd2.ru-central1.internal:2380,etcd3=http://etcd3.ru-central1.internal:2380"
         ETCD_INITIAL_CLUSTER_STATE="new"
         ETCD_DATA_DIR="/var/lib/etcd"
         EOF
         cat temp.cfg | sudo tee -a /etc/default/etcd

         sudo systemctl enable etcd

         sudo systemctl start etcd
         ```
     * Проверяем статус etcd на одной из ВМ
       ```console
       ubuntu@etcd1:~$ etcdctl cluster-health
       member 61f991406239b07 is healthy: got healthy result from http://etcd1.ru-central1.internal:2379
       member 3376d9633be63daf is healthy: got healthy result from http://etcd3.ru-central1.internal:2379
       member 64b82de471857189 is healthy: got healthy result from http://etcd2.ru-central1.internal:2379
       cluster is healthy
       ubuntu@etcd1:~$ 
       ```
  * Создаем 3 ВМ для PostgreSQL 15+Patroni в YandexCloud
