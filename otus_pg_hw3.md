## OTUS PostgreSQL Cloud Solutions ##

### Домашняя работа 3 ###

**Сделать инстанс с OpenSUSe Leap 15.4**

![image](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/2c30e992-1283-4ba4-9ba4-275bc80b7b4b)


**Поставить PostgreSQL 15**

https://en.opensuse.org/SDB:PostgreSQL

```
zypper --non-interactive --quiet addrepo --refresh -p 90  http://download.opensuse.org/repositories/server:database:postgresql/openSUSE_Tumbleweed/ PostgreSQL

uniadmin@endor-shire-dz3:~> sudo zypper --non-interactive --quiet addrepo --refresh -p 90  http://download.opensuse.org/repositories/server:database:postgresql/openSUSE_Tumbleweed/ PostgreSQL
90 (raised priority)  :  1 repository
99 (default priority) :  5 repositories
uniadmin@endor-shire-dz3:~> sudo zypper refresh

New repository or package signing key received:

Repository:       PostgreSQL
Key Fingerprint:  116E B863 3158 3E47 E63C DF4D 5621 11AC 0590 5EA8
Key Name:         server:database OBS Project <server:database@build.opensuse.org>
Key Algorithm:    DSA 1024
Key Created:      Sun 23 May 2021 02:05:45 PM MSK
Key Expires:      Tue 01 Aug 2023 02:05:45 PM MSK (expires in 60 days)
Rpm Name:         gpg-pubkey-05905ea8-60aa3709

    Note: Signing data enables the recipient to verify that no modifications occurred after the data
    were signed. Accepting data with no, wrong or unknown signature can lead to a corrupted system
    and in extreme cases even to a system compromise.

    Note: A GPG pubkey is clearly identified by its fingerprint. Do not rely on the key's name. If
    you are not sure whether the presented key is authentic, ask the repository provider or check
    their web site. Many providers maintain a web page showing the fingerprints of the GPG keys they
    are using.

Do you want to reject the key, trust temporarily, or trust always? [r/t/a/?] (r): a
Retrieving repository 'PostgreSQL' metadata ..................................................................................................................................................................................[done]
Building repository 'PostgreSQL' cache .......................................................................................................................................................................................[done]
Repository 'openSUSE-Leap-15.3-2' is up to date.
Repository 'openSUSE-Leap-15.3-Update' is up to date.
Repository 'Update repository of openSUSE Backports' is up to date.
Repository 'Update repository with updates from SUSE Linux Enterprise 15' is up to date.
Repository 'repo-update-non-oss' is up to date.
All repositories have been refreshed.
uniadmin@endor-shire-dz3:~>
```

**Проверить что кластер запущен.**

```
uniadmin@endor-shire-dz3:~> sudo systemctl status postgresql
● postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
Active: active (running) since Thu 2023-06-01 14:25:07 MSK; 1h 14min ago
Process: 2253 ExecStart=/usr/share/postgresql/postgresql-script start (code=exited, status=0/SUCCESS)
Main PID: 2270 (postgres)
Tasks: 7 (limit: 4915)
CGroup: /system.slice/postgresql.service
├─2270 /usr/lib/postgresql15/bin/postgres -D /var/lib/pgsql/data
├─2271 postgres: logger
├─2272 postgres: checkpointer
├─2273 postgres: background writer
├─2275 postgres: walwriter
```

**Из под пользователя postgres в psql сделать произвольную таблицу с произвольным содержимым**

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo su postgres
postgres@fhmo8heg5f5s5htvi4v7:/home/uniadmin> psql
psql (15.3)
Type "help" for help.

postgres=# select version();
version
--------------------------------------------------------------------------------------
PostgreSQL 15.3 on x86_64-suse-linux-gnu, compiled by gcc (SUSE Linux) 7.5.0, 64-bit
(1 row)

postgres=# create table test1 (a int, b char(20), c text);
CREATE TABLE
postgres=# insert into test1 values (1,'a','aaaaaaaaa');
INSERT 0 1
postgres=# insert into test1 values (2,'b','bbbbbbbbb');
INSERT 0 1
postgres=# insert into test1 values (3,'c','ccccccccc');
INSERT 0 1
postgres=# select * from test1;
a |          b           |     c     
---+----------------------+-----------
1 | a                    | aaaaaaaaa
2 | b                    | bbbbbbbbb
3 | c                    | ccccccccc
(3 rows)

postgres=#
```

**Остановить postgres.**

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo systemctl stop postgresql

uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo systemctl status postgresql
○ postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
Drop-In: /etc/systemd/system/postgresql.service.d
└─override.conf
Active: inactive (dead) since Fri 2023-06-02 11:11:17 MSK; 10s ago
Process: 1376 ExecStart=/usr/share/postgresql/postgresql-script start (code=exited, status=0/SUCCESS)
Process: 1551 ExecStop=/usr/share/postgresql/postgresql-script stop (code=exited, status=0/SUCCESS)
Main PID: 1386 (code=exited, status=0/SUCCESS)
```

**Создать новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс 
размером например 10GB - или аналог в другом облаке/виртуализации.
Добавить свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и 
дальше выбрать пункт attach existing disk.**

![image](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/309a1ef0-6702-45c3-902b-bcc1715c3c09)

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
vda    254:0    0  15G  0 disk
├─vda1 254:1    0   4M  0 part
└─vda2 254:2    0  15G  0 part /
vdb    254:16   0  10G  0 disk
uniadmin@fhmo8heg5f5s5htvi4v7:~>
```

**Проинициализировать диск и подмонтировать файловую систему.**

```
uniadmin@endor-shire-dz3:~> sudo yast partitioner
Используется предложенный GUI для создания раздела на диске.

uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo yast partitioner

uniadmin@fhmo8heg5f5s5htvi4v7:~> lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
vda    254:0    0  15G  0 disk
├─vda1 254:1    0   4M  0 part
└─vda2 254:2    0  15G  0 part /
vdb    254:16   0  10G  0 disk
└─vdb1 254:17   0  10G  0 part /mnt/data

uniadmin@fhmo8heg5f5s5htvi4v7:~>
```

**Перезагрузить инстанс и убедится, что диск остается примонтированным.**

Перезапуск виртуальной машины.

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
vda    254:0    0  15G  0 disk            
├─vda1 254:1    0   4M  0 part            
└─vda2 254:2    0  15G  0 part /          
vdb    254:16   0  10G  0 disk            
└─vdb1 254:17   0  10G  0 part /mnt/data  

uniadmin@fhmo8heg5f5s5htvi4v7:~> cat /etc/fstab
UUID=c0ccac5a-80dd-4b1d-8d69-07b06bd139a4  /          ext4  acl,user_xattr  0  1
UUID=443499dc-8a2a-45d9-ac3f-ce1dd6d653ef  /mnt/data  xfs   defaults        0  0

uniadmin@fhmo8heg5f5s5htvi4v7:~>
```

**Cделать пользователя postgres владельцем /mnt/data/15_1.**

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo chown -R postgres:postgres /mnt/data/15_1
```

**Перенести содержимое /var/lib/pgsql/ в /mnt/data/15_1.**

Использовался 'Midnight Commander' и клавиша F6

```
uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data/15_1> sudo ls -la /var/lib/pgsql
total 28
drwxr-x---  2 postgres postgres 4096 Jun  2 11:18 .
drwxr-xr-x 24 root     root     4096 Jun  1 23:21 ..
-rw-------  1 postgres postgres  199 Jun  2 11:11 .bash_history
-rw-r-----  1 postgres postgres  192 Jun  1 11:15 .bash_profile
-rw-r--r--  1 postgres postgres  905 Jun  2 00:02 initlog
-rw-------  1 postgres postgres   20 Jun  2 10:59 .lesshst
-rw-------  1 postgres postgres  813 Jun  2 11:11 .psql_history

uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data/15_1> ls -la /mnt/data/15_1/
total 68
drwxr-xr-x 20 postgres postgres  4096 Jun  2 11:23 .
drwxr-xr-x  3 postgres postgres    18 Jun  2 11:14 ..
drwx------  5 postgres postgres    33 Jun  2 00:02 base
-rw-------  1 postgres postgres    44 Jun  2 11:04 current_logfiles
drwx------  2 postgres postgres  4096 Jun  2 11:05 global
drwx------  2 postgres postgres   286 Jun  2 11:04 log
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_commit_ts
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_dynshmem
-rw-------  1 postgres postgres  4545 Jun  2 00:01 pg_hba.conf
-rw-------  1 postgres postgres  1636 Jun  2 00:01 pg_ident.conf
drwx------  4 postgres postgres    68 Jun  2 11:11 pg_logical
drwx------  4 postgres postgres    36 Jun  2 00:01 pg_multixact
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_notify
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_replslot
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_serial
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_snapshots
drwx------  2 postgres postgres    25 Jun  2 11:11 pg_stat
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_stat_tmp
drwx------  2 postgres postgres    18 Jun  2 00:01 pg_subtrans
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_tblspc
drwx------  2 postgres postgres     6 Jun  2 00:01 pg_twophase
-rw-------  1 postgres postgres     3 Jun  2 00:01 PG_VERSION
drwx------  3 postgres postgres    60 Jun  2 00:01 pg_wal
drwx------  2 postgres postgres    18 Jun  2 00:01 pg_xact
-rw-------  1 postgres postgres    88 Jun  2 00:01 postgresql.auto.conf
-rw-------  1 postgres postgres 29515 Jun  2 00:01 postgresql.conf
-rw-------  1 postgres postgres    62 Jun  2 11:04 postmaster.opts
```

**Запустить кластер Postgres.**

```
uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo systemctl start postgresql

uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo systemctl status postgresql
● postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
Drop-In: /etc/systemd/system/postgresql.service.d
└─override.conf
Active: active (running) since Fri 2023-06-02 11:26:43 MSK; 8s ago
Process: 1803 ExecStart=/usr/share/postgresql/postgresql-script start (code=exited, status=0/SUCCESS)
Main PID: 1820 (postgres)
Tasks: 7 (limit: 4915)
CGroup: /system.slice/postgresql.service
├─ 1820 /usr/lib/postgresql15/bin/postgres -D /var/lib/pgsql/data
├─ 1821 "postgres: logger " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 1822 "postgres: checkpointer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 1823 "postgres: background writer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 1825 "postgres: walwriter " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 1826 "postgres: autovacuum launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
└─ 1827 "postgres: logical replication launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""

uniadmin@fhmo8heg5f5s5htvi4v7:~> sudo su postgres

postgres@fhmo8heg5f5s5htvi4v7:/home/uniadmin> psql
psql (15.3)
Type "help" for help.

postgres=# select * from test1;
ERROR:  relation "test1" does not exist
LINE 1: select * from test1;
^
postgres=#
```

**Получилось или нет запустить кластер.**

Запустить сервис получилось, но видимо это не тот кластер а новый, **переинициализированный.**

**Найти конфигурационный параметр который надо поменять и поменять его.**

Для настройки сервиса postgresql на новую $PGDATA меняем следующие параметры в файле: **/etc/sysconfig/postgresql**

POSTGRES_DATADIR="/mnt/data/15_1"

#POSTGRES_INITDB_OPTS="--auth=ident" <- Данную строку необходимо закомментировать иначе кластер БД переинициализируется если не найдёт новый каталог с $PGDATA

**Запустить кластер.**

```
uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> sudo systemctl start postgresql
uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> sudo systemctl status postgresql

* postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
Drop-In: /etc/systemd/system/postgresql.service.d
└─override.conf
Active: failed (Result: exit-code) since Fri 2023-06-02 15:12:49 MSK; 23s ago
Process: 4658 ExecStart=/usr/share/postgresql/postgresql-script start (code=exited, status=1/FAILURE)

Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 systemd[1]: Starting PostgreSQL database server...
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 postgresql-script[4668]: 2023-06-02 15:12:49.684 MSK   [4668]FATAL:  data directory "/mnt/data/15_1" has invalid permissions   
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 postgresql-script[4668]: 2023-06-02 15:12:49.684 MSK   [4668]DETAIL:  Permissions should be u=rwx (0700) or u=rwx,g=rx (0750).
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 postgresql-script[4666]: pg_ctl: could not start server
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 postgresql-script[4666]: Examine the log output.
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 systemd[1]: postgresql.service: Control process exited, code=exited, status=1/FAILURE
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 systemd[1]: postgresql.service: Failed with result 'exit-code'.
Jun 02 15:12:49 fhmo8heg5f5s5htvi4v7 systemd[1]: Failed to start PostgreSQL database server.

uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data/15_1> sudo chmod 0750 /mnt/data/15_1

uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> ls -la
total 8
drwxr-xr-x  3 postgres postgres   18 Jun  2 11:14 .
drwxr-xr-x  3 postgres postgres 4096 Jun  1 23:44 ..
drwxr-x--- 20 postgres postgres 4096 Jun  2 15:03 15_1

uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> sudo systemctl start postgresql

uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> systemctl status postgresql.service
● postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
Drop-In: /etc/systemd/system/postgresql.service.d
└─override.conf
Active: active (running) since Fri 2023-06-02 15:15:07 MSK; 6s ago
Process: 4682 ExecStart=/usr/share/postgresql/postgresql-script start (code=exited, status=0/SUCCESS)
Main PID: 4692 (postgres)
Tasks: 7 (limit: 4915)
CGroup: /system.slice/postgresql.service
├─ 4692 /usr/lib/postgresql15/bin/postgres -D /mnt/data/15_1
├─ 4693 "postgres: logger " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 4694 "postgres: checkpointer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 4695 "postgres: background writer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 4697 "postgres: walwriter " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
├─ 4698 "postgres: autovacuum launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
└─ 4699 "postgres: logical replication launcher " "" "" "" "" "" "" "" "" "" "" "" "" ""
uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data>
```

**Зайти через через psql и проверить содержимое ранее созданной таблицы.**

```
uniadmin@fhmo8heg5f5s5htvi4v7:/mnt/data> sudo su postgres

postgres@fhmo8heg5f5s5htvi4v7:/mnt/data> psql
psql (15.3)
Type "help" for help.

postgres=# select * from test1;
a |          b           |     c
---+----------------------+-----------
1 | a                    | aaaaaaaaa
2 | b                    | bbbbbbbbb
3 | c                    | ccccccccc
(3 rows)

postgres=#

postgres=# show data_directory;
data_directory
----------------
/mnt/data/15_1
(1 row)
```


**Задание со звездочкой \*: не удаляя существующий GCE инстанс/ЯО сделайте новый, поставьте на его PostgreSQL, 
удалите файлы с данными из /var/lib/postgresql, перемонтируйте внешний диск который сделали ранее от 
первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с 
данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.**


