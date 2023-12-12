
root@9091a9f0ca42:~# sudo -u postgres psql<br/>
psql (14.7 (Ubuntu 14.7-1.pgdg20.04+1))<br/>
Type "help" for help.<br/>
<br/>
postgres=# \l<br/>
                               List of databases<br/>
    Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges<br/>
-------------+----------+----------+---------+---------+-----------------------<br/>
 buffer_temp | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 lock        | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +<br/>
             |          |          |         |         | postgres=CTc/postgres<br/>
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +<br/>
             |          |          |         |         | postgres=CTc/postgres<br/>
 testdb      | postgres | UTF8     | C.UTF-8 | C.UTF-8 | postgres=CTc/postgres+<br/>
             |          |          |         |         | readonly=c/postgres<br/>
 testdb1     | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres         +<br/>
             |          |          |         |         | postgres=CTc/postgres+<br/>
             |          |          |         |         | readonly1=c/postgres<br/>
 tuning      | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
(8 rows)<br/>
<br/>
1) Создаем БД и в ней таблицу.<br/>
<br/>
postgres=# Create database test_backup;<br/>
CREATE DATABASE<br/>
postgres=# c \test_backup<br/>
invalid command \test_backup<br/>
Try \? for help.<br/>
postgres-# \c test_backup<br/>
You are now connected to database "test_backup" as user "postgres".<br/>
test_backup=# CREATE TABLE emp as select generate_series(1,100) as id, md5(random()::text)::char(10) as emp_Name; -- создание таблицы<br/>
SELECT 100<br/>
test_backup=# select * from emp limit 10;  -- проверка, что данные есть<br/>
 id |  emp_name  <br/>
----+------------<br/>
  1 | bf996b59bb<br/>
  2 | 53c53354be<br/>
  3 | 5870b9bfad<br/>
  4 | 28e843a29d<br/>
  5 | f96af999fc<br/>
  6 | 04f29d4223<br/>
  7 | a21797eafa<br/>
  8 | 9c0f66c55b<br/>
  9 | ea5c9ce1fc<br/>
 10 | 3a9d48b11a<br/>
(10 rows)<br/>
<br/>
--1) Логическое копирование<br/>
<br/>
а) root@9091a9f0ca42:~# su postgres<br/>
postgres@9091a9f0ca42:/root$ cd ~<br/>
postgres@9091a9f0ca42:~$ pwd   -- проверяем где мы<br/>
/var/lib/postgresql<br/>
postgres@9091a9f0ca42:~$ mkdir backup   -- создаем директорию для бэкапов<br/>
postgres@9091a9f0ca42:~$ ls -all<br/>
total 36<br/>
drwxr-xr-x 6 postgres postgres 4096 Dec 11 21:10 .<br/>
drwxr-xr-x 1 root     root     4096 Mar  4  2023 ..<br/>
-rw------- 1 postgres postgres 2814 May  9  2023 .bash_history<br/>
drwx------ 3 postgres postgres 4096 May  2  2023 .cache<br/>
drwx------ 3 postgres postgres 4096 May  2  2023 .config<br/>
drwx------ 3 postgres postgres 4096 May  2  2023 .local<br/>
-rw------- 1 postgres postgres 1506 Mar 19  2023 .psql_history<br/>
drwxrwxr-x 2 postgres postgres 4096 Dec 11 21:10 backup<br/>
postgres@9091a9f0ca42:~$ <br/>
<br/>
<br/>
b) --запускаем копирование<br/>
test_backup=# \copy emp to '/var/lib/postgresql/backup/emp.sql';<br/>
COPY 100<br/>
test_backup=#<br/>
<br/>
c)--проверяем, что данные в файле есть<br/>
<br/>
postgres@9091a9f0ca42:~$ cd backup<br/>
postgres@9091a9f0ca42:~/backup$ ls -la<br/>
total 12<br/>
drwxrwxr-x 2 postgres postgres 4096 Dec 11 21:27 .<br/>
drwxr-xr-x 6 postgres postgres 4096 Dec 11 21:10 ..<br/>
-rw-rw-r-- 1 postgres postgres 1392 Dec 11 21:27 emp.sql<br/>
postgres@9091a9f0ca42:~/backup$ cat emp.sql<br/>
1bf996b59bb<br/>
253c53354be<br/>
35870b9bfad<br/>
428e843a29d<br/>
....<br/>
986a7f8dfd06<br/>
99ba0ee9147f<br/>
100db62e6e565<br/>
postgres@9091a9f0ca42:~/backup$<br/>
<br/>
d) --создаем новую таблицу и копируем в нее<br/>
<br/>
test_backup=# CREATE TABLE emp_department1 ( id int, emp_Name char(10));<br/>
CREATE TABLE<br/>
test_backup=# \copy emp_department1 from '/var/lib/postgresql/backup/emp.sql';<br/>
COPY 100<br/>
test_backup=# select * from emp_department1 limit 15;<br/>
 id |  emp_name  <br/>
----+------------<br/>
  1 | bf996b59bb<br/>
  2 | 53c53354be<br/>
  3 | 5870b9bfad<br/>
  4 | 28e843a29d<br/>
  5 | f96af999fc<br/>
  6 | 04f29d4223<br/>
  7 | a21797eafa<br/>
  8 | 9c0f66c55b<br/>
  9 | ea5c9ce1fc<br/>
 10 | 3a9d48b11a<br/>
 11 | d1c7d2a915<br/>
 12 | 69c9d82166<br/>
 13 | f713c2f65d<br/>
 14 | 22b533a25e<br/>
 15 | 0b4a8cb4f6<br/>
(15 rows)<br/>
<br/>
test_backup=#<br/>
<br/>
2)  Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц<br/>
<br/>
-- for 2 tables<br/>
<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1 -Fc -d test_backup -C > emp_dump_2tables.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup --create | gzip  > emp_dump_2tables_Zip.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup --create | gzip  > emp_dump_2tables_Zip.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup -z 5 --create | gzip  > emp_dump_2tables_Zip5.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup -Z 0 --create | gzip  > emp_dump_2tables_Zip0.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup -Z 5 -Fc   > emp_dump_2tables_Zip5_1.gz<br/>
postgres@9091a9f0ca42:~/backup$ pg_dump -t emp -t emp_department1  -d test_backup -Z 9 --create | gzip  > emp_dump_2tables_Zip9.gz<br/>
postgres@9091a9f0ca42:~/backup$ ls -la<br/>
<br/>
drwxrwxr-x 2 postgres postgres 4096 Dec 11 23:25 .<br/>
drwxr-xr-x 6 postgres postgres 4096 Dec 11 21:10 ..<br/>
-rw-rw-r-- 1 postgres postgres 1392 Dec 11 21:27 emp.sql<br/>
-rw-rw-r-- 1 postgres postgres 5002 Dec 11 22:08 emp_dump.sql<br/>
-rw-rw-r-- 1 postgres postgres 3365 Dec 11 22:46 emp_dump_2tables.gz<br/>
-rw-rw-r-- 1 postgres postgres 3365 Dec 11 22:26 emp_dump_2tables.sql<br/>
-rw-rw-r-- 1 postgres postgres 1538 Dec 11 23:07 emp_dump_2tables_Zip.gz<br/>
-rw-rw-r-- 1 postgres postgres 1538 Dec 11 23:22 emp_dump_2tables_Zip0.gz<br/>
-rw-rw-r-- 1 postgres postgres 1561 Dec 11 23:21 emp_dump_2tables_Zip5.gz<br/>
-rw-rw-r-- 1 postgres postgres 3365 Dec 11 23:24 emp_dump_2tables_Zip5_1.gz<br/>
-rw-rw-r-- 1 postgres postgres 1561 Dec 11 23:25 emp_dump_2tables_Zip9.gz<br/>
postgres@9091a9f0ca42:~/backup$ <br/>
<br/>
<br/>
 3) Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!<br/>
<br/>
postgres@9091a9f0ca42:~/backup$ pg_restore -C test_backup2 -U postgres -t emp_department1 emp_dump_2tables.gz<br/>
pg_restore: error: too many command-line arguments (first is "emp_dump_2tables.gz") - получила ошибку !!!<br/>
Try "pg_restore --help" for more information.<br/>
<br/>
- нашла вариант с созданием новой базы на основе template0<br/>
postgres@9091a9f0ca42:~/backup$ createdb -T template0 test_backup2<br/>
postgres@9091a9f0ca42:~/backup$ pg_restore -d  test_backup2 -U postgres -t emp_department1 emp_dump_2tables.gz<br/>
postgres@9091a9f0ca42:~/backup$ <br/>
<br/>
-- Проверяем, что база есть, и в ней только вторая таблица с данными<br/>
test_backup=# \l<br/>
                               List of databases<br/>
     Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges<br/>
--------------+----------+----------+---------+---------+-----------------------<br/>
 buffer_temp  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 lock         | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 postgres     | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 template0    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +<br/>
              |          |          |         |         | postgres=CTc/postgres<br/>
 template1    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +<br/>
              |          |          |         |         | postgres=CTc/postgres<br/>
 test_backup  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 test_backup2 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
 testdb       | postgres | UTF8     | C.UTF-8 | C.UTF-8 | postgres=CTc/postgres+<br/>
              |          |          |         |         | readonly=c/postgres<br/>
 testdb1      | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres         +<br/>
              |          |          |         |         | postgres=CTc/postgres+<br/>
              |          |          |         |         | readonly1=c/postgres<br/>
 tuning       | postgres | UTF8     | C.UTF-8 | C.UTF-8 | <br/>
(10 rows)<br/>
<br/>
test_backup=# \c test_backup2<br/>
You are now connected to database "test_backup2" as user "postgres".<br/>
test_backup2=# \dt<br/>
              List of relations<br/>
 Schema |      Name       | Type  |  Owner<br/>
--------+-----------------+-------+----------<br/>
 public | emp_department1 | table | postgres<br/>
(1 row)<br/>
<br/>
test_backup2=# select * from emp_department1;<br/>
 id  |  emp_name  <br/>
-----+------------<br/>
   1 | bf996b59bb<br/>
   2 | 53c53354be<br/>
   3 | 5870b9bfad<br/>
   4 | 28e843a29d<br/>
   5 | f96af999fc<br/>
   6 | 04f29d4223<br/>
   ---<br/>
   100 | db62e6e565<br/>

