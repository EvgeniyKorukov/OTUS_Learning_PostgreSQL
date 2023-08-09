
<div align="center"><h2> 1. Подготовка, настройка и описание стенда </h2></div>

***

### В идеале, надо создавать группы ВМ для каждого компонента:
  * 3 ВМ с Patroni
  * 3 ВМ с Consul
  * 3 ВМ с PGBouncer
  * 2 ВМ с HAProxy+KeepAlived

***

### ❗️Но у нас бюджетный проект и мы ограничены ресурсами и имеем всего 5 серверов в виде ВМ в YandexCloud:
  * #### Создаем 3 ВМ для Patroni+Consul+PGBouncer.
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
        | Объём дискового пространства | `5 ГБ` |
        | Макс. IOPS (чтение / запись) | `1000 / 1000` |
        | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |
        | Прерываемая | :ballot_box_with_check: |
          
     * Индивидуальные параметры каждой ВМ
        :hammer_and_wrench: Название ВМ | :memo: Внутренний IPv4 |
        --------------:|---------------|
        | **`pg-srv1`** | `10.129.0.21` |
        | **`pg-srv2`** | `10.129.0.22` |      
        | **`pg-srv3`** | `10.129.0.23` |

    ```bash
    yc compute instance create \
      --name pg-srv1 \
      --hostname pg-srv1 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.21 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    ```
    ```console
    yc compute instance create \
      --name pg-srv1 \
      --hostname pg-srv1 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.21 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    done (51s)
    id: epdc3hgv218fb9c8b1l0
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-08-09T21:05:23Z"
    name: pg-srv1
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "4294967296"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epdff2veknj825kicj6j
      auto_delete: true
      disk_id: epdff2veknj825kicj6j
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:c1:c6:1f:10
        subnet_id: e2lk4cvo04hq6d33rlkt
        primary_v4_address:
          address: 10.129.0.21
          one_to_one_nat:
            address: 51.250.27.115
            ip_version: IPV4
    gpu_settings: {}
    fqdn: pg-srv1.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}
    ```
    ```bash
    yc compute instance create \
      --name pg-srv2 \
      --hostname pg-srv2 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.22 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    ```
    ```console
    yc compute instance create \
      --name pg-srv2 \
      --hostname pg-srv2 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.22 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    done (32s)
    id: epdkbdt84gtqrkv3f9du
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-08-09T21:09:16Z"
    name: pg-srv2
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "4294967296"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epd91ck7agfmiqmpq6ji
      auto_delete: true
      disk_id: epd91ck7agfmiqmpq6ji
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:14:5b:7a:82
        subnet_id: e2lk4cvo04hq6d33rlkt
        primary_v4_address:
          address: 10.129.0.22
          one_to_one_nat:
            address: 62.84.122.92
            ip_version: IPV4
    gpu_settings: {}
    fqdn: pg-srv2.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}
    ```
    ```bash
    yc compute instance create \
      --name pg-srv3 \
      --hostname pg-srv3 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.23 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    ```
    ```console
    yc compute instance create \
      --name pg-srv3 \
      --hostname pg-srv3 \
      --cores 2 \
      --memory 4 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.23 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    done (50s)
    id: epd9ei86t3ctsh0d9b8d
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-08-09T21:09:36Z"
    name: pg-srv3
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "4294967296"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epdl4vcf5nnooeckfv3e
      auto_delete: true
      disk_id: epdl4vcf5nnooeckfv3e
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:97:49:06:e8
        subnet_id: e2lk4cvo04hq6d33rlkt
        primary_v4_address:
          address: 10.129.0.23
          one_to_one_nat:
            address: 51.250.96.91
            ip_version: IPV4
    gpu_settings: {}
    fqdn: pg-srv3.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}
    ```
    * На всех 3 ВМ ставим PostgreSQL 15 и удаляем кластер по умолчанию
    ```bash
    sudo apt update && sudo apt upgrade -y -q && echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee -a /etc/apt/sources.list.d/pgdg.list && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15 && sudo -u postgres pg_dropcluster 15 main --stop && pg_lsclusters
    ```

  * #### Создаем 2 ВМ для HAProxy+KeepAlived.
    * Общие параметры для всех 2х ВМ для HAProxy+KeepAlived с фиксированным внутренним IPv4.
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

    ```bash
    yc compute instance create \
      --name hap1 \
      --hostname hap1 \
      --cores 2 \
      --memory 2 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.11 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    ```
    ```console
    yc compute instance create \
      --name hap1 \
      --hostname hap1 \
      --cores 2 \
      --memory 2 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.11 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    done (41s)
    id: epdu4ca1etl7e0v323aj
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-08-09T21:33:30Z"
    name: hap1
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "2147483648"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epd0s6libvqf818v61i1
      auto_delete: true
      disk_id: epd0s6libvqf818v61i1
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:1e:23:14:17
        subnet_id: e2lk4cvo04hq6d33rlkt
        primary_v4_address:
          address: 10.129.0.11
          one_to_one_nat:
            address: 51.250.31.223
            ip_version: IPV4
    gpu_settings: {}
    fqdn: hap1.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}
    ```
    
    ```bash
    yc compute instance create \
      --name hap2 \
      --hostname hap2 \
      --cores 2 \
      --memory 2 \
      --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.12 \
      --zone ru-central1-b \
      --core-fraction 20 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    ```
    ```console
    yc compute instance create \
          --name hap2 \
          --hostname hap2 \
          --cores 2 \
          --memory 2 \
          --create-boot-disk size=5G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
          --network-interface subnet-name=default-subnet,nat-ip-version=ipv4,ipv4-address=10.129.0.12 \
          --zone ru-central1-b \
          --core-fraction 20 \
          --preemptible \
          --metadata-from-file ssh-keys=/home/muser/.ssh/yc_key.pub
    done (47s)
    id: epddj88rneil10t018im
    folder_id: b1g59qc1dbgj9fu1qp9t
    created_at: "2023-08-09T21:34:17Z"
    name: hap2
    zone_id: ru-central1-b
    platform_id: standard-v2
    resources:
      memory: "2147483648"
      cores: "2"
      core_fraction: "20"
    status: RUNNING
    metadata_options:
      gce_http_endpoint: ENABLED
      aws_v1_http_endpoint: ENABLED
      gce_http_token: ENABLED
      aws_v1_http_token: DISABLED
    boot_disk:
      mode: READ_WRITE
      device_name: epd8tfl2r9o73nommiae
      auto_delete: true
      disk_id: epd8tfl2r9o73nommiae
    network_interfaces:
      - index: "0"
        mac_address: d0:0d:d9:a1:1b:bb
        subnet_id: e2lk4cvo04hq6d33rlkt
        primary_v4_address:
          address: 10.129.0.12
          one_to_one_nat:
            address: 51.250.23.58
            ip_version: IPV4
    gpu_settings: {}
    fqdn: hap2.ru-central1.internal
    scheduling_policy:
      preemptible: true
    network_settings:
      type: STANDARD
    placement_policy: {}
    ```

***
### В итоге получился вот такой стенд:
  ```bash
  yc compute instance list
  ```

  ```console
  yc compute instance list
  +----------------------+---------+---------------+---------+---------------+-------------+
  |          ID          |  NAME   |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
  +----------------------+---------+---------------+---------+---------------+-------------+
  | epd9ei86t3ctsh0d9b8d | pg-srv3 | ru-central1-b | RUNNING | 51.250.96.91  | 10.129.0.23 |
  | epdc3hgv218fb9c8b1l0 | pg-srv1 | ru-central1-b | RUNNING | 51.250.27.115 | 10.129.0.21 |
  | epddj88rneil10t018im | hap2    | ru-central1-b | RUNNING | 51.250.23.58  | 10.129.0.12 |
  | epdkbdt84gtqrkv3f9du | pg-srv2 | ru-central1-b | RUNNING | 62.84.122.92  | 10.129.0.22 |
  | epdu4ca1etl7e0v323aj | hap1    | ru-central1-b | RUNNING | 51.250.31.223 | 10.129.0.11 |
  +----------------------+---------+---------------+---------+---------------+-------------+
  ```

***
