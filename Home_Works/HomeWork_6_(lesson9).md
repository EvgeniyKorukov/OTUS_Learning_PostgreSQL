#### *Отчет о выполнении домашнего задания:*


> Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
  * **_Меняем параметры СУБД Postgres_** 
    * Включаем логирование блокировок логах Postgres
      ```sql
      alter system set log_lock_waits=on; 
      ```
    * Включаем логирование блокировок, которые удерживаются более 200 миллисекунд
      ```sql 
      alter system set deadlock_timeout=200;
      ```
    * Применяем параметры к Postgres. 
      ```sql
      select pg_reload_conf();
      ```
    * Данные из лога Postgres, где видно срабатывание. Данные от задания ниже.
      ```
      2023-05-10 22:04:22.428 UTC [4493] postgres@locks LOG:  process 4493 still waiting for ShareLock on transaction 31723 after 200.512 ms
      2023-05-10 22:04:22.428 UTC [4493] postgres@locks DETAIL:  Process holding the lock: 4490. Wait queue: 4493.
      2023-05-10 22:04:22.428 UTC [4493] postgres@locks CONTEXT:  while updating tuple (0,10) in relation "accounts"
      2023-05-10 22:04:22.428 UTC [4493] postgres@locks STATEMENT:  UPDATE accounts SET amount = amount + 380.00 WHERE acc_no = 1; 
      ```


> Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
  * **_Session 1_** 
      ```sql
      BEGIN;
      SELECT txid_current(), pg_backend_pid(); --Session_1: 31725 : 4490
      UPDATE accounts 
      SET amount = amount + 100.00 WHERE acc_no = 1; 
      ```    
  * **_Session 2_** 
      ```sql
      BEGIN;
      SELECT txid_current(), pg_backend_pid(); --Session_2: 31726 : 4492
      UPDATE accounts SET amount = amount - 50.00 WHERE acc_no = 1;
      ```
  * **_Session 3_** 
      ```sql
      BEGIN;
      SELECT txid_current(), pg_backend_pid(); --Session_3: 31727 : 4493
      UPDATE accounts SET amount = amount + 3800.00 WHERE acc_no = 1;
      ```
  * **_Session 1 all locks_** 
      ```sql
      SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
        FROM pg_locks 
        WHERE pid = 4490;
      
         locktype    |   relation    | virtxid |  xid  |       mode       | granted 
      ---------------+---------------+---------+-------+------------------+---------
       relation      | pg_locks      |         |       | AccessShareLock  | t
       relation      | accounts_pkey |         |       | RowExclusiveLock | t
       relation      | accounts      |         |       | RowExclusiveLock | t
       virtualxid    |               | 7/391   |       | ExclusiveLock    | t
       transactionid |               |         | 31725 | ExclusiveLock    | t
      (5 rows)
      ```
      * Блокировки первой сессии с pid = 4490
        * AccessShareLock на таблице pg_locks и уже полученная блокировка granted=t. Обычная блокировка при SELECT из таблицы.
        * RowExclusiveLock на таблице accounts и ее первичном ключе accounts_pkey. На первичном ключе (primary key) всегда есть индекс. Эта блокировка операции UPDATE.
        * ExclusiveLock на виртуальном идентификаторе транзакции (virtualxid), которая реально не меняет данные (virtxid=7/391). 
        * ExclusiveLock исключительная блокировка настоящего номера транзакции (transactionid), которая начала менять данные (xid=31725).

     
      ```sql      
      SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
        FROM pg_locks 
        WHERE pid = 4492;
        
         locktype    |   relation    | virtxid |  xid  |       mode       | granted 
      ---------------+---------------+---------+-------+------------------+---------
       relation      | accounts_pkey |         |       | RowExclusiveLock | t
       relation      | accounts      |         |       | RowExclusiveLock | t
       virtualxid    |               | 6/24    |       | ExclusiveLock    | t
       transactionid |               |         | 31725 | ShareLock        | f
       tuple         | accounts      |         |       | ExclusiveLock    | t
       transactionid |               |         | 31726 | ExclusiveLock    | t
      (6 rows)
      ```   

      * Блокировки второй сессии с pid = 4492
        * RowExclusiveLock на таблице accounts и ее первичном ключе accounts_pkey. На первичном ключе (primary key) всегда есть индекс. Эта блокировка операции UPDATE.
        * ExclusiveLock на виртуальном идентификаторе транзакции (virtualxid), которая реально не меняет данные (virtxid=6/24). 
        * ShareLock. Транзакция пытается наложить блокировку ShareLock на xid=31725, но не может granted=f  
        * ExclusiveLock (tuple) возникает, когда несколько сессий пытаются изменить одни и те же данные. Чтобы не ссылаться на одни и те же идентификаторы транзакций, формируется некий общий tuple с блокровкой на таблицу.
        * ExclusiveLock исключительная блокировка настоящего номера транзакции (transactionid), которая начала менять данные (xid=31726).

      ```sql
      SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
        FROM pg_locks 
        WHERE pid = 4493;
        
         locktype    |   relation    | virtxid |  xid  |       mode       | granted 
      ---------------+---------------+---------+-------+------------------+---------
       relation      | accounts_pkey |         |       | RowExclusiveLock | t
       relation      | accounts      |         |       | RowExclusiveLock | t
       virtualxid    |               | 5/11    |       | ExclusiveLock    | t
       tuple         | accounts      |         |       | ExclusiveLock    | f
       transactionid |               |         | 31727 | ExclusiveLock    | t
      (5 rows)
      ```   
      * Блокировки третьей сессии с pid = 4493
        * RowExclusiveLock на таблице accounts и ее первичном ключе accounts_pkey. На первичном ключе (primary key) всегда есть индекс. Эта блокировка операции UPDATE.
        * ExclusiveLock на виртуальном идентификаторе транзакции (virtualxid), которая реально не меняет данные (virtxid=5/11).
        * ExclusiveLock (tuple) возникает, когда несколько сессий пытаются изменить одни и те же данные. Чтобы не ссылаться на одни и те же идентификаторы транзакций, формируется некий общий tuple с блокровкой на таблицу. В данном случае не получилось наложить блокировку на таблицу (granted=f) 
        * ExclusiveLock исключительная блокировка настоящего номера транзакции (transactionid), которая начала менять данные (xid=31727).


> Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
  * **_Раз_** 
    * Два


> Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?


> Задание со звездочкой*
> Попробуйте воспроизвести такую ситуацию
  * **_Раз_** 
    * Два



