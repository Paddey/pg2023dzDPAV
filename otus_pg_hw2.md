Сделать в GCE/ЯО/Аналоги инстанс с OpenSUSe Leap 15.3. 

![image](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/1a2f642c-aa60-4cbf-8db0-7085a8ec777b)

Поставить на нем Docker Engine

Официально докер доступен для OpenSUSE Leap 15.3 и Tumbleweed
Инсталлировал как написано в https://en.opensuse.org/Docker
Для openSUSE инсталлируется python3-docker-compose

```
uniadmin@endor-shire-dz2:~> sudo zypper in docker
uniadmin@endor-shire-dz2:~> sudo zypper in docker-compose 
uniadmin@endor-shire-dz2:~> sudo systemctl enable docker
uniadmin@endor-shire-dz2:~> sudo usermod -G docker -a $USER
uniadmin@endor-shire-dz2:~> sudo systemctl restart docker
```

Проверить установку

```
uniadmin@endor-shire-dz2:~> docker version

Client: 
Version:           20.10.17-ce 
API version:       1.41 
Go version:        go1.17.13 
Git commit:        a89b84221c85 
Built:             Wed Jun 29 12:00:00 2022 
OS/Arch:           linux/amd64 
Context:           default 
Experimental:      true 

Server: 
Engine: 
Version:          20.10.17-ce 
API version:      1.41 (minimum version 1.12) 
Go version:       go1.17.13 
Git commit:       a89b84221c85 
Built:            Wed Jun 29 12:00:00 2022 
OS/Arch:          linux/amd64 
Experimental:     false 
containerd: 
Version:          v1.6.12 
GitCommit:        a05d175400b1145e5e6a735a6710579d181e7fb0 
runc: 
Version:          1.1.4 
GitCommit:        v1.1.4-0-ga916309fff0f 
docker-init: 
Version:          0.1.7_catatonit 
GitCommit: 
```

Создать сеть для коммуникации между контейнерами.

```
uniadmin@endor-shire-dz2:~> docker network create \
> --driver=bridge \
> --subnet=192.168.1.0/24 \
> --ip-range=192.168.1.0/24 \
> --gateway=192.168.1.1 \
> pg_dz2

bf3b61615b9b1b156ec88687defb913869e902bc865d6ee8882067251cb39c5b
```

Проверить, что сеть создана.

```
uniadmin@endor-shire-dz2:~> docker network ls

NETWORK ID     NAME      DRIVER    SCOPE 
d5909104ef11   bridge    bridge    local 
0b04a41ae318   host      host      local 
560e4f57306f   none      null      local 
bf3b61615b9b   pg_dz2    bridge    local

uniadmin@endor-shire-dz2:~>
```

Cделать каталог /var/lib/postgres

```
uniadmin@endor-shire-dz2:~> sudo mkdir /var/lib/postgres
uniadmin@endor-shire-dz2:~> sudo ls -la /var/lib/postgres

total 8
drwxr-xr-x  2 root root 4096 May 29 14:09 .
drwxr-xr-x 25 root root 4096 May 29 14:09 ..

uniadmin@endor-shire-dz2:~>
```

Pазвернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres

```
uniadmin@endor-shire-dz2:~> docker pull postgres:14.8
14.8: Pulling from library/postgres
f03b40093957: Already exists # Связано с предыдущими установками docker.
9d674c93414d: Already exists
de781e8e259a: Already exists
5ea6efaf51f6: Already exists
b078d5f4ac82: Already exists
97f84fb2a918: Already exists
5a6bf2f43fb8: Already exists
f1a40e88fea4: Already exists
e3826fb7af2a: Pull complete
ff1099d10196: Pull complete
67b873b24c5f: Pull complete
6e3ab17f013c: Pull complete
45f5f0ba2b88: Pull complete
Digest: sha256:af8b776d4b97636bc4844075d4d6e93df1eed120d4817cafda9000de24f1bd8a
Status: Downloaded newer image for postgres:14.8
docker.io/library/postgres:14.8
```

Проверить, что docker image pulled.

```
uniadmin@endor-shire-dz2:~> docker images

REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
postgres     14.8      ab8f641699b4   6 days ago   377MB

uniadmin@endor-shire-dz2:~>
```

Развернуть контейнер с клиентом postgres. Есть несколько вариантов:

1) Собрать свой docker image
2) Использовать стороний docker image. docker search postgresl-client
3) Использовать текущий docker image, но тогда будет две базы.
4) Использовать официальный docker image pgAdmin4 - выбран наиболее простой путь.

```
uniadmin@endor-shire-dz2:~> docker pull dpage/pgadmin4
Using default tag: latest
latest: Pulling from dpage/pgadmin4
f56be85fc22e: Pull complete
71351bf65ca8: Pull complete
24b9dc0be287: Pull complete
95702eafd1e6: Pull complete
18a87b29c6ef: Pull complete
906461613a94: Pull complete
cc3a25492ccd: Pull complete
00df49b7a51c: Pull complete
24181d9b86df: Pull complete
f77380753073: Pull complete
3b3f7d547196: Pull complete
16138b24ecdf: Pull complete
8c35897ae279: Pull complete
90864806e2cc: Pull complete


Digest: sha256:c4f63d42fb10e31a797ea1bfde15cb6e601372b2c39da7429294bed5e70c84f8

Status: Downloaded newer image for dpage/pgadmin4:latest
docker.io/dpage/pgadmin4:latest

uniadmin@endor-shire-dz2:~>
```

Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

Запустить контейнер с Postgres

```
docker run -itd \
--name postgres_dz2 \
--network=pg_dz2 \
-e POSTGRES_PASSWORD=trendoff \
-e PGDATA=/var/lib/postgres/148_1/data \
-v /var/lib/postgres:/var/lib/postgres/148_1/data \
-p 5432:5432 \
postgres:14.8
```

Убедится, что  контейнер запущен.

```
uniadmin@endor-shire-dz2:~> docker container ls

CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                       NAMES
68a50787b490   postgres:14.8   "docker-entrypoint.s…"   7 seconds ago   Up 7 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres_dz2
```

Убедится, что  PGDATA смотрит в подключённую директорию

```
uniadmin@endor-shire-dz2:~> sudo ls -latr /var/lib/postgres

total 136
drwxr-xr-x 25 root root  4096 May 29 20:15 ..
-rw-------  1  999  999     3 May 29 20:15 PG_VERSION
drwx------  2  999  999  4096 May 29 20:15 pg_twophase
drwx------  2  999  999  4096 May 29 20:15 pg_tblspc
drwx------  2  999  999  4096 May 29 20:15 pg_snapshots
drwx------  2  999  999  4096 May 29 20:15 pg_serial
drwx------  2  999  999  4096 May 29 20:15 pg_replslot
drwx------  2  999  999  4096 May 29 20:15 pg_notify
drwx------  4  999  999  4096 May 29 20:15 pg_multixact
drwx------  2  999  999  4096 May 29 20:15 pg_dynshmem
drwx------  2  999  999  4096 May 29 20:15 pg_commit_ts
-rw-------  1  999  999 28835 May 29 20:15 postgresql.conf
-rw-------  1  999  999    88 May 29 20:15 postgresql.auto.conf
-rw-------  1  999  999  1636 May 29 20:15 pg_ident.conf
drwx------  2  999  999  4096 May 29 20:15 pg_xact
drwx------  3  999  999  4096 May 29 20:15 pg_wal
drwx------  2  999  999  4096 May 29 20:15 pg_subtrans
drwx------  5  999  999  4096 May 29 20:15 base
-rw-------  1  999  999  4821 May 29 20:15 pg_hba.conf
drwx------ 19  999 root  4096 May 29 20:15 .
-rw-------  1  999  999    36 May 29 20:15 postmaster.opts
-rw-------  1  999  999    98 May 29 20:15 postmaster.pid
drwx------  2  999  999  4096 May 29 20:15 pg_stat
drwx------  2  999  999  4096 May 29 20:16 global
drwx------  4  999  999  4096 May 29 20:20 pg_logical
drwx------  2  999  999  4096 May 29 20:31 pg_stat_tmp

uniadmin@endor-shire-dz2:~>
```
UID 999 GUID 999 - это UID пользователя postgres внутри контейнера. При инициализации БД, владельцем каталога /var/lib/postgresql/ 
назначается пользователь postgres. Когда маппится каталог в текущий, то все рекурсивно будет принадлежать 999 UID'у. 
Это логичное и ожидаемое поведение.

Запустить контейнер с клиентом.

```
uniadmin@endor-shire-dz2:~> docker run -p 80:80 \
> --name pgadmin_dz2 \
> --network=pg_dz2 \
> -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' \
> -e 'PGADMIN_DEFAULT_PASSWORD=trendon' \
> -d dpage/pgadmin4

4b3522b6be5dd6356b3b9a5f12f1c36c94ea398017e4135cce7db04d790ee166
```

Проверить, что  контейнер с клиентом запустился

```
uniadmin@endor-shire-dz2:~> docker container ls
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                        NAMES
4b3522b6be5d   dpage/pgadmin4   "/entrypoint.sh"         9 seconds ago    Up 8 seconds    0.0.0.0:80->80/tcp, :::80->80/tcp, 443/tcp   pgadmin_dz2
68a50787b490   postgres:14.8    "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp    postgres_dz2
uniadmin@endor-shire-dz2:~>
```

Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

Из браузера на локальной машине http://62.84.115.193:80 - что не очень то безопасно :(

![pgadmin0](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/28f85a0f-bb23-4087-b50f-e77580de6932)

Настроить соединение с контейнером БД 192.168.1.2 

![pgadmin2](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/b8b82513-795b-47b0-9eca-e0321b82f33c)

Создать БД stores. Создать таблицу t1 в схеме public.

![pgadmin4](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/3be40f47-c044-4fc4-a024-cf4573758f03)

Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/Аналоги

```
ftpbox1@localhost:~> psql -d postgres -h 62.84.115.193 -U postgres -W
Password:
psql (15.2, server 14.8 (Debian 14.8-1.pgdg110+1))
Type "help" for help.

postgres=# \c stores
Password:
psql (15.2, server 14.8 (Debian 14.8-1.pgdg110+1))            
You are now connected to database "stores" as user "postgres".
stores=# select * from public.t1;
a |          b           | c  
---+----------------------+----
1 | a                    | ab
2 | b                    | bc
3 | c                    | cd
(3 rows)

stores=#
```

Удалить контейнер с сервером

```
uniadmin@endor-shire-dz2:~> docker container ls
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                        NAMES
4b3522b6be5d   dpage/pgadmin4   "/entrypoint.sh"         20 minutes ago   Up 20 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp, 443/tcp   pgadmin_dz2
68a50787b490   postgres:14.8    "docker-entrypoint.s…"   34 minutes ago   Up 34 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp    postgres_dz2

uniadmin@endor-shire-dz2:~> docker container stop postgres_dz2
postgres_dz2

uniadmin@endor-shire-dz2:~> docker container rm  postgres_dz2
postgres_dz2

uniadmin@endor-shire-dz2:~> docker container ls

CONTAINER ID   IMAGE            COMMAND            CREATED          STATUS          PORTS                                        NAMES
4b3522b6be5d   dpage/pgadmin4   "/entrypoint.sh"   22 minutes ago   Up 22 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp, 443/tcp   pgadmin_dz2
uniadmin@endor-shire-dz2:~>
```

Проверить, что данные остались в смонтированной папке.

```
uniadmin@endor-shire-dz2:~> sudo ls -la /var/lib/postgres
total 132
drwx------ 19  999 root  4096 May 29 20:50 .
drwxr-xr-x 25 root root  4096 May 29 20:15 ..
drwx------  6  999  999  4096 May 29 20:41 base
drwx------  2  999  999  4096 May 29 20:41 global
drwx------  2  999  999  4096 May 29 20:15 pg_commit_ts
drwx------  2  999  999  4096 May 29 20:15 pg_dynshmem
-rw-------  1  999  999  4821 May 29 20:15 pg_hba.conf
-rw-------  1  999  999  1636 May 29 20:15 pg_ident.conf
drwx------  4  999  999  4096 May 29 20:50 pg_logical
drwx------  4  999  999  4096 May 29 20:15 pg_multixact
drwx------  2  999  999  4096 May 29 20:15 pg_notify
drwx------  2  999  999  4096 May 29 20:15 pg_replslot
drwx------  2  999  999  4096 May 29 20:15 pg_serial
drwx------  2  999  999  4096 May 29 20:15 pg_snapshots
drwx------  2  999  999  4096 May 29 20:50 pg_stat
drwx------  2  999  999  4096 May 29 20:50 pg_stat_tmp
drwx------  2  999  999  4096 May 29 20:15 pg_subtrans
drwx------  2  999  999  4096 May 29 20:15 pg_tblspc
drwx------  2  999  999  4096 May 29 20:15 pg_twophase
-rw-------  1  999  999     3 May 29 20:15 PG_VERSION
drwx------  3  999  999  4096 May 29 20:15 pg_wal
drwx------  2  999  999  4096 May 29 20:15 pg_xact
-rw-------  1  999  999    88 May 29 20:15 postgresql.auto.conf
-rw-------  1  999  999 28835 May 29 20:15 postgresql.conf
-rw-------  1  999  999    36 May 29 20:15 postmaster.opts

uniadmin@endor-shire-dz2:~>
```

Создать контейнер заново

```
uniadmin@endor-shire-dz2:~> docker run -itd \
> --name postgres_dz2 \
> --network=pg_dz2 \
> -e POSTGRES_PASSWORD=trendoff \
> -e PGDATA=/var/lib/postgres/148_1/data \
> -v /var/lib/postgres:/var/lib/postgres/148_1/data \
> -p 5432:5432 \
> postgres:14.8

3cc72eec17c86bf4ef13151720db11316b49a118e22f2ae56b763d1d69ac85e7

uniadmin@endor-shire-dz2:~> docker container ls
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                        NAMES
3cc72eec17c8   postgres:14.8    "docker-entrypoint.s…"   17 seconds ago   Up 15 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp    postgres_dz2
4b3522b6be5d   dpage/pgadmin4   "/entrypoint.sh"         25 minutes ago   Up 25 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp, 443/tcp   pgadmin_dz2

uniadmin@endor-shire-dz2:~>
```

Подключиться из контейнера клиента к контейнеру серверу

![pgadmin5](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/4ba4da20-64ab-469d-b063-e000a25eaeef)

Выполнить запрос к таблице t1 из схемы public

![pgadmin6](https://github.com/Paddey/pg2023dzDPAV/assets/36312830/0bb28900-346b-4512-87a4-b626a86a6f4a)

Подключится снова из контейнера с клиентом к контейнеру с сервером

```
ftpbox1@localhost:~> psql -d postgres -h 62.84.115.193 -U postgres -W
Password:
psql (15.2, server 14.8 (Debian 14.8-1.pgdg110+1))
Type "help" for help.

postgres=# \l
List of databases
Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
stores    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
|          |          |            |            |            |                 | postgres=CTc/postgres
template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
|          |          |            |            |            |                 | postgres=CTc/postgres
(4 rows)

postgres=# \c stores
Password:
psql (15.2, server 14.8 (Debian 14.8-1.pgdg110+1))
You are now connected to database "stores" as user "postgres".
stores=# select * froom t1;
ERROR:  syntax error at or near "froom"
LINE 1: select * froom t1;
^
stores=# select * from t1;
a |          b           | c  
---+----------------------+----
1 | a                    | ab
2 | b                    | bc
3 | c                    | cd
(3 rows)

stores=#
```

Данные остались на месте...
