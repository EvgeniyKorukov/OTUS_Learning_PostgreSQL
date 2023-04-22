#### *Отчет о выполнении домашнего задания:*


> создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
* **_Установлен Ubuntu server 22.04.2 на виртуальной машине в VirtualBox_**
	
> поставьте на нее PostgreSQL 15 через sudo apt
* **_Установлен Postgres 15_**  
    * sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
	
> проверьте что кластер запущен через sudo -u postgres pg_lsclusters
* **_Title_**  
    * Text1
	
> зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
* **_Title_**  
    * Text1
	
> postgres=# create table test(c1 text);
* **_Title_**  
    * Text1
	
> postgres=# insert into test values('1');
* **_Title_**  
    * Text1
	
> \q
* **_Title_**  
    * Text1
	
> остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
* **_Title_**  
    * Text1
	
> создайте новый диск к ВМ размером 10GB
* **_Title_**  
    * Text1
	
> добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
* **_Title_**  
    * Text1
	
> проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
* **_Title_**  
    * Text1
	
> перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
* **_Title_**  
    * Text1
	
> сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
* **_Title_**  
    * Text1
	
> перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
* **_Title_**  
    * Text1
	
> попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* **_Title_**  
    * Text1
	
> напишите получилось или нет и почему
* **_Title_**  
    * Text1
	
> задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
* **_Title_**  
    * Text1
	
> напишите что и почему поменяли
* **_Title_**  
    * Text1
	
> попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
* **_Title_**  
    * Text1
	
> напишите получилось или нет и почему
* **_Title_**  
    * Text1
	
> зайдите через через psql и проверьте содержимое ранее созданной таблицы
* **_Title_**  
    * Text1
	
> задание со звездочкой >: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
* **_Title_**  
    * Text1
