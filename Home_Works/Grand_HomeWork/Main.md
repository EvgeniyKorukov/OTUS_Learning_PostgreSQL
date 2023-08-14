<div align="center"><h2> Проект "Создание отказоустойчивого кластера на базе: Patroni+Consul+PGBouncer+HAProxy+KeepAlived" </h2></div>

***

### 1. [Подготовка, настройка и описание стенда](Stand_Info.md)
### 2. [Установка и настройка Consul](Consul.md)
### 3. [Установка и настройка Patroni](Patroni.md)
### 4. [Установка и настройка PgBouncer](PGBouncer.md)
### 5. [Установка и настройка KeepAlived](KeepAlived.md)
### -6. [Установка и настройка HA Proxy](HAProxy.md)
### -7. Проверка отказоусточивости при switchover и failsafe в patroni
### -8. Проверка отказоусточивости при падении HA Proxy и переносе VIP



***
### ? Установка и настройка patroni (желательно преобразовать имеющуюся СУБД в patroni, логи выбора мастера)
https://patroni.readthedocs.io/en/latest/existing_data.html
### ? Резервное копирование с реплики
***
### Мои замечания:
  * keepalive
    * Не работает VRRP в YandexCloud. Другими словами, но vip manager, ни keepalived там не запустить....
    * https://telq.org/question/62551949b2d5debe9e6d1778
    * https://github.com/yandex-cloud/yc-architect-solution-library/blob/main/demos/dmz-fw-ha/README.md
    * В облачной сети Yandex Cloud не поддерживается работа протоколов VRRP/HSRP между FW.

  * pgbouncer
    * Может надо чтобы каждый сервер с pgbouncer смотрел на все ip серверов с СУБД ?
