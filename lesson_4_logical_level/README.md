1-2) Зашла в созданный кластер под пользователем postgres<br/>
3. Coздаем новую базу данных : CREATE DATABASE testdb;<br/>
4. Переходим в новую базу: \c testdb<br/>
5. Создаем схему: CREATE SCHEMA testnm;<br/>
   5a. Список существующих схем : \dn<br/>
   testdb=# \dn<br/>
   List of schemas<br/>
   Name  |  Owner<br/>
   --------+----------<br/>
   public | postgres<br/>
   testnm | postgres<br/>
   (2 rows)<br/>
   <br/>
6. CREATE TABLE t1(c1 integer);<br/>
7. INSERT INTO t1 values(1);<br/>
   Проверка, где создалась таблица: \dt<br/>
   testdb=# \dt<br/>
   List of relations<br/>
   Schema | Name | Type  |  Owner<br/>
   --------+------+-------+----------<br/>
   public | t1   | table | postgres<br/>
   (1 row)<br/>
   <br/>
Видим, что таблица в схеме public, поэтому удаляем ее с создаем снова в схеме testnm:<br/>
CREATE TABLE testnm.t1(c1 integer);<br/>
INSERT INTO testnm.t1 values(1)<br/>
   <br/>
8. CREATE role readonly;<br/>
9. grant connect on DATABASE testdb TO readonly;<br/>
10. grant usage on SCHEMA testnm to readonly;<br/>
11. grant SELECT on all TABLEs in SCHEMA testnm TO readonly;<br/>
12. CREATE USER testread with password 'test123';<br/>
13. grant readonly TO testread;<br/>
14. Логинимся в базу под юзером testread: \c testdb testread<br/>
15. SELECT * FROM t1;<br/>
    Ошибка, так как текущая схема public и в ней этой таблицы<br/>
    testdb=# select current_schema();<br/>
    current_schema<br/>
----------------<br/>
public<br/>
(1 row)<br/>
Поэтому выполняем запрос<br/>
SELECT * FROM testnm.t1<br/>
testdb=> select * from testnm.t1;<br/>
c1<br/>
----<br/>
1<br/>
(1 row)<br/>
    <br/>
**Вывод : всегда указывать схему при создании таблицы и работы с ней ( insert/update/delete)**<br/>  
16.Команды create table t2(c1 integer); insert into t2 values (2) выполнились так как таблица создалась в схеме public, которая добавляется всем новым пользователям.<br/>
Каждый пользователь может по умолчанию создавать объекты в схеме public любой базы данных если есть право на подключение к ней.<br/>
Чтобы раз и навсегда забыть про роль public:<br/>
testdb=> \c testdb postgres;<br/>
Password for user postgres:<br/>
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
You are now connected to database "testdb" as user "postgres".<br/>
testdb=# revoke CREATE on SCHEMA public FROM public;<br/>
REVOKE<br/>
testdb=# revoke all on DATABASE testdb FROM public;<br/>
REVOKE<br/>
testdb=# \c testdb1 testread;<br/>
Password for user testread:<br/>
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
You are now connected to database "testdb" as user "testread".<br/>
testdb=> create table t3(c1 int);<br/>
ERROR:  permission denied for schema public<br/>
LINE 1: create table t3(c1 int);<br/>
^<br/>
testdb=><br/>
**Вывод : На продакшине надо запретить любому пользователю создавать объекты в схеме public**<br/>
