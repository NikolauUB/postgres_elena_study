
1. ALTER SYSTEM SET checkpoint_timeout = '30s';<br/>
   <br/>
   after restart server<br/>
   postgres=# show checkpoint_timeout;<br/>
   checkpoint_timeout<br/>
   --------------------<br/>
   30s<br/>
   (1 row)<br/>
   <br/>
   <br/>
   ALTER SYSTEM SET max_wal_size = '1GB';<br/>
   ALTER SYSTEM SET min_wal_size = '80MB';<br/>
   <br/>
   postgres=# show checkpoint_completion_target;<br/>
   checkpoint_completion_target<br/>
   ------------------------------<br/>
   0.9<br/>
   (1 row)<br/>
   <br/>
   postgres=# ALTER SYSTEM SET log_checkpoints = on;<br/>
   ALTER SYSTEM<br/>
   <br/>
   postgres=# SELECT pg_reload_conf();<br/>
   pg_reload_conf<br/>
   ----------------<br/>
   t<br/>
   (1 row)<br/>
   <br/>
   -- Снимаем первоначальное значение lsn<br/>
   postgres=# \c buffer_temp;<br/>
   SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
   You are now connected to database "buffer_temp" as user "postgres".<br/>
   buffer_temp=# SELECT pg_current_wal_insert_lsn();<br/>
   pg_current_wal_insert_lsn<br/>
   ---------------------------<br/>
   0/F182FB78<br/>
   (1 row)<br/>
   <br/>
   buffer_temp=#<br/>
   <br/>
2.  Подаем нагрузку<br/>
    pgbench -i buffer_temp<br/>
    pgbench -c1 -P 60 -T 600 -U postgres buffer_temp<br/>
    <br/>
    postgres@9091a9f0ca42:/root$ pgbench -c1 -P 60 -T 600 -U postgres buffer_temp<br/>
    pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))<br/>
    starting vacuum...end.<br/>
    progress: 60.0 s, 633.5 tps, lat 1.578 ms stddev 0.610<br/>
    progress: 120.0 s, 650.6 tps, lat 1.537 ms stddev 0.389<br/>
    progress: 180.0 s, 547.9 tps, lat 1.825 ms stddev 1.525<br/>
    progress: 240.0 s, 602.1 tps, lat 1.660 ms stddev 0.774<br/>
    progress: 300.0 s, 622.1 tps, lat 1.607 ms stddev 1.034<br/>
    progress: 360.0 s, 648.5 tps, lat 1.542 ms stddev 0.916<br/>
    progress: 420.0 s, 649.9 tps, lat 1.538 ms stddev 0.653<br/>
    progress: 480.0 s, 648.4 tps, lat 1.542 ms stddev 0.693<br/>
    progress: 540.0 s, 639.2 tps, lat 1.564 ms stddev 0.692<br/>
    progress: 600.0 s, 625.5 tps, lat 1.598 ms stddev 0.651<br/>
    transaction type: <builtin: TPC-B (sort of)><br/>
    scaling factor: 1<br/>
    query mode: simple<br/>
    number of clients: 1<br/>
    number of threads: 1<br/>
    duration: 600 s<br/>
    number of transactions actually processed: 376067<br/>
    latency average = 1.595 ms<br/>
    latency stddev = 0.836 ms<br/>
    initial connection time = 1.747 ms<br/>
    tps = 626.778634 (without initial connection time)<br/>
    postgres@9091a9f0ca42:/root$<br/>
    <br/>
    <br/>
    buffer_temp=# SELECT pg_current_wal_insert_lsn();<br/>
    pg_current_wal_insert_lsn<br/>
    ---------------------------<br/>
    1/D2358C8<br/>
    (1 row)<br/>
    <br/>
    buffer_temp=#<br/>
    <br/>
    buffer_temp=# SELECT pg_size_pretty('1/D2358C8'::pg_lsn - '0/F182FB78'::pg_lsn);<br/>
    pg_size_pretty<br/>
    ----------------<br/>
    442 MB<br/>
    (1 row)<br/>
    <br/>
    buffer_temp=#<br/>
    <br/>
    На одну контрольную точку приходится , в среднем, 22 MB<br/>
    <br/>
    Проверка журнальных записей возвращает ошибку ( не смогла разобраться почему), такого файла в директории действительно нет<br/>
    postgres@9091a9f0ca42:/root$ /usr/lib/postgresql/14/bin/pg_waldump -p /mnt/data/14/main/pg_wal -s 0/F182FB78 -e 1/D2358C8<br/>
    pg_waldump: fatal: could not find file "0000000100000000000000D6": No such file or directory<br/>
    <br/>
    postgres@9091a9f0ca42:/root$ /usr/lib/postgresql/14/bin/pg_controldata -D /mnt/data/14/main/ | egrep 'Latest.*location'<br/>
    <br/>
    Latest checkpoint location:1/D235818<br/>
    Latest checkpoint's REDO location:    1/D2357E0<br/>
    <br/>
    <br/>
    buffer_temp=# SELECT * FROM pg_stat_bgwriter \gx<br/>
    -[ RECORD 1 ]---------+-----------------------------<br/>
    checkpoints_timed     | 101      -- Количество запланированных контрольных точек, которые уже были выполнены<br/>
    checkpoints_req       | 2           Количество запрошенных контрольных точек, которые уже были выполнены<br/>
    checkpoint_write_time | 166155<br/>
    checkpoint_sync_time  | 1330<br/>
    buffers_checkpoint    | 170        Количество буферов, записанных при выполнении контрольных точек<br/>
    buffers_clean         | 96078<br/>
    maxwritten_clean      | 0<br/>
    buffers_backend       | 359050<br/>
    buffers_backend_fsync | 0<br/>
    buffers_alloc         | 674322<br/>
    stats_reset           | 2023-04-03 21:00:03.86489+00<br/>
    <br/>
    buffer_temp=#<br/>
    <br/>
    По логу postgresql-14-main.log получилось 20 контрольных точек<br/>
    а запрос SELECT * FROM pg_stat_bgwriter \gx<br/>
    показывает, что все прошли по расписанию ( checkpoints_req = 2 было до запуска нагрузки )<br/>
    Все контрольные точки сработали по checkpoint_timeout<br/>
    <br/>
    <br/>
5. Устанавливаем асинхронный режим<br/>
   buffer_temp=# ALTER SYSTEM set synchronous_commit = off;<br/>
   ALTER SYSTEM<br/>
   buffer_temp=# select pg_reload_conf();<br/>
   pg_reload_conf<br/>
   ----------------<br/>
   t<br/>
   (1 row)<br/>
   <br/>
   Запускаем pgbench<br/>
   postgres@9091a9f0ca42:/root$ pgbench -c1 -P 60 -T 600 -U postgres buffer_temp<br/>
   pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))<br/>
   starting vacuum...end.<br/>
   progress: 60.0 s, 1159.1 tps, lat 0.862 ms stddev 0.245<br/>
   progress: 120.0 s, 1170.7 tps, lat 0.854 ms stddev 0.227<br/>
   progress: 180.0 s, 1162.8 tps, lat 0.860 ms stddev 0.225<br/>
   progress: 240.0 s, 1151.1 tps, lat 0.868 ms stddev 0.231<br/>
   progress: 300.0 s, 1106.2 tps, lat 0.904 ms stddev 0.301<br/>
   progress: 360.0 s, 1150.5 tps, lat 0.869 ms stddev 0.270<br/>
   progress: 420.0 s, 1094.0 tps, lat 0.914 ms stddev 0.379<br/>
   progress: 480.0 s, 1193.7 tps, lat 0.837 ms stddev 0.238<br/>
   progress: 540.0 s, 1224.8 tps, lat 0.816 ms stddev 0.209<br/>
   progress: 600.0 s, 1230.0 tps, lat 0.813 ms stddev 0.192<br/>
   transaction type: <builtin: TPC-B (sort of)><br/>
   scaling factor: 1<br/>
   number of clients: 1<br/>
   number of threads: 1<br/>
   duration: 600 s<br/>
   number of transactions actually processed: 698567<br/>
   latency average = 0.859 ms<br/>
   latency stddev = 0.257 ms<br/>
   initial connection time = 1.785 ms<br/>
   tps = 1164.281149 (without initial connection time)<br/>
   postgres@9091a9f0ca42:/root$<br/>
   <br/>
   При синхронной фиксации мы получали примерно 626 транзакций в секунду (tps), при асинхронной — 1164<br/>
   <br/>
6. Начиная с 11 версии включать и выключать контрольные суммы можно с помощью утилиты pg_checksums<br/>
   Останавливаем кластер  sudo pg_ctlcluster 14 main stop<br/>
   инсталируем :  apt-get install postgresql-14-pg-checksums<br/>
   <br/>
   Включаем контрольные суммы:<br/>
   root@9091a9f0ca42:~# su - postgres -c '/usr/lib/postgresql/14/bin/pg_checksums --enable -D "/mnt/data/14/main"'<br/>
   Checksum operation completed<br/>
   Files scanned:  1842<br/>
   Blocks scanned: 17058<br/>
   pg_checksums: syncing data directory<br/>
   pg_checksums: updating control file<br/>
   Checksums enabled in cluster<br/>
   root@9091a9f0ca42:~#<br/>
   <br/>
   Запускаем кластер и создаем таблицу . После чего находим ее файл на диске<br/>
   Останавливаем кластер и меняем в нем байт информации<br/>
   buffer_temp=# CREATE TABLE test_text(t text);<br/>
   CREATE TABLE<br/>
   buffer_temp=# INSERT INTO test_text SELECT 'Тест '||s.id FROM generate_series(1,500) AS s(id);<br/>
   INSERT 0 500<br/>
   buffer_temp=# select pg_relation_filepath('test_text')<br/>
   buffer_temp-# ;<br/>
   pg_relation_filepath<br/>
   ----------------------<br/>
   base/26492/35351<br/>
   (1 row)<br/>
   <br/>
   Запускаем кластер и делаем выборку из таблицы<br/>
   postgres=# \c buffer_temp;<br/>
   SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)<br/>
   You are now connected to database "buffer_temp" as user "postgres".<br/>
   buffer_temp=# select * from test_text;<br/>
   WARNING:  page verification failed, calculated checksum 44726 but expected 53188<br/>
   ERROR:  invalid page in block 3 of relation base/26492/35351<br/>
   buffer_temp=#<br/>
   <br/>
   Получили ошибку о несовпадении контрольных сумм.<br/>
   Чтобы получить данные:<br/>
   <br/>
   set ignore_checksum_failure = on;<br/>
   выдаётся предупреждение, что контрольная сумма данных не совпадает, но запрос выполняется.<br/>
