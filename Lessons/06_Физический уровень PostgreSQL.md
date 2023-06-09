## **Физический уровень PostgreSQL** ##


#### *Цели занятия:*
* поговорить об устройстве PostgreSQL;
* рассмотреть процессы PostgreSQL и структуру памяти;
* обсудить как PostgreSQL работает с данными на физическом уровне.


#### *Краткое содержание:*
* работа с PostgreSQL;
* серверные процессы и память;
* физическая структура данных;
* практика.


#### *Результаты:*
1. конспект занятия:
   * чтобы не удалить что-нибудь нужное в файловой системе;
   * чтобы не убить случайно нужный процесс и убить ненужный.


#### *Преподаватель:*
  * Михаил Ржевский


#### *Компетенции:*
1. владение базовыми навыками работы в SQL
   * понимание работы PostgreSQL на физическом уровне

#### *Дата и время:*
* 20 апреля, четверг в 20:00
* Длительность занятия: 90 минут


#### *Материалы:*
* [pspg. Часть 1 — Толмачев Павел Владимирович](https://ptolmachev.ru/pspg-chast-1.html)
* [How To Partition and Format Storage Devices in Linux | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
* [Как использовать pspg](https://pgconf.ru/2021/288291)
* [06 PG physical.txt](https://cdn.otus.ru/media/private/c4/e7/06_PG_physical-25239-c4e7c5.txt?hash=kQS4xH5FfVA1xsvhCFF6xA&expires=1682217441)
* [06 PG physical.txt](https://cdn.otus.ru/media/private/e8/86/06_PG_physical-25239-e88644.txt?hash=HRaEm1g2zWddPRkIUgCsJA&expires=1682217441)
* [06 PG Физический уровень.pdf](https://cdn.otus.ru/media/private/ee/71/06_PG_%D0%A4%D0%B8%D0%B7%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D1%83%D1%80%D0%BE%D0%B2%D0%B5%D0%BD%D1%8C-25239-ee7173.pdf?hash=H1B63P9QIRTk9_cB9OfNDg&expires=1682217441)
* [chat.txt](https://cdn.otus.ru/media/public/78/2d/chat-15865-782dc1.txt)


#### *Домашнее задание:*
Установка и настройка PostgreSQL
* Цель:
    * создавать дополнительный диск для уже существующей виртуальной машины, размечать его и делать на нем файловую систему
    * переносить содержимое базы данных PostgreSQL на дополнительный диск
    * переносить содержимое БД PostgreSQL между виртуальными машинами


#### *Описание/Пошаговая инструкция выполнения домашнего задания:*
* создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
* поставьте на нее PostgreSQL 15 через sudo apt
* проверьте что кластер запущен через sudo -u postgres pg_lsclusters
* зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
  * postgres=# create table test(c1 text);
  * postgres=# insert into test values('1');
  * \q
* остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
* создайте новый диск к ВМ размером 10GB
* добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
* проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
* перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
* сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
* перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
* попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* напишите получилось или нет и почему
* задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
* напишите что и почему поменяли
* попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* напишите получилось или нет и почему
* зайдите через через psql и проверьте содержимое ранее созданной таблицы
* задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
