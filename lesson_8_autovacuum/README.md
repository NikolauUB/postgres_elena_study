Default data for claster:
<br/>
max_connections = 100
<br/>
shared_buffers = 128MB
<br/>
effective_cache_size = 4GB
<br/>
maintenance_work_mem = 64MB
<br/>
checkpoint_completion_target = 0.9
<br/>
wal_buffers = 4MB
<br/>
default_statistics_target = 100
<br/>
random_page_cost = 4
<br/>  
effective_io_concurrency = 1
<br/>
work_mem = 4MB
<br/>
min_wal_size = 80MB
<br/>
max_wal_size = 1GB
<br/>

<br/>
1. Set new data:
<br/>
  max_connections = 40
<br/>
  shared_buffers = 1GB
<br/>
  effective_cache_size = 3GB
<br/>
  maintenance_work_mem = 512MB
<br/>
  checkpoint_completion_target = 0.9
<br/>
  wal_buffers = 16MB
<br/>
  default_statistics_target = 500
<br/>
  random_page_cost = 4
<br/>
  effective_io_concurrency = 2
<br/>
  work_mem = 6553kB
<br/>
  min_wal_size = 4GB
<br/>
  max_wal_size = 16G
<br/>
<br/>
ALTER SYSTEM SET shared_buffers = '1GB';
<br/>
ALTER SYSTEM SET effective_cache_size = '3GB';
<br/>
ALTER SYSTEM SET maintenance_work_mem = '512MB';
<br/>
ALTER SYSTEM SET wal_buffers = '16MB';
<br/>
ALTER SYSTEM SET default_statistics_target = 500;
<br/>
ALTER SYSTEM SET effective_io_concurrency = 2;
<br/>
ALTER SYSTEM SET work_mem = '6553kB';
<br/>
ALTER SYSTEM SET min_wal_size = '4GB';
<br/>
ALTER SYSTEM SET max_wal_size = '16GB';
<br/>
<br/>
2. --restart cluster for applying new settings:
<br/>
sudo pg_ctlcluster 14 main restart
<br/>
psql -U  postgres -h localhost -p 5432
<br/>

<br/>
3. Run pgbench in other window
<br/>
pgbench -i postgres
<br/>
pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 907.4 tps, lat 8.802 ms stddev 6.737
<br/>
progress: 120.0 s, 576.5 tps, lat 13.865 ms stddev 23.444
<br/>
progress: 180.0 s, 256.3 tps, lat 31.202 ms stddev 50.713
<br/>
progress: 240.0 s, 772.3 tps, lat 10.348 ms stddev 8.548
<br/>
progress: 300.0 s, 852.4 tps, lat 9.374 ms stddev 7.335
<br/>
progress: 360.0 s, 890.6 tps, lat 8.971 ms stddev 7.316
<br/>
progress: 420.0 s, 862.3 tps, lat 9.266 ms stddev 7.045
<br/>
progress: 480.0 s, 886.4 tps, lat 9.013 ms stddev 6.803
<br/>
progress: 540.0 s, 903.8 tps, lat 8.840 ms stddev 6.720
<br/>
progress: 600.0 s, 913.0 tps, lat 8.750 ms stddev 6.591
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 469268
<br/>
latency average = 10.217 ms
<br/>
latency stddev = 13.670 ms
<br/>
initial connection time = 12.154 ms
<br/>
tps = 782.118039 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
4. Test 1
<br/>
ALTER SYSTEM SET log_autovacuum_min_duration = 0;
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 10;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '15s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 25;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.05;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1000;
<br/>

<br/>
ostgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 892.4 tps, lat 8.951 ms stddev 6.955
<br/>
progress: 120.0 s, 846.2 tps, lat 9.442 ms stddev 7.276
<br/>
progress: 180.0 s, 868.1 tps, lat 9.205 ms stddev 7.117
<br/>
progress: 240.0 s, 911.4 tps, lat 8.767 ms stddev 6.656
<br/>
progress: 300.0 s, 881.2 tps, lat 9.066 ms stddev 6.873
<br/>
progress: 360.0 s, 905.8 tps, lat 8.820 ms stddev 6.824
<br/>
progress: 420.0 s, 907.7 tps, lat 8.802 ms stddev 6.700
<br/>
progress: 480.0 s, 885.0 tps, lat 9.028 ms stddev 7.098
<br/>
progress: 540.0 s, 916.4 tps, lat 8.718 ms stddev 6.564
<br/>
progress: 600.0 s, 876.6 tps, lat 9.115 ms stddev 7.574
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 533458
<br/>
latency average = 8.986 ms
<br/>
latency stddev = 6.968 ms
<br/>
initial connection time = 14.666 ms
<br/>
tps = 889.101037 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
--
<br/>
5. Test 2
<br/>
ALTER SYSTEM SET log_autovacuum_min_duration = 0;
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '30s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 100;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.1;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1000;
<br/>

<br/>
ostgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 981.4 tps, lat 8.138 ms stddev 5.635
<br/>
progress: 120.0 s, 923.4 tps, lat 8.651 ms stddev 6.342
<br/>
progress: 180.0 s, 931.8 tps, lat 8.573 ms stddev 6.536
<br/>
progress: 240.0 s, 897.9 tps, lat 8.897 ms stddev 6.991
<br/>
progress: 300.0 s, 917.1 tps, lat 8.711 ms stddev 6.479
<br/>
progress: 360.0 s, 897.1 tps, lat 8.906 ms stddev 6.802
<br/>
progress: 420.0 s, 898.3 tps, lat 8.894 ms stddev 6.697
<br/>
progress: 480.0 s, 893.1 tps, lat 8.946 ms stddev 6.786
<br/>
progress: 540.0 s, 902.8 tps, lat 8.849 ms stddev 6.634
<br/>
progress: 600.0 s, 904.7 tps, lat 8.831 ms stddev 6.559
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 548874
<br/>
latency average = 8.733 ms
<br/>
latency stddev = 6.551 ms
<br/>
initial connection time = 12.641 ms
<br/>
tps = 914.794861 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
6. Test 3
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '15s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 50;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.01;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1500;
<br/>

<br/>
postgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 905.3 tps, lat 8.823 ms stddev 6.858
<br/>
progress: 120.0 s, 901.5 tps, lat 8.863 ms stddev 6.411
<br/>
progress: 180.0 s, 877.0 tps, lat 9.110 ms stddev 6.818
<br/>
progress: 240.0 s, 866.4 tps, lat 9.223 ms stddev 6.995
<br/>
progress: 300.0 s, 886.3 tps, lat 9.015 ms stddev 7.314
<br/>
progress: 360.0 s, 896.1 tps, lat 8.917 ms stddev 6.821
<br/>
progress: 420.0 s, 909.3 tps, lat 8.786 ms stddev 6.562
<br/>
progress: 480.0 s, 883.8 tps, lat 9.040 ms stddev 6.756
<br/>
progress: 540.0 s, 887.2 tps, lat 9.005 ms stddev 6.774
<br/>
progress: 600.0 s, 893.2 tps, lat 8.945 ms stddev 6.809
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>

<br/>
7. Test 4
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '15s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 50;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.01;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 20;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 3000;
<br/>
postgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 961.4 tps, lat 8.307 ms stddev 5.930
<br/>
progress: 120.0 s, 922.4 tps, lat 8.661 ms stddev 6.227
<br/>
progress: 180.0 s, 923.8 tps, lat 8.648 ms stddev 6.247
<br/>
progress: 240.0 s, 909.4 tps, lat 8.786 ms stddev 6.606
<br/>
progress: 300.0 s, 921.7 tps, lat 8.668 ms stddev 6.505
<br/>
progress: 360.0 s, 872.7 tps, lat 9.155 ms stddev 6.957
<br/>
progress: 420.0 s, 894.6 tps, lat 8.932 ms stddev 6.785
<br/>
progress: 480.0 s, 854.5 tps, lat 9.351 ms stddev 7.740
<br/>
progress: 540.0 s, 866.5 tps, lat 9.221 ms stddev 7.055
<br/>
progress: 600.0 s, 865.2 tps, lat 9.236 ms stddev 7.097
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 539544
<br/>
latency average = 8.885 ms
<br/>
latency stddev = 6.724 ms
<br/>
initial connection time = 12.023 ms
<br/>
tps = 899.247149 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
8. Test 5
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '30s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 100;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.1;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1500;
<br/>

<br/>
ostgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 952.3 tps, lat 8.386 ms stddev 6.151
<br/>
progress: 120.0 s, 867.1 tps, lat 9.216 ms stddev 6.865
<br/>
progress: 180.0 s, 913.5 tps, lat 8.746 ms stddev 6.337
<br/>
progress: 240.0 s, 870.1 tps, lat 9.183 ms stddev 7.664
<br/>
progress: 300.0 s, 884.7 tps, lat 9.031 ms stddev 6.791
<br/>
progress: 360.0 s, 893.6 tps, lat 8.939 ms stddev 6.716
<br/>
progress: 420.0 s, 889.1 tps, lat 8.988 ms stddev 6.786
<br/>
progress: 480.0 s, 895.3 tps, lat 8.924 ms stddev 6.667
<br/>
progress: 540.0 s, 886.3 tps, lat 9.015 ms stddev 7.122
<br/>
progress: 600.0 s, 901.9 tps, lat 8.858 ms stddev 6.669
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 537244
<br/>
latency average = 8.923 ms
<br/>
latency stddev = 6.784 ms
<br/>
number of transactions actually processed: 537244
<br/>
latency average = 8.923 ms
<br/>
latency stddev = 6.784 ms
<br/>
initial connection time = 12.614 ms
<br/>
tps = 895.413875 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
9. Test 6
<br/>

<br/>
ALTER SYSTEM SET log_autovacuum_min_duration = 0;
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '30s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 50;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.05;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 20;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 3000;
<br/>

<br/>
postgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 947.2 tps, lat 8.432 ms stddev 6.343
<br/>
progress: 120.0 s, 877.4 tps, lat 9.106 ms stddev 6.644
<br/>
progress: 180.0 s, 903.8 tps, lat 8.839 ms stddev 6.391
<br/>
progress: 240.0 s, 913.5 tps, lat 8.746 ms stddev 6.546
<br/>
progress: 300.0 s, 867.8 tps, lat 9.207 ms stddev 7.000
<br/>
progress: 360.0 s, 890.7 tps, lat 8.971 ms stddev 6.811
<br/>
progress: 420.0 s, 909.0 tps, lat 8.789 ms stddev 6.661
<br/>
progress: 480.0 s, 844.7 tps, lat 9.459 ms stddev 8.038
<br/>
progress: 540.0 s, 859.2 tps, lat 9.300 ms stddev 7.384
<br/>
progress: 600.0 s, 875.3 tps, lat 9.128 ms stddev 6.874
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 533332
<br/>
latency average = 8.988 ms
<br/>
latency stddev = 6.879 ms
<br/>
initial connection time = 12.794 ms
<br/>
tps = 888.894345 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
10. Test 7
<br/>
ALTER SYSTEM SET autovacuum_max_workers = 5;
<br/>
ALTER SYSTEM SET autovacuum_naptime = '30s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 50;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.01;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1500;
<br/>

<br/>
postgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 900.1 tps, lat 8.874 ms stddev 6.679
<br/>
progress: 120.0 s, 881.2 tps, lat 9.067 ms stddev 6.697
<br/>
progress: 180.0 s, 883.7 tps, lat 9.041 ms stddev 6.737
<br/>
progress: 240.0 s, 920.0 tps, lat 8.684 ms stddev 6.457
<br/>
progress: 300.0 s, 910.6 tps, lat 8.774 ms stddev 6.630
<br/>
progress: 360.0 s, 908.4 tps, lat 8.795 ms stddev 6.456
<br/>
progress: 420.0 s, 922.2 tps, lat 8.664 ms stddev 6.533
<br/>
progress: 480.0 s, 888.6 tps, lat 8.992 ms stddev 6.832
<br/>
progress: 540.0 s, 868.5 tps, lat 9.200 ms stddev 7.076
<br/>
progress: 600.0 s, 892.4 tps, lat 8.953 ms stddev 6.690
<br/>
progress: 420.0 s, 922.2 tps, lat 8.664 ms stddev 6.533
<br/>
progress: 480.0 s, 888.6 tps, lat 8.992 ms stddev 6.832
<br/>
progress: 540.0 s, 868.5 tps, lat 9.200 ms stddev 7.076
<br/>
progress: 600.0 s, 892.4 tps, lat 8.953 ms stddev 6.690
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 538554
<br/>
latency average = 8.901 ms
<br/>
latency stddev = 6.680 ms
<br/>
initial connection time = 12.847 ms
<br/>
tps = 897.598063 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<br/>

<br/>
11. Test 8
<br/>
ALTER SYSTEM SET autovacuum_naptime = '15s';
<br/>
ALTER SYSTEM SET autovacuum_vacuum_threshold = 100;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.01;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
<br/>
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1000;
<br/>
<br/>
postgres@9091a9f0ca42:~$ pgbench -c8 -P 60 -T 600 -U postgres postgres
<br/>
pgbench (14.7 (Ubuntu 14.7-1.pgdg20.04+1))
<br/>
starting vacuum...end.
<br/>
progress: 60.0 s, 921.3 tps, lat 8.670 ms stddev 6.445
<br/>
progress: 120.0 s, 939.2 tps, lat 8.506 ms stddev 6.040
<br/>
progress: 180.0 s, 933.3 tps, lat 8.559 ms stddev 6.113
<br/>
progress: 240.0 s, 920.0 tps, lat 8.684 ms stddev 6.569
<br/>
progress: 300.0 s, 909.3 tps, lat 8.787 ms stddev 6.596
<br/>
progress: 360.0 s, 918.4 tps, lat 8.699 ms stddev 6.536
<br/>
progress: 420.0 s, 900.8 tps, lat 8.870 ms stddev 6.735
<br/>
progress: 480.0 s, 914.3 tps, lat 8.738 ms stddev 7.085
<br/>
progress: 540.0 s, 863.2 tps, lat 9.257 ms stddev 7.089
<br/>
progress: 600.0 s, 878.0 tps, lat 9.100 ms stddev 6.908
<br/>
transaction type: <builtin: TPC-B (sort of)>
<br/>
scaling factor: 1
<br/>
query mode: simple
<br/>
number of clients: 8
<br/>
number of threads: 1
<br/>
duration: 600 s
<br/>
number of transactions actually processed: 545882
<br/>
latency average = 8.781 ms
<br/>
latency stddev = 6.617 ms
<br/>
initial connection time = 12.080 ms
<br/>
tps = 909.805831 (without initial connection time)
<br/>
postgres@9091a9f0ca42:~$
<img src="./investigation.jpg"/>