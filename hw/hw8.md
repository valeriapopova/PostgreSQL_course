> 1. Создать таблицу с продажами.
```shell
postgres=# CREATE TABLE hw8.sales (id int, dt date, price numeric);
CREATE TABLE
postgres=# INSERT INTO hw8.sales (id, dt, price)
SELECT generate_series(1,100) as id, date '2023-01-01' + (random()*364)::int as dt,
(random() * 1000)::numeric(10,2) as price;
INSERT 0 100
```
> 2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)
а. через case
```shell
CREATE OR REPLACE FUNCTION hw8(dt date) RETURNS text
LANGUAGE plpgsql
AS $$
DECLARE
    i text;
BEGIN
    CASE
        WHEN date_part('month', dt) <= 4 THEN
            i := 'первая треть';
        WHEN date_part('month', dt) BETWEEN 5 AND 8 THEN
            i := 'вторая треть';
        ELSE
            i := 'третья';
    END CASE;

    RETURN i;
END;
$$;
CREATE FUNCTION
```
b. * (бонуса в виде зачета дз не будет) используя математическую операцию (лучше 2+ варианта)
тут я поняла, как могла бы уменьшить кол-во повторений date_part('month', dt) прописав в переменную
```shell
CREATE OR REPLACE FUNCTION hw8(dt date) RETURNS text
LANGUAGE plpgsql
AS $$
DECLARE
    i text;
    m int := (date_part('month', dt) - 1);
BEGIN
    CASE
        WHEN m/4 = 0 THEN
            i := 'первая треть';
        WHEN m/4 = 1 THEN
            i := 'вторая треть';
        WHEN m/4  = 2 THEN
            i := 'третья';
    END CASE;
    RETURN i;
END;
$$;
с. предусмотреть NULL на входе
> 3. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало
```shell
postgres=# SELECT dt, hw8(dt) FROM hw8.sales;

     dt     |     hw8
------------+--------------
 2023-01-21 | первая треть
 2023-10-21 | третья
 2023-04-28 | первая треть
 2023-02-21 | первая треть
 2023-10-05 | третья
 2023-11-14 | третья
 2023-07-20 | вторая треть
 2023-11-25 | третья
 2023-12-08 | третья
 2023-05-23 | вторая треть
 2023-10-27 | третья
 2023-12-10 | третья
 2023-08-24 | вторая треть
 2023-02-17 | первая треть
 2023-09-24 | третья
 2023-11-24 | третья
 2023-08-19 | вторая треть
 2023-03-17 | первая треть
 2023-10-09 | третья
 2023-08-20 | вторая треть
 2023-01-03 | первая треть
 ```