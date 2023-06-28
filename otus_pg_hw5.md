## OTUS PostgreSQL Cloud Solutions. ##

### Домашняя работа 5 ###

**Сделать бэкап PostgreSQL используя pg_probackup и выполнить восстановление.**

Инсталлировать PostgreSQL 14 и pg_probackup 14 во избежании ошибки несовместимости версий.
Пакет rpm pg_probackup нужной версии загрузить c http://repo.postgrespro.ru/pg_probackup/rpm/2.5.8/

Инсталлировать rpm на клиенте и на сервере.

``
sudo zypper in pg_probackup-14-2.5.8-1.9e9509d8aab21565d95b44daeafcca6b7516597c.x86_64.rpm
``

На клиенте создать каталог копий.

``
mkdir /home/uniadmin/backupDB
pg_probackup-14 init -B /home/uniadmin/backupDB
``

**На клиенте настроить удалённый режим.**

``
uniadmin@localhost:~>ssh-copy-id uniadmin@158.160.103.127
``

Для настройки непрерывного архивирования WAL на сервере PostgreSQL необходимо выполнить
ssh-copy-id postgres@<хост на клиенте pg_probackup> #IP локальной машины не публичный 
поэтому этот этап пропускаем.


На сервере, где запущен кластер PostgreSQL cоздать роль backup.
Можно использовать базу данных postgres а можно создать отдельную (рекомендуется)
В даннном случае используется БД postgres.

```
BEGIN;
CREATE ROLE backup WITH LOGIN;
GRANT USAGE ON SCHEMA pg_catalog TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.current_setting(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.set_config(text, text, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_is_in_recovery() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_start_backup(text, boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_stop_backup(boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_create_restore_point(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_switch_wal() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_last_wal_replay_lsn() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current_snapshot() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_snapshot_xmax(txid_snapshot) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_control_checkpoint() TO backup;
COMMIT;
```

 
**Добавить возможность резервного потокового копирования WAL (--stream).**

```
ALTER ROLE backup WITH REPLICATION;
```

В файле pg_hba.conf разрешить выполнение репликации для роли backup.

На клиенте добавить инстанс в репозиторий pg_probackup. 
Копию базы PostgreSQL  можно сделать только от имени того пользователя, который запускает сервер Postgres. 
Например, если сервер Postgres запускает пользователь postgres, команду backup также должен выполнять пользователь postgres.
Для удовлетворения этого требования в случае использования удалённого режима и SSH в параметре --remote-user необходимо передать postgres.

```
uniadmin@localhost:~> pg_probackup-14 add-instance -B /home/uniadmin/backupDB/ --instance=enshire --remote-host=158.
160.103.127 --remote-user=postgres --pgdata=/var/lib/pgsql/data
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa':
INFO: Instance 'enshire' successfully inited
```

**На клиенте запустить резервное копирование.**

```
uniadmin@localhost:~> pg_probackup-14 backup -B /home/uniadmin/backupDB --instance=enshire --backup-mode=FULL --comp
ress --stream --delete-expired --pguser=backup --remote-host=158.160.103.127 --remote-user=postgres

INFO: Backup start, pg_probackup version: 2.5.8, instance: enshire, backup ID: RWV59E, backup mode: FULL, wal mode:
STREAM, remote: true, compress-algorithm: zlib, compress-level: 1
Password for user backup:
INFO: This PostgreSQL instance was initialized with data block checksums. Data block corruption will be detected
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa':
INFO: Database backup start
INFO: wait for pg_start_backup()
INFO: Wait for WAL segment /home/uniadmin/backupDB/backups/enshire/RWV59E/database/pg_wal/000000010000000000000085 t
o be streamed
INFO: PGDATA size: 2665MB
INFO: Current Start LSN: 0/85000028, TLI: 1
INFO: Start transferring data files
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa':
INFO: Data files are transferred, time elapsed: 1m:43s
INFO: wait for pg_stop_backup()
INFO: pg_stop backup() successfully executed
INFO: stop_lsn: 0/8500BA68
INFO: Getting the Recovery Time from WAL
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 0
INFO: Validating backup RWV59E
INFO: Backup RWV59E data files are valid
INFO: Backup RWV59E resident size: 777MB
INFO: Backup RWV59E completed
WARNING: Retention policy is not set
uniadmin@localhost:~>
```

**Посмотреть статус резервного копирования.**

```
uniadmin@localhost:~/backupDB/backups/enshire> pg_probackup-14 show -B /home/uniadmin/backupDB/

BACKUP INSTANCE 'enshire'
=====================================================================================================================================   
Instance  Version  ID      Recovery Time           Mode  WAL Mode  TLI    Time   Data   WAL  Zratio  Start LSN   Stop LSN    Status    
=====================================================================================================================================   
enshire   14       RWV59E  2023-06-26 16:55:31+03  FULL  STREAM    1/0  1m:55s  761MB  16MB    3.50  0/85000028  0/8500BA68  OK        
enshire   ----     RWV57F  ----                    FULL  STREAM    0/0      4s      0     0    1.00  0/0         0/0         ERROR     
enshire   14       RWV515  ----                    FULL  STREAM    0/0     11s      0     0    1.00  0/0         0/0         ERROR     
enshire   14       RWV4RD  ----                    FULL  STREAM    1/0     25s      0     0    1.00  0/83000060  0/0         ERROR     
uniadmin@localhost:~/backupDB/backups/enshire>
```

Настроить retention policy

``
pg_probackup-14 set-config --instance enshire --retention-window=1 --retention-redundancy=2
``

**Создать таблицу для проверки создания полной копии.**

```
demo=# create table privileges (a serial, b char(1), c text);
CREATE TABLE
demo=# insert into privileges values ('1','A','INGIS MANIS');
INSERT 0 1
demo=# insert into privileges values ('2','A','INGIS MANIS');
INSERT 0 1
demo=# insert into privileges values ('2','D','LEOS MESSI');
INSERT 0 1

demo=# select * from privileges;
a | b |      c      
---+---+-------------
1 | A | INGIS MANIS
2 | A | INGIS MANIS
2 | D | LEOS MESSI
(3 rows)

demo=# \q
uniadmin@localhost:/home> 
```

**Выполнить резервное копирование.**

```
pg_probackup-14 backup -B /home/uniadmin/backupDB --instance=enshire --backup-mode=FULL --comp
ress --stream --delete-expired --pguser=backup --remote-host=158.160.103.127 --remote-user=postgres

Done...

uniadmin@localhost:~/backupDB/backups/enshire> pg_probackup-14 show -B /home/uniadmin/backupDB/

BACKUP INSTANCE 'enshire'
=====================================================================================================================================   
Instance  Version  ID      Recovery Time           Mode  WAL Mode  TLI    Time   Data   WAL  Zratio  Start LSN   Stop LSN    Status
=====================================================================================================================================   
enshire   14       RWV5VT  2023-06-26 17:08:47+03  FULL  STREAM    1/0  1m:44s  761MB  16MB    3.50  0/87000028  0/870001A0  OK        
enshire   14       RWV59E  2023-06-26 16:55:31+03  FULL  STREAM    1/0  1m:55s  761MB  16MB    3.50  0/85000028  0/8500BA68  OK        
enshire   ----     RWV57F  ----                    FULL  STREAM    0/0      4s      0     0    1.00  0/0         0/0         ERROR     
enshire   14       RWV515  ----                    FULL  STREAM    0/0     11s      0     0    1.00  0/0         0/0         ERROR     
enshire   14       RWV4RD  ----                    FULL  STREAM    1/0     25s      0     0    1.00  0/83000060  0/0         ERROR     
uniadmin@localhost:~/backupDB/backups/enshire>
```

**Создать таблицу для проверки метода копирования DELTA.**

```
uniadmin@localhost:/home> psql -d demo -h 158.160.103.127 -U uniadmin -W
Password:
psql (15.3, server 14.8)
Type "help" for help.

demo=# create table privileges3 (a int, b char(1), c text);
CREATE TABLE
demo=# insert into privileges3 values (1,'A','TEST TEST');
INSERT 0 1
demo=# insert into privileges3 values (2,'B','TEST1 TEST1');
INSERT 0 1
demo=# insert into privileges3 values (3,'D','TEST4 TEST4');
INSERT 0 1
demo=# insert into privileges3 values (5,'D','TEST5 TEST4');
INSERT 0 1
demo=# select * from privileges3
demo-# ;
a | b |      c      
---+---+-------------
1 | A | TEST TEST
2 | B | TEST1 TEST1
3 | D | TEST4 TEST4
5 | D | TEST5 TEST4
(4 rows)

demo=# \q
```

**Выполнить резервное копирование DELTA, предварительно выполнив полный бэкап - Parent backup: RWV7V1.**

```
uniadmin@localhost:/home> pg_probackup-14 backup -B /home/uniadmin/backupDB --instance=enshire --backup-mode=DELTA --compress --stream -
-delete-expired --pguser=backup --pgdatabase=demo --remote-host=158.160.103.127 --remote-user=postgres

uniadmin@localhost:/home> pg_probackup-14 backup -B /home/uniadmin/backupDB --instance=enshire --backup-mode=DELTA --compress --stream -
-delete-expired --pguser=backup --pgdatabase=demo --remote-host=158.160.103.127 --remote-user=postgres
INFO: Backup start, pg_probackup version: 2.5.8, instance: enshire, backup ID: RWV8RQ, backup mode: DELTA, wal mode: STREAM, remote: tru
e, compress-algorithm: zlib, compress-level: 1
Password for user backup:
INFO: This PostgreSQL instance was initialized with data block checksums. Data block corruption will be detected
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa':
INFO: Database backup start
INFO: wait for pg_start_backup()
INFO: Parent backup: RWV7V1
INFO: Wait for WAL segment /home/uniadmin/backupDB/backups/enshire/RWV8RQ/database/pg_wal/00000001000000000000008B to be streamed
INFO: PGDATA size: 2665MB
INFO: Current Start LSN: 0/8B000028, TLI: 1
INFO: Parent Start LSN: 0/89000028, TLI: 1
INFO: Start transferring data files
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa':
INFO: Data files are transferred, time elapsed: 17s
INFO: wait for pg_stop_backup()
INFO: pg_stop backup() successfully executed
INFO: stop_lsn: 0/8B0001A0
INFO: Getting the Recovery Time from WAL
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 1s
INFO: Validating backup RWV8RQ
INFO: Backup RWV8RQ data files are valid
INFO: Backup RWV8RQ resident size: 16MB
INFO: Backup RWV8RQ completed
INFO: Evaluate backups by retention
INFO: Backup RWV8RQ, mode: DELTA, status: OK. Redundancy: 1/2, Time Window: 0d/1d. Active
INFO: Backup RWV7V1, mode: FULL, status: OK. Redundancy: 1/2, Time Window: 0d/1d. Active
INFO: Backup RWV5VT, mode: FULL, status: OK. Redundancy: 2/2, Time Window: 0d/1d. Active
INFO: Backup RWV59E, mode: FULL, status: OK. Redundancy: 3/2, Time Window: 0d/1d. Active
INFO: Backup RWV57F, mode: FULL, status: ERROR. Redundancy: 4/2, Time Window: 0d/1d. Active
INFO: Backup RWV515, mode: FULL, status: ERROR. Redundancy: 4/2, Time Window: 0d/1d. Active
INFO: Backup RWV4RD, mode: FULL, status: ERROR. Redundancy: 4/2, Time Window: 0d/1d. Active
INFO: There are no backups to merge by retention policy
INFO: There are no backups to delete by retention policy
INFO: There is no WAL to purge by retention policy
uniadmin@localhost:/home>

uniadmin@localhost:/home> pg_probackup-14 show -B /home/uniadmin/backupDB/

BACKUP INSTANCE 'enshire'
======================================================================================================================================= 
Instance  Version  ID      Recovery Time           Mode   WAL Mode  TLI    Time    Data   WAL  Zratio  Start LSN   Stop LSN    Status
======================================================================================================================================= 
enshire   14       RWV8RQ  2023-06-26 18:09:52+03  DELTA  STREAM    1/1     29s   169kB  16MB    2.03  0/8B000028  0/8B0001A0  OK      
enshire   14       RWV7V1  2023-06-26 17:54:22+03  FULL   STREAM    1/0  4m:35s  2668MB  16MB    1.00  0/89000028  0/890001A0  OK      
enshire   14       RWV5VT  2023-06-26 17:08:47+03  FULL   STREAM    1/0  1m:44s   761MB  16MB    3.50  0/87000028  0/870001A0  OK      
enshire   14       RWV59E  2023-06-26 16:55:31+03  FULL   STREAM    1/0  1m:55s   761MB  16MB    3.50  0/85000028  0/8500BA68  OK      
enshire   ----     RWV57F  ----                    FULL   STREAM    0/0      4s       0     0    1.00  0/0         0/0         ERROR   
enshire   14       RWV515  ----                    FULL   STREAM    0/0     11s       0     0    1.00  0/0         0/0         ERROR   
enshire   14       RWV4RD  ----                    FULL   STREAM    1/0     25s       0     0    1.00  0/83000060  0/0         ERROR   
uniadmin@localhost:/home>
```

**Остановть сервис PostgreSQL и удалить $PGDATA. Восстановить один из бэкапов.**

```
uniadmin@localhost:~> pg_probackup-14 restore -B /home/uniadmin/backupDB/ --instance enshire --remote-user=postgres --remote-host=158.16
0.103.127 --archive-user=backup -i RWV59E
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Validating backup RWV59E
INFO: Backup RWV59E data files are valid
INFO: Backup RWV59E WAL segments are valid
INFO: Backup RWV59E is valid.
INFO: Restoring the database from backup at 2023-06-26 16:53:38+03
INFO: Start restoring backup files. PGDATA size: 2681MB
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Backup files are restored. Transfered bytes: 2673MB, time elapsed: 2m:3s
INFO: Restore incremental ratio (less is better): 100% (2673MB/2681MB)
INFO: Syncing restored files to disk
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Restored backup files are synced, time elapsed: 39s
INFO: Restore of backup RWV59E completed.
```

**Проверить таблицу privileges.**

```
postgres@endor-shire-dz5:/home/uniadmin> psql
psql (14.8)
Type "help" for help.

postgres=# \c demo
You are now connected to database "demo" as user "postgres".
demo=# select * from privileges
demo-# ;
 a | b |      c
---+---+-------------
 1 | A | INGIS MANIS
 2 | A | INGIS MANIS
 2 | D | LEOS MESSI
(3 rows)

demo=# select * from privileges3;
ERROR:  relation "privileges3" does not exist  # Это в другом бэкапе.
```

**Восстановить DELTA backup предварительно остановив PostgreSQL и удалив $PGDATA.**

```
uniadmin@localhost:~> pg_probackup-14 restore -B /home/uniadmin/backupDB -i RWV8RQ  --instance enshire --remote-user=postgres --remote-h
ost=158.160.110.47
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Validating parents for backup RWV8RQ                # DELTA бэкап
INFO: Validating backup RWV7V1                            # Родительский бэкап 
INFO: Backup RWV7V1 data files are valid
INFO: Validating backup RWV8RQ
INFO: Backup RWV8RQ data files are valid
INFO: Backup RWV8RQ WAL segments are valid
INFO: Backup RWV8RQ is valid.
INFO: Restoring the database from backup at 2023-06-26 18:09:26+03
INFO: Start restoring backup files. PGDATA size: 2681MB
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Backup files are restored. Transfered bytes: 2681MB, time elapsed: 2m:22s
INFO: Restore incremental ratio (less is better): 100% (2681MB/2681MB)
INFO: Syncing restored files to disk
Enter passphrase for key '/home/uniadmin/.ssh/id_rsa': 
INFO: Restored backup files are synced, time elapsed: 34s
INFO: Restore of backup RWV8RQ completed.
uniadmin@localhost:~> 
```

**Проверить наличие таблиц.**

```
uniadmin@endor-shire-dz5:~> sudo systemctl start postgresql
uniadmin@endor-shire-dz5:~> sudo su postgres
postgres@endor-shire-dz5:/home/uniadmin> psql
psql (14.8)
Type "help" for help.

postgres=# \c demo
You are now connected to database "demo" as user "postgres".
demo=# select * from privileges;
 a | b |      c
---+---+-------------
 1 | A | INGIS MANIS
 2 | A | INGIS MANIS
 2 | D | LEOS MESSI
(3 rows)

demo=# select * from privileges3;
 a | b |      c
---+---+-------------
 1 | A | TEST TEST
 2 | B | TEST1 TEST1
 3 | D | TEST4 TEST4
 5 | D | TEST5 TEST4
(4 rows)

demo=#
```









