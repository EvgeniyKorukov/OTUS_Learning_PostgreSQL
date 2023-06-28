<div align="center"><h2> Отчет о выполнении домашнего задания по теме: "Секционирование " </h2></div>

***

> ### Секционировать большую таблицу из демо базы flights
  * Работаем с ВМ в YandexCloud
    :hammer_and_wrench: Параметр | :memo: Значение |:hammer_and_wrench: Параметр | :memo: Значение |
    --------------:|---------------|--------------:|---------------|
    | Название ВМ | **`pg-srv`** | Гарантированная доля vCPU | `100%` | 
    | Зона доступности | `ru-central1-b` | Внутренний IPv4 | `10.129.0.21` | 
    | Операционная система | `Ubuntu 20.04 LTS` | Публичный IPv4 | `0.0.0.0` |
    | Платформа | `Intel Cascade Lake	` | Тип диска | `SSD` | 
    | vCPU | `2` | Объём дискового пространства | `50 ГБ` |
    | RAM | `8 ГБ` | Макс. IOPS (чтение / запись) | `1000 / 1000` |
    | Прерываемая | :ballot_box_with_check: | Макс. bandwidth (чтение / запись) | `15 МБ/с / 15 МБ/с` |

    ```console
    eugink@nb-xiaomi ~ $ yc compute instance create \
      --name pg-srv \
      --hostname pg-srv \
      --cores 2 \
      --memory 8 \
      --create-boot-disk size=50G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2004-lts \
      --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4,ipv4-address=10.129.0.21 \
      --zone ru-central1-b \
      --core-fraction 100 \
      --preemptible \
      --metadata-from-file ssh-keys=/home/eugink/.ssh/eugin_yandex_key.pub
       ```
    
  * Устанавливаем PostgreSQL15
  * Загружаем большую БД с полетами за год [demo-big](https://edu.postgrespro.ru/demo-big.zip)


  * Подключаемся к ВМ 
    ```console
    eugink@nb-xiaomi ~ $ ssh ubuntu@0.0.0.0
    ubuntu@pg-srv1:~$
    ```
