#### *Отчет о выполнении домашнего задания:*


> создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
* **_Установлен Ubuntu server 22.04.2 на виртуальной машине в VirtualBox_**  

> поставить на нем Docker Engine
* **_Установлен Docker Engine_**  
    * curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER

> сделать каталог /var/lib/postgres
* **_Создаем каталог_**  
	* sudo mkdir -p /var/lib/postgres

> развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
* **_Разворачиваем контейнер с PostgreSQL 15_**  
   * sudo docker network create pg-net
   * sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15


> развернуть контейнер с клиентом postgres 
> подключится из контейнера с клиентом к контейнеру с сервером
* **_Разворачиваем контейнер с клиентом и запускаем совместно с контейнером с СУБД_**  
   * sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres


> и сделать таблицу с парой строк
* **_Делаем таблицу с данными_**  
	* create table persons(id serial, first_name text, second_name text); 
   * insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
   * insert into persons(first_name, second_name) values('petr', 'petrov'); 
   * commit;

> подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
* **_Подключаемся к контейнеру с сервером с ноутбука_**  
	* Нет клиентского ПО PostgreSQL 15, поэтому его надо установить
		* sudo dnf install postgresql15
	* Подключение к контейнеру с PostgreSQL 15 с ноутбука
		* psql -p 5432 -U postgres -h 192.168.143.195 -d postgres -W
	* Считаем количество данных в таблицу или выводим ее содержимое
		* psql -p 5432 -U postgres -h 192.168.143.195 -d postgres -W -c "select count(*) from persons"
		* psql -p 5432 -U postgres -h 192.168.143.195 -d postgres -W -c "select * from persons"
	
> удалить контейнер с сервером
* **_Удаляем контейнер с сервером_**  
	* Получаем список контейнеров, чтобы получить container id
		* sudo docker ps -a
	* Удаляем контейнер
		* sudo docker rm 6c986aa54227 -f
	
> создать контейнер заново
* **_создаем контейнер с PostgreSQL 15 заново_**  
   * sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15

> подключится снова из контейнера с клиентом к контейнеру с сервером
* **_Подключаемся к контейнеру с клиентом и запускаем совместно с контейнером с СУБД_**  
   * sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres

> проверить, что данные остались на месте
* **_Проверяем данные, после персоздания контейнера с PostgreSQL 15_**  
	* Считаем количество данных в таблицу или выводим ее содержимое
		* select count(*) from persons;
		* select * from persons;
* **_Данные на месте и они никуда не пропали т.к. мы делали мапинг данных с контейнера на локальный сервер т.е. при удалении контейнера-данные с сервера не удаляются_**  

    
<kbd>
  <img src="/Images/Docker-dark.jpg" />
</kbd>
