<div align="center"><h2> 3. Настройка Patroni </h2></div>

***

> ### Немного теории:
  * Для работы в режиме высокой доступности `HA` (High Availability) нужно некоторое ПО, которое контроллирует кто сейчас `master` или `главный сервер`, а кто `secondary` или `реплика`
  * В PostgreSQL это очень важно чтобы не было `Split Brain` чтобы в кластере не появилось больше одного `master`
  * Подобные функции может выполнять следующее ПО:
    * [`Pacemaker and Corosync`](https://clusterlabs.org/)
    * [`Patroni`](https://patroni.readthedocs.io/en/latest/) ❗️Именно его мы и будет использовать
    * [`RepMgr`](https://www.repmgr.org/)
    * [`Stolon`](https://github.com/sorintlab/stolon)
    * [`Slony`](https://www.slony.info/)
   
*** 

