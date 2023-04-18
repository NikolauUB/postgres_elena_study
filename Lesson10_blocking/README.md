sudo pg_ctlcluster 14 main start<br/>
psql -U  postgres -h localhost -p 5432<br/>
1. Настройка сервера так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд<br/>
postgres=# show log_lock_waits;<br/>
log_lock_waits<br/>
----------------<br/>
on<br/>
(1 row)<br/>
<br/>
postgres=#<br/>
SHOW deadlock_timeout;<br/>
Result :1 sec<br/>
<br/>
ALTER SYSTEM SET deadlock_timeout = '0.2s';<br/>
SELECT pg_reload_conf();<br/>
postgres=# SHOW deadlock_timeout;<br/>
deadlock_timeout<br/>
------------------<br/>
200ms<br/>
(1 row)<br/>
<br/>
postgres=#<br/>
<br/>
<img src="./log_file.PNG"/>
2. Обновления одной и той же строки тремя командами UPDATE в разных сеансах<br/>
--session 1<br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;<br/>
<br/>
--session 2<br/>
UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;<br/>
<br/>
# Открываем новое окно терминала<br/>
tail -n 10 /var/log/postgresql/postgresql-14-main.log<br/>
<br/>
2)Обновления одной и той же строки тремя командами UPDATE в разных сеансах<br/>
<br/>
--Сессия 1<br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
 txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778906       |328<br/>
(1 row)<br/>
<br/>
lock=*# UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;<br/>
UPDATE 1<br/>
lock=*# SELECT * FROM locks_v WHERE pid = 328;<br/>
 pid |   locktype    |  lockid  |mode| granted<br/>
-----+---------------+----------+------------------+---------<br/>
 328 | relation      | accounts | RowExclusiveLock | t<br/>
 328 | transactionid | 7778906  | ExclusiveLock    | t<br/>
(2 rows)<br/>
<br/>
lock=*#<br/>
<br/>
--Сессия 2<br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
 txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778907       |341<br/>
(1 row)<br/>
<br/>
--Сессия 3 <br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
 txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778908       |331<br/>
(1 row)<br/>
<br/>
lock=*#<br/>
<br/>
Запустили апдейт в сессии 2 и смотрим , что произошло для этого процесса<br/>
<br/>
lock=*# SELECT * FROM locks_v WHERE pid = 341;<br/>
 pid |   locktype    |   lockid   |mode| granted<br/>
-----+---------------+------------+------------------+---------<br/>
 341 | relation      | accounts   | RowExclusiveLock | t<br/>
 341 | transactionid | 7778907    | ExclusiveLock    | t<br/>
 341 | tuple         | accounts:9 | ExclusiveLock    | t<br/>
 341 | transactionid | 7778906    | ShareLock        | f<br/>
 (4 rows)<br/>
 <br/>
Запустили апдейт в сессии 3 и смотрим , что произошло для этого процесса<br/>
<br/>
lock=*# SELECT * FROM locks_v WHERE pid = 331;<br/>
pid |   locktype    |   lockid   |mode| granted<br/>
-----+---------------+------------+------------------+---------<br/>
331 | relation      | accounts   | RowExclusiveLock | t<br/>
331 | transactionid | 7778908    | ExclusiveLock    | t<br/>
331 | tuple         | accounts:9 | ExclusiveLock    | f<br/>
(3 rows)<br/>
<br/>
lock=*#<br/>
<br/>
lock=*# SELECT pid, wait_event_type, wait_event, pg_blocking_pids(pid) FROM pg_stat_activity  WHERE backend_type = 'client backend' ORDER BY pid;<br/>
pid | wait_event_type |  wait_event   | pg_blocking_pids<br/>
-----+-----------------+---------------+------------------<br/>
328 |     |               |  {}<br/>
331 | Lock| tuple         | {341}<br/>
341 | Lock| transactionid | {328}<br/>
(3 rows)<br/>
<br/>
lock=*#<br/>
<br/>
Итог:<br/>
после апдейта в первой сессии   (pid 328)- наложили блокировку<br/>
во второй сессии (pid = 341)  - появилась блокировка shareLock c параметром false, и появилась запись tuple ( строка заблокирована)<br/>
сессия 3 (pid = 331) - ссылается на tuple, доступ закрыт, не может сделать ExclusiveLock.<br/>
<br/>
Информация о блокирующих процессах:<br/>
есть транзакция без блокировки<br/>
341 блокируется 328 процессом<br/>
Дальше идет tuple, который блокируется 341-ым<br/>
<br/>
<br/>
3)Взаимоблокировка трех транзакций<br/>
 step 1  сессия 1  UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;<br/>
 step 2  сессия 2  UPDATE accounts SET amount = amount - 10.00 WHERE acc_no = 2;<br/>
 step 3  сессия 3  UPDATE accounts SET amount = amount - 30.00 WHERE acc_no = 3;<br/>
 step 4  сессия 1  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 3;<br/>
 step 5  сессия 3  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;<br/>
 step 6  сессия 2  UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 1;<br/>
 <br/> 
-- Сессия  #1<br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778914      |328<br/>
(1 row)<br/>
<br/>
lock=*# UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;<br/>
UPDATE 1<br/>
lock=*# UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 3;<br/>
<br/>
<br/>
-- Сессия  #2<br/>
lock=# BEGIN;<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778915 |331<br/>
(1 row)<br/>
<br/>
lock=*# UPDATE accounts SET amount = amount - 10.00 WHERE acc_no = 2;<br/>
UPDATE 1<br/>
lock=*# UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 1;<br/>
ERROR:  deadlock detected<br/>
DETAIL:  Process 331 waits for ShareLock on transaction 7778914; blocked by process 328.<br/>
Process 328 waits for ShareLock on transaction 7778916; blocked by process 341.<br/>
Process 341 waits for ShareLock on transaction 7778915; blocked by process 331.<br/>
HINT:  See server log for query details.<br/>
CONTEXT:  while updating tuple (0,9) in relation "accounts"<br/>
lock=!#<br/>
<br/>
-- Сессия #3<br/>
BEGIN<br/>
lock=*# SELECT txid_current(), pg_backend_pid();<br/>
txid_current | pg_backend_pid<br/>
--------------+----------------<br/>
7778916 |341<br/>
(1 row)<br/>
<br/>
lock=*# UPDATE accounts SET amount = amount - 30.00 WHERE acc_no = 3;<br/>
UPDATE 1<br/>
lock=*# UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;<br/>
UPDATE 1<br/>
lock=*#<br/>
<br/>
<img src="./deadlock.PNG"/>
4) Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?<br/>
Теоритически могут, но получить не удалось<br/>
SELECT FOR UPDATE временно блокирует другие транзакции<br/>
Блокировка, получаемая транзакцией Repeatable Read, гарантирует, что никакая другая транзакция, изменяющая таблицу, не выполняется<br/>
Был сделан тест:<br/>
Создаем таблицу<br/>
CREATE TABLE t(id INT GENERATED ALWAYS AS IDENTITY, n float);<br/>
INSERT INTO t(n) select random() from generate_series( 1,1000000);<br/>
<br/>
--Сессия 1<br/>
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;<br/>
UPDATE t set n = ( select id from t order by ID  LIMIT 1 for update);<br/>
<br/>
--Сессия 2<br/>
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;<br/>
UPDATE t set n = ( select id from t order by ID  LIMIT 1 for update);<br/>
<br/>
В первой сессии апдейт прошел , во второй - ждал блокировку<br/>
как только сделала commit в Сессии 1, во второй появилось сообщение:<br/>
ERROR:  could not serialize access due to concurrent update<br/>
<br/>