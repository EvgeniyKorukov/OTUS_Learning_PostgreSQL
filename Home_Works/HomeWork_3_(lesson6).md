#### *Отчет о выполнении домашнего задания:*


> создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
* **_Установлен Ubuntu server 22.04.2 на виртуальной машине в VirtualBox_**


> поставьте на нее PostgreSQL 15 через sudo apt
* **_Установлен Postgres 15_**  
    * sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15
	
	
> проверьте что кластер запущен через sudo -u postgres pg_lsclusters
* **_sudo -u postgres pg_lsclusters_**  
    * Ver Cluster Port Status Owner    Data directory              Log file
    * 15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
	
	
> зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
* **_sudo -u postgres psql -U postgres_**  
    * create table test(c1 text);
    	* postgres=# create table test(c1 text);
    	* CREATE TABLE
    * insert into test values('1');
    	* postgres=# insert into test values('1');
    	* INSERT 0 1
    * \q
	
	
> остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
* **_Остановка Postgres_**  
    * sudo -u postgres pg_ctlcluster 15 main stop
    * sudo -u postgres pg_lsclusters
    	* Ver Cluster Port Status Owner    Data directory              Log file
    	* 15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

	
> создайте новый диск к ВМ размером 10GB
* **_Создан новый диск в VirtualBox размером 10Gb_**  

	
> добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
* **_Новый диск подключен к виртуальной машине с PostgreSQL_**  

	
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
