> 1.Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
Я хотела немного поучиться составлять процедуры, так что попробовала написать так:

```shell
postgres=# CREATE OR REPLACE PROCEDURE insert_random_text(cnt int)
LANGUAGE plpgsql
AS $$
DECLARE
    a INT;
BEGIN
    FOR a IN 1..cnt LOOP
        INSERT INTO test.hw3(random)
        VALUES (md5(random()::text));
    END LOOP;
END;
$$;
CREATE PROCEDURE
postgres=# CALL insert_random_text(1000000);
CALL
```
Но намного эффективнее было бы использовать запрос, так как он выполняет только одну транзакцию вместо миллиона как в процедуре:
```shell
postgres=# insert into test.hw3(random) select md5(random()::text) from generate_series(1, 1000000);
INSERT 0 1000000
```
> 2. Посмотреть размер файла с таблицей

```shell
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test.hw3'));
 pg_size_pretty
----------------
 87 MB
(1 строка)
```
> 3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```shell
postgres=# CREATE OR REPLACE PROCEDURE insert_q()
LANGUAGE plpgsql
AS $$
DECLARE
    a INT;
    q  varchar := 'q';
BEGIN
    FOR a IN 1..5 LOOP
        UPDATE test.hw3
        SET random = random ||  q;
    END LOOP;
END;
$$;
CREATE PROCEDURE
postgres=# CALL insert_q();
CALL
```
> 4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
автовакуум

```shell
postgres=# SELECT relname, n_dead_tup, last_autovacuum FROM pg_stat_all_tables WHERE relid='test.hw3'::regclass;
 relname | n_dead_tup |        last_autovacuum
---------+------------+-------------------------------
 hw3     |    5000000 | 2024-10-16 21:19:15.264126+03
(1 строка)
```
> 5. Подождать некоторое время, проверяя, пришел ли автовакуум
Автовакуум пришел и стало 0 мертвых строк
```shell
postgres=# SELECT relname, n_dead_tup, last_autovacuum FROM pg_stat_all_tables WHERE relid='test.hw3'::regclass;
 relname | n_dead_tup |        last_autovacuum
---------+------------+-------------------------------
 hw3     |          0 | 2024-10-16 21:27:03.501037+03
(1 строка)
```
> 6. 5 раз обновить все строчки и добавить к каждой строчке любой символ

```shell
postgres=# CALL insert_q();
CALL
```
> 7. Посмотреть размер файла с таблицей

```shell
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test.hw3'));
 pg_size_pretty
----------------
 554 MB
(1 строка)
```
> 8. Отключить Автовакуум на конкретной таблице
postgres=# ALTER TABLE test.hw3 SET (autovacuum_enabled = false);
ALTER TABLE
> 9. 10 раз обновить все строчки и добавить к каждой строчке любой символ

тут я поняла, что стоило прочитать дз полностью сразу, а в процедуру добавить аргумент cnt (кол-во итераций):

```shell
postgres=# CREATE OR REPLACE PROCEDURE insert_q(cnt int)
LANGUAGE plpgsql
AS $$
DECLARE
    a INT;
    q  varchar := 'q';
BEGIN
    FOR a IN 1..cnt LOOP
        UPDATE test.hw3
        SET random = random ||  q;
    END LOOP;
END;
$$;
CREATE PROCEDURE
postgres=# CALL insert_q(10);
CALL
```
> 10. Посмотреть размер файла с таблицей

```shell
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test.hw3'));
 pg_size_pretty
----------------
 1452 MB
(1 строка)
```
> 11. Объясните полученный результат

Размер файла увеличился, так как автовакуум отключен мертвые строки
увеличили размер файла и не были удалены (как произошло бы с включенным автовакуумом),
поэтому они продолжают занимать место на диске

> 12. Не забудьте включить автовакуум)

```shell
postgres=# ALTER TABLE test.hw3 SET (autovacuum_enabled = true);
ALTER TABLE
```
Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.

```shell
DO $$
DECLARE
    a INT;
BEGIN
    FOR a IN 1..10 LOOP
        UPDATE test.hw3
        SET random = random || 'q';
        RAISE NOTICE 'номер шага цикла %', a;
    END LOOP;
END $$;
```