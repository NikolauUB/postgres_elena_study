1. На VM c Ubuntu развернула докер:<br/>
   sudo apt update<br/>
   инсталяция докера  sudo apt install docker.io -y<br/>
   инсталяция зависимостей sudo snap install docker<br/>
   проверка, что докер запущен sudo systemctl status docker<br/>

java@Ubuntu1604x64:$ sudo systemctl status docker<br/>
● docker.service - Docker Application Container Engine<br/>
Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)<br/>
Active: active (running) since Sun 2023-02-19 23:55:56 MSK; 5min ago<br/>
Docs: https://docs.docker.com<br/>
Main PID: 16839 (dockerd)<br/>

2. Создаем docker-сеть:<br/>
   sudo docker network create pg-net<br/>
3. Создала директорию /home/java/postgresql14/<br/>
4. Подключаем созданную сеть к контейнеру сервера Postgres ( порт 5432 на виртулке занят, поэтому прописала 5454):<br/>
   java@Ubuntu1604x64:$ sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5454:5432 -v /home/java/postgresql14/data:/var/lib/postgresql/data postgres:14<br/>
5. Проверка, что контейнер есть:<br/>
   sudo docker container ls
6. Проверка, что в подмонтированном каталоге появились файлы:<br/>
   cd postgresql14<br/>
   java@Ubuntu1604x64:~/postgresql14$ sudo ls data/<br/>
   base    pg_commit_ts  pg_hba.conf    pg_logical    pg_notify    pg_serial     pg_stat      pg_subtrans  pg_twophase  pg_wal   postgresql.auto.conf  postmaster.opts<br/>
   global  pg_dynshmem   pg_ident.conf  pg_multixact  pg_replslot  pg_snapshots  pg_stat_tmp  pg_tblspc    PG_VERSION   pg_xact  postgresql.conf       postmaster.pid<br/>


7. Запускаем отдельный контейнер с клиентом в общей сети с БД:<br/>
   sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres

8. Создаем базу, таблицу:<br/>
   postgres=# create database lena;<br/>
   CREATE DATABASE<br/>
   postgres=# \l<br/>
   List of databases<br/>
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges<br/>
   -----------+----------+----------+------------+------------+-----------------------<br/>
   lena      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |<br/>
   postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |<br/>
   template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +<br/>
   |          |          |            |            | postgres=CTc/postgres<br/>
   template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +<br/>
   |          |          |            |            | postgres=CTc/postgres<br/>
   (4 rows)<br/>

postgres=# CREATE TABLE USERS ( ID INT NOT NULL, NAME varchar(255) NOT NULL);<br/>
CREATE TABLE<br/>
postgres=# INSERT INTO USERS ( ID,NAME) VALUES (1,'IVANOV IVAN'), (2,'Petrov Petr');<br/>
INSERT 0 2<br/>
postgres=# \q<br/>

9. Удаление контейнера с сервером<br/>
   sudo docker stop pg-server<br/>
   sudo docker rm pg-server<br/>
10. Создаем контейнер с сервером снова:<br/>
    sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5454:5432 -v /home/java/postgresql14/data:/var/lib/postgresql/data postgres:14<br/>
11. Запускаем клиента<br/>
    sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres<br/>
    12.Смотрим список баз \l<br/>

postgres=# \l<br/>
List of databases<br/>
Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges<br/>
-----------+----------+----------+------------+------------+-----------------------<br/>
lena      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |<br/>
postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |<br/>
template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +<br/>
|          |          |            |            | postgres=CTc/postgres<br/>
template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +<br/>
|          |          |            |            | postgres=CTc/postgres<br/>
(4 rows)<br/>

Данные в таблице есть :<br/>
postgres=# select * from Users;<br/>
id |    name<br/>
----+-------------<br/>
1 | IVANOV IVAN<br/>
2 | Petrov Petr<br/>
(2 rows)<br/>
