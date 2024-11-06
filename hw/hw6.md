> Развернуть асинхронную реплику (можно использовать 1 ВМ, просто рядом кластер развернуть и подключиться через localhost): ❖ тестируем производительность по сравнению с сингл инстансом
```shell
sudo su postgres
   wget https://storage.googleapis.com/thaibus/thai_small.tar.gz
   tar -xf thai_small.tar.gz
   psql -U postgres < thai.sql
```

Создаю юзера как в лекции для репликации
```shell
postgres@test1:/home/popova.valeriya21$    psql -c "CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD '***';"

CREATE ROLE
```
Создаю слот для устойчивости
```shell
postgres@test1:/home/popova.valeriya21$    psql -p 5432 -c "SELECT pg_create_physical_replication_slot('test');"
 pg_create_physical_replication_slot
-------------------------------------
 (test,)
(1 row)
```

```shell
su - pg_createcluster 15 main2

pg_basebackup -h localhost -p 5432 -U replicator -R -S test -D /var/lib/postgresql/15/main2

postgres@test1:/home/popova.valeriya21$    pg_ctlcluster 15 main2 start

Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-main2
postgres@test1:/home/popova.valeriya21$    pg_lsclusters

Ver Cluster Port Status          Owner    Data directory               Log file
15  main    5432 online          postgres /var/lib/postgresql/15/main  /var/log/postgresql/postgresql-15-main.log
15  main2   5433 online,recovery postgres /var/lib/postgresql/15/main2 /var/log/postgresql/postgresql-15-main2.log
```

Нагрузка на запись
```shell
postgres@test1:/home/popova.valeriya21$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 3408
number of failed transactions: 0 (0.000%)
latency average = 23.466 ms
initial connection time = 9.329 ms
tps = 340.911819 (without initial connection time)
```
Нагрузка на чтение
```shell
postgres@test1:/home/popova.valeriya21$ cat > ~/workload.sql << EOL

\set r random(1, 5000000)
SELECT id, fkRide, fio, contact, fkSeat FROM book.tickets WHERE id = :r;

EOL
postgres@test1:/home/popova.valeriya21$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 272942
number of failed transactions: 0 (0.000%)
latency average = 0.293 ms
initial connection time = 12.006 ms
tps = 27309.364890 (without initial connection time)
```
