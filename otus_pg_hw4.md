## OTUS PostgreSQL Cloud Solutions ##

### Домашняя работа 3 ###

**Развернуть инстанс Постгреса в YandexCloud**

![image](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/93aeff59-9869-4f7d-9363-f8d30fad050b)

**Протестировать sysbench**

Изначально настройки PostgreSQL выбраны по умолчанию.

Создать БД, которая будет использоваться для тестов.

```
create user sbtest with password 'sbtest';
create database sbtest;
grant all privileges on database sbtest to sbtest;
\c sbtest
grant all on schema public to sbtest;
\q 
```

Наполнить тестовую БД

```
sysbench \
--db-driver=pgsql \
--oltp-table-size=1000000 \
--oltp-tables-count=5 \
--threads=5 \
--pgsql-host=84.252.128.100 \
--pgsql-port=5432 \
--pgsql-user=sbtest \
--pgsql-password=sbtEst34! \
--pgsql-db=sbtest \
/usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua run
```

Запустить тестовый сценарий при  настройках по умолчанию на локальной машине. 
В дальнейшем тестовый сценарий запускаем на локальной машине и коннектимся по сети 
к СУБД. Запускаемый сценарий:

```
sysbench \
--db-driver=pgsql \
--report-interval=5 \
--threads=5 \
--pgsql-host=84.252.128.100 \ 
--pgsql-port=5432 \ 
--time=600 \ 
--pgsql-user=sbtest \ 
--pgsql-password=sbtEst34! \ 
--pgsql-db=sbtest \ 
--oltp-tables-count=5 \ 
/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua run
```

**Статистика сгенерированная sysbench при настройках по умолчанию**

```
SQL statistics:
queries performed:
read:                            184268
write:                           52603
other:                           26347
total:                           263218
transactions:                        13152  (21.91 per sec.)
queries:                             263218 (438.55 per sec.)
ignored errors:                      10     (0.02 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.1944s
total number of events:              13152

Latency (ms):
min:                                  189.33
avg:                                  228.12
max:                                 1688.95
95th percentile:                      292.60 - 95% запросов к БД имеют задержку меньше 292 ms
sum:                              3000233.05

Threads fairness:
events (avg/stddev):           2630.4000/49.80
execution time (avg/stddev):   600.0466/0.06
```

**Увеличили shared_buffers до 256**

```
SQL statistics:
queries performed:
read:                            180642
write:                           51578
other:                           25820
total:                           258040
transactions:                        12893  (21.48 per sec.)
queries:                             258040 (429.96 per sec.)
ignored errors:                      10     (0.02 per sec.)
reconnects:                          0      (0.00 per sec.)
```
```
General statistics:
total time:                          600.1497s
total number of events:              12893

Latency (ms):
min:                                  198.60
avg:                                  232.69
max:                                 1190.11
95th percentile:                      314.45 - 95% запросов к БД имеют задержку меньше 314 ms - как то не очень
sum:                              3000125.78

Threads fairness:
events (avg/stddev):           2578.6000/16.33
executi on time (avg/stddev):   600.0252/0.05
```

**Увеличили maintenance_work_mem**

```
postgres=# alter system set maintenance_work_mem='400MB';
ALTER SYSTEM
postgres=# show maintenance_work_mem 

postgres=# show maintenance_work_mem;
maintenance_work_mem
----------------------
400MB
(1 row)

SQL statistics:
queries performed:
read:                            184674
write:                           52727
other:                           26401
total:                           263802
transactions:                        13182  (21.96 per sec.)
queries:                             263802 (439.52 per sec.)
ignored errors:                      9      (0.01 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.2070s
total number of events:              13182

Latency (ms):
min:                                  190.51
avg:                                  227.60
max:                                  669.70
95th percentile:                      282.25 - 95% запросов к БД имеют задержку меньше 282 ms - получше.
sum:                              3000275.50

Threads fairness:
events (avg/stddev):           2636.4000/54.24
execution time (avg/stddev):   600.0551/0.06
```

**Увеличили work_mem**

```
postgres=# show work_mem ;
work_mem
----------
200MB
(1 row)

postgres=# show maintenance_work_mem ;
maintenance_work_mem
----------------------
200MB
(1 row)


postgres=# \q
uniadmin@endor-shire-dz4:~>

SQL statistics:
queries performed:
read:                            183834
write:                           52495
other:                           26281
total:                           262610
transactions:                        13126  (21.87 per sec.)
queries:                             262610 (437.52 per sec.)
ignored errors:                      5      (0.01 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.2289s
total number of events:              13126

Latency (ms):
min:                                  189.60
avg:                                  228.58
max:                                  490.96
95th percentile:                      262.64 - 95 % апросов к БД имеют задержку меньше 262 ms - получше.
sum:                              3000288.70

Threads fairness:
events (avg/stddev):           2625.2000/27.39
execution time (avg/stddev):   600.0577/0.07

uniadmin@localhost:~>
```

**Восстановили значения по умолчанию**

```
SQL statistics:
queries performed:
read:                            176568
write:                           50424
other:                           25238
total:                           252230
transactions:                        12608  (21.01 per sec.)
queries:                             252230 (420.24 per sec.)
ignored errors:                      4      (0.01 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.2038s
total number of events:              12608

Latency (ms):
min:                                  199.34
avg:                                  237.96
max:                                 1457.64
95th percentile:                      292.60  - 95 % апросов к БД имеют задержку меньше 292.60 ms
sum:                              3000261.39

Threads fairness:
events (avg/stddev):           2521.6000/25.76
execution time (avg/stddev):   600.0523/0.06
```

**Простое увеличение maintenance_work_mem для тестового сценария oltp.lua помогло а увеличение размера буфферного кэша - нет**

**Перенесли wal на диск SSD**

![image](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/877aab1c-6260-4893-af40-d1fc7972013a)

```
SQL statistics:
queries performed:
read:                            200270
write:                           57171
other:                           28633
total:                           286074
transactions:                        14292  (23.81 per sec.)
queries:                             286074 (476.69 per sec.)
ignored errors:                      13     (0.02 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.1293s
total number of events:              14292

Latency (ms):
min:                                  169.15
avg:                                  209.93
max:                                 1301.96
95th percentile:                      248.83 - 95 % запросов к БД имеют задержку меньше 248,83 ms - пока самый лучший результат.
sum:                              3000386.18

Threads fairness:
events (avg/stddev):           2858.4000/34.99
execution time (avg/stddev):   600.0772/0.02

uniadmin@localhost:~> 
```

**Проверить различные fsync для выбора оптимального wal_sync_method**

```
uniadmin@endor-shire-dz4:~> sudo pg_test_fsync -f /pgwals/test.log
5 seconds per test
O_DIRECT supported on this platform for open_datasync and open_sync.

Compare file sync methods using one 8kB write:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                       303.434 ops/sec    3296 usecs/op
        fdatasync                           249.186 ops/sec    4013 usecs/op
        fsync                               343.084 ops/sec    2915 usecs/op
        fsync_writethrough                              n/a
        open_sync                           221.075 ops/sec    4523 usecs/op

Compare file sync methods using two 8kB writes:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                       197.507 ops/sec    5063 usecs/op
        fdatasync                           244.557 ops/sec    4089 usecs/op
        fsync                               139.381 ops/sec    7175 usecs/op
        fsync_writethrough                              n/a
        open_sync                           108.410 ops/sec    9224 usecs/op

Compare open_sync with different write sizes:
(This is designed to compare the cost of writing 16kB in different write
open_sync sizes.)
         1 * 16kB open_sync write           164.676 ops/sec    6073 usecs/op
         2 *  8kB open_sync writes           99.070 ops/sec   10094 usecs/op
         4 *  4kB open_sync writes           53.803 ops/sec   18586 usecs/op
         8 *  2kB open_sync writes           18.331 ops/sec   54554 usecs/op
        16 *  1kB open_sync writes           10.860 ops/sec   92079 usecs/op

Test if fsync on non-write file descriptor is honored:
(If the times are similar, fsync() can sync data written on a different
descriptor.)
        write, fsync, close                 129.709 ops/sec    7710 usecs/op
        write, close, fsync                 199.088 ops/sec    5023 usecs/op

Non-sync'ed 8kB writes:
        write                           1497591.807 ops/sec       1 usecs/op
uniadmin@endor-shire-dz4:~> 


5 seconds per test
O_DIRECT supported on this platform for open_datasync and open_sync.

Compare file sync methods using one 8kB write:
(in wal_sync_method preference order, except fdatasync is Linux's default)
open_datasync                       281.729 ops/sec    3550 usecs/op
fdatasync                           359.587 ops/sec    2781 usecs/op
fsync                               200.318 ops/sec    4992 usecs/op
fsync_writethrough                              n/a
open_sync                           173.548 ops/sec    5762 usecs/op

Compare file sync methods using two 8kB writes:
(in wal_sync_method preference order, except fdatasync is Linux's default)
open_datasync                       108.391 ops/sec    9226 usecs/op
fdatasync                           277.877 ops/sec    3599 usecs/op
fsync                                85.746 ops/sec   11662 usecs/op
fsync_writethrough                              n/a
open_sync                            80.968 ops/sec   12351 usecs/op

Compare open_sync with different write sizes:
(This is designed to compare the cost of writing 16kB in different write
open_sync sizes.)
1 * 16kB open_sync write           125.365 ops/sec    7977 usecs/op
2 *  8kB open_sync writes           61.933 ops/sec   16147 usecs/op
4 *  4kB open_sync writes           53.143 ops/sec   18817 usecs/op
8 *  2kB open_sync writes           27.954 ops/sec   35774 usecs/op
16 *  1kB open_sync writes            8.385 ops/sec  119260 usecs/op

Test if fsync on non-write file descriptor is honored:
(If the times are similar, fsync() can sync data written on a different
descriptor.)
write, fsync, close                 194.204 ops/sec    5149 usecs/op
write, close, fsync                 131.505 ops/sec    7604 usecs/op

Non-sync'ed 8kB writes:
write                           1358148.069 ops/sec       1 usecs/op
uniadmin@endor-shire-dz4:~> 
```

**fdatasync является оптимальным выбором. Можно вообще fsync отключить - это плохая привычка а мы с ними боремся**

**Увеличить max_wal_senders**

```
SQL statistics:
queries performed:
read:                            209510
write:                           59817
other:                           29957
total:                           299284
transactions:                        14957  (24.92 per sec.)
queries:                             299284 (498.67 per sec.)
ignored errors:                      8      (0.01 per sec.)
reconnects:                          0      (0.00 per sec.)

General statistics:
total time:                          600.1619s
total number of events:              14957

Latency (ms):
min:                                  165.92
avg:                                  200.60
max:                                 1223.78
95th percentile:                      231.53 - Пока наилучшее значение на этом остановимся
sum:                              3000338.08

Threads fairness:
events (avg/stddev):           2991.4000/76.44
execution time (avg/stddev):   600.0676/0.06

uniadmin@localhost:~>
```


