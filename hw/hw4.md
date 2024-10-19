>1. Создать таблицу accounts(id integer, amount numeric);

```shell
postgres=# create table test.accounts(id integer, amount numeric);
CREATE TABLE
```
>2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).

```shell
postgres=# INSERT INTO test.accounts VALUES (1,800.00), (2,1000.00), (3,500.00);
INSERT 0 3
CREATE TABLE
```
в первом терминале:
```shell
begin;
BEGIN
postgres=*# update test.accounts set amount = 2000.00 where id=1;
UPDATE 1
```
во втором терминале:
```shell
begin;
BEGIN
postgres=*# update test.accounts set amount=2000.00 where id=2;
```
в первом терминале:
```shell
postgres=*# update test.accounts set amount = 500.00 where id=2;
UPDATE 1
```
во втором терминале:
```shell
postgres=*# update test.accounts set amount=90.00 where id=1;
ERROR:  deadlock detected
ПОДРОБНОСТИ:  Process 143173 waits for ShareLock on transaction 943; blocked by process 143166.
Process 143166 waits for ShareLock on transaction 944; blocked by process 143173.
ПОДСКАЗКА:  See server log for query details.
КОНТЕКСТ:  while updating tuple (0,6) in relation "accounts"
```
>3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.
```shell
sudo tail /var/log/postgresql/postgresql-15-main.log
2024-10-19 17:35:31.769 MSK [143173] postgres@postgres ERROR:  deadlock detected
2024-10-19 17:35:31.769 MSK [143173] postgres@postgres DETAIL:  Process 143173 waits for ShareLock on transaction 943; blocked by process 143166.
	Process 143166 waits for ShareLock on transaction 944; blocked by process 143173.
	Process 143173: update test.accounts set amount=90.00 where id=1;
	Process 143166: update test.accounts set amount = 500.00 where id=2;
2024-10-19 17:35:31.769 MSK [143173] postgres@postgres HINT:  See server log for query details.
2024-10-19 17:35:31.769 MSK [143173] postgres@postgres CONTEXT:  while updating tuple (0,6) in relation "accounts"
2024-10-19 17:35:31.769 MSK [143173] postgres@postgres STATEMENT:  update test.accounts set amount=90.00 where id=1;
```