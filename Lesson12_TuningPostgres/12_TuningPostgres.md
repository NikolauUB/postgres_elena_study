Текущие настройки :
name|setting
--------------------------------+-----------------------------------------
port                           | 5432
shared_buffers                 | 200 MB
dynamic_shared_memory_type     | posix
logging_collector              | on
max_connections                | 40
wal_buffers                    | 16MB
default_statistics_target      | 500
effective_io_concurrency       | 2
work_mem                       | 6553kB
log_autovacuum_min_duration    | 0
autovacuum_max_workers         | 5
autovacuum_naptime             | 15s
autovacuum_vacuum_threshold    | 50
autovacuum_vacuum_scale_factor | 0.01
autovacuum_vacuum_cost_delay   | 10
autovacuum_vacuum_cost_limit   | 1500
checkpoint_timeout             | 30s
log_checkpoints                | on
max_wal_size                   | 1GB
min_wal_size                   | 80MB
fsync                          | on
synchronous_commit             | off
log_lock_waits                 | on
deadlock_timeout               | 0.2s

Запускаем  pgbench -c 10 -j 2 -P 60 -T 300 -M extended tuning
tps = 4322.860073 (without initial connection time)

Выставляем параметр synchronous_commit = on
Запускаем  pgbench -c 10 -j 2 -P 60 -T 300 -M extended tuning
tps = 1945.829303 (without initial connection time)
Вывод : Отключенный параметр synchronous_commit дает очень хороший прирост производительности, но это небезопасно.

2. Запучкаем утилиту pgtune
Создаем файл pgtune.conf в conf.d папке

max_connections = 100
shared_buffers = 256MB
effective_cache_size = 768MB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 7864kB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 655kB
min_wal_size = 1GB
max_wal_size = 4GB

Снова запускаем pgbench -c 10 -j 2 -P 50 -T 150 -M extended tuning
 (1) synchronous_commit = on  : tps = 2750.168942 (without initial connection time)
 (2) synchronous_commit = off : tps = 5214.910754 (without initial connection time)
Увеличили shared_buffers, work_mem  уменьшили в 10 раз, увеличили  min_wal_size, max_wal_size  => 
производительность увеличилась на 20% для варианта (1) и на 40% для варианта (2)

3) Следующий тул https://pgconfigurator.cybertec.at/
Проапдейтим файл pgtune.conf:

# Connectivity
max_connections = 100
# Memory Settings
shared_buffers = '256 MB'
work_mem = '4 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '1 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25
checkpoint_timeout  = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '80 MB'

(2) synchronous_commit = off  : tps = 4402.874735 (without initial connection time)
Если уменьшить work_mem = '1 MB', то
             (2) synchronous_commit = off  : tps = 4391.366729 (without initial connection time)
Если изменить еще :
min_wal_size = 1GB
max_wal_size = 4GB, то 
             (2) synchronous_commit = off  : tps = 4485.671969 (without initial connection time)

В моем случае настройки pgtune дали лучший результат.