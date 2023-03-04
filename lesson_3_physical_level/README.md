ДЗ сделано на windows c докером<br/>
1. Создала папку C:/mnt/data в windows<br/>
2. Создала контейнер с ubuntu с примонтированной папкой C:/mnt<br/>
$ docker run -d   --name ubuntu_desktop   -v /c/dev/shm:/dev/shm --volume //c/mnt:/mnt   -p 6080:80   dorowu/ubuntu-desktop-lxde-vnc<br/>
   <br/>
3. Запустила docker and проинсталировала postress 14<br/>
sudo apt update && sudo apt upgrade<br/>
sudo apt -y install gnupg2 wget vim --install necessary packages<br/>
sudo apt-cache search postgresql | grep postgresql<br/>
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'<br/>
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -<br/>
sudo apt -y update<br/>
sudo apt -y install postgresql-14<br/>
   <br/>
4. Проверим состояние кластера<br/>
root@9091a9f0ca42:/# sudo -u postgres pg_lsclusters<br/>
Ver Cluster Port Status Owner    Data directory              Log file<br/>
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log<br/>
   <br/>
5. Запустить кластер<br/>
root@9091a9f0ca42:/# sudo  -u postgres pg_ctlcluster 14 main start<br/>
root@9091a9f0ca42:/# sudo -u  postgres pg_lsclusters<br/>
Ver Cluster Port Status Owner    Data directory              Log file<br/>
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log<br/>
   <br/>
6. Заходим в postgres <br/>
root@9091a9f0ca42:/# psql -u postgres localhost -5432<br/>
Получила ошибку => меняю пароль<br/>
root@9091a9f0ca42:/# sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'postgres';"<br/>
ALTER ROLE<br/>
root@9091a9f0ca42:/# psql -U postgres -h localhost -p 5432<br/>
Password for user postgres: <br/>
psql (14.7 (Ubuntu 14.7-1.pgdg20.04+1))<br/>
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
Type "help" for help.<br/>
   <br/>
7. Создаю таблицу<br/>
postgres=# Create table testtest ( i int)<br/>
postgres=# insert into testtest values (1);<br/>
INSERT 0 1<br/>
postgres=# insert into testtest values (3),(2);<br/>
INSERT 0 2<br/>
postgres=# select * from testtest;<br/>
 i <br/>
---<br/>
 1<br/>
 3<br/>
 2<br/>
(3 rows)<br/>
   <br/>
postgres=# <br/>

8. Убедилась, что windows папка уже примонтирована:<br/>
root@9091a9f0ca42:/# ls /mnt<br/>
data<br/>
   <br/>
9. Сделала пользователя postgres владельцем<br/>
root@9091a9f0ca42:/mnt# sudo chown -R postgres:postgres /mnt/data<br/>
   <br/>
10. Остановка postgres<br/>
root@9091a9f0ca42:/mnt# sudo -u postgres pg_ctlcluster 14 main stop<br/>
    <br/>
11. Перенесла содержимое в /mnt/data:<br/>
root@9091a9f0ca42:/mnt# mv /var/lib/postgresql/14 /mnt/data<br/>
    <br/>
12. Запуск кластера<br/>
root@9091a9f0ca42:/mnt# sudo -u postgres pg_ctlcluster 14 main start<br/>
Error: /var/lib/postgresql/14/main is not accessible or does not exist<br/>
root@9091a9f0ca42:/mnt# <br/>
Результат : ошибка, так как в конфигурационном файле postgresql.conf прописана переменная data_directory = '/var/lib/postgresql/14/main'. <br/>
Этого пути уже нет. Поэтому меняем путь '/var/lib/postgresql' на '/mnt/data'<br/>
    <br/>
root@9091a9f0ca42:/etc/postgresql/14/main# cat postgresql.conf|grep 'var/lib/postgre'<br/>
data_directory = '/var/lib/postgresql/14/main'# use data in another directory<br/>
root@9091a9f0ca42:/etc/postgresql/14/main# vi postgresql.conf<br/>
    <br/>
13.Запускаем сервер<br/>
    <br/>
root@9091a9f0ca42:/etc/postgresql/14/main# sudo -u postgres pg_ctlcluster 14 main start<br/>
root@9091a9f0ca42:/etc/postgresql/14/main# sudo -u postgres pg_lsclusters<br/>
Ver Cluster Port Status Owner    Data directory    Log file<br/>
14  main    5432 online postgres /mnt/data/14/main /var/log/postgresql/postgresql-14-main.log<br/>
root@9091a9f0ca42:/etc/postgresql/14/main#<br/>
    <br/>
14. <br/>
root@9091a9f0ca42:/etc/postgresql/14/main# psql -U postgres -h localhost -p 5432<br/>
Password for user postgres: <br/>
psql (14.7 (Ubuntu 14.7-1.pgdg20.04+1))<br/>
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
Type "help" for help.<br/>
<br/>
postgres=# select * from testtest;<br/>
 i <br/>
---<br/>
 1<br/>
 3<br/>
 2<br/>
(3 rows)<br/>
<br/>
postgres=# <br/>