# postgres_elena_study
выключить auto commit
\set AUTOCOMMIT OFF
сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); 
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;
посмотреть текущий уровень изоляции: show transaction isolation level;
read committed
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
1) в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
2) сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
<br/>**2 строки, новой записи не видно, так как изменения в первой сессии не закомичены**

3) завершить первую транзакцию - commit;
4) сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
<br/>**3 строки , новая запись выбралась, так как первая транзакция закомичена. Все данные из таблицы выбираются**

4 )завершите транзакцию во второй сессии
5) начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
6) в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
<br/>**3 строки, новая запись не выбралась, так как открытая транзакция читает только те данные, которые были в таблице до ее открытия**

7) завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
<br/>**3 строки, новая запись не выбралась, так как открытая транзакция читает только те данные, которые были в таблице до ее открытия**

8) завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
<br/>**4 строки, новая запись выбралась, так как обе транзакции закрыты**
