Текущие настройки : <br/>
name|setting<br/>
--------------------------------+-----------------------------------------<br/>
port                           | 5432<br/>
shared_buffers                 | 200 MB<br/>
dynamic_shared_memory_type     | posix<br/>
logging_collector              | on<br/>
max_connections                | 40<br/>
wal_buffers                    | 16MB<br/>
default_statistics_target      | 500<br/>
effective_io_concurrency       | 2<br/>
work_mem                       | 6553kB<br/>
log_autovacuum_min_duration    | 0<br/>
autovacuum_max_workers         | 5<br/>
autovacuum_naptime             | 15s<br/>
autovacuum_vacuum_threshold    | 50<br/>
autovacuum_vacuum_scale_factor | 0.01<br/>
autovacuum_vacuum_cost_delay   | 10<br/>
autovacuum_vacuum_cost_limit   | 1500<br/>
checkpoint_timeout             | 30s<br/>
log_checkpoints                | on<br/>
max_wal_size                   | 1GB<br/>
min_wal_size                   | 80MB<br/>
fsync                          | on<br/>
synchronous_commit             | off<br/>
log_lock_waits                 | on<br/>
deadlock_timeout               | 0.2s<br/>
<br/>
Запускаем  pgbench -c 10 -j 2 -P 60 -T 300 -M extended tuning<br/>
tps = 4322.860073 (without initial connection time)<br/>
<br/>
Выставляем параметр synchronous_commit = on<br/>
Запускаем  pgbench -c 10 -j 2 -P 60 -T 300 -M extended tuning<br/>
tps = 1945.829303 (without initial connection time)<br/>
Вывод : Отключенный параметр synchronous_commit дает очень хороший прирост производительности, но это небезопасно.<br/>
<br/>
2. Запучкаем утилиту pgtune<br/>
Создаем файл pgtune.conf в conf.d папке<br/>
   <br/>
max_connections = 100<br/>
shared_buffers = 256MB<br/>
effective_cache_size = 768MB<br/>
maintenance_work_mem = 64MB<br/>
checkpoint_completion_target = 0.9<br/>
wal_buffers = 7864kB<br/>
default_statistics_target = 100<br/>
random_page_cost = 1.1<br/>
effective_io_concurrency = 200<br/>
work_mem = 655kB<br/>
min_wal_size = 1GB<br/>
max_wal_size = 4GB<br/>
   <br/>   
Снова запускаем pgbench -c 10 -j 2 -P 50 -T 150 -M extended tuning<br/>
 (1) synchronous_commit = on  : tps = 2750.168942 (without initial connection time)<br/>
 (2) synchronous_commit = off : tps = 5214.910754 (without initial connection time)<br/>
Увеличили shared_buffers, work_mem  уменьшили в 10 раз, увеличили  min_wal_size, max_wal_size  =><br/> 
производительность увеличилась на 20% для варианта (1) и на 40% для варианта (2)<br/>
   <br/>
3) Следующий тул https://pgconfigurator.cybertec.at/<br/>
Проапдейтим файл pgtune.conf:<br/>
   <br/>
# Connectivity<br/>
max_connections = 100<br/>
# Memory Settings<br/>
shared_buffers = '256 MB'<br/>
work_mem = '4 MB'<br/>
maintenance_work_mem = '320 MB'<br/>
huge_pages = off<br/>
effective_cache_size = '1 GB'<br/>
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function<br/>
random_page_cost = 1.25<br/>
checkpoint_timeout  = '15 min'<br/>
checkpoint_completion_target = 0.9<br/>
max_wal_size = '1024 MB'<br/>
min_wal_size = '80 MB'<br/>
<br/>
(2) synchronous_commit = off  : tps = 4402.874735 (without initial connection time)<br/>
Если уменьшить work_mem = '1 MB', то<br/>
             (2) synchronous_commit = off  : tps = 4391.366729 (without initial connection time)<br/>
Если изменить еще :<br/>
min_wal_size = 1GB<br/>
max_wal_size = 4GB, то <br/>
             (2) synchronous_commit = off  : tps = 4485.671969 (without initial connection time)<br/>
<br/>
В моем случае настройки pgtune дали лучший результат.<br/>