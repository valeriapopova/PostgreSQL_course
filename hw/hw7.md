> 1. Развернуть ВМ (Linux) с PostgreSQL
> 2. Залить Тайские перевозки
https://github.com/aeuge/postgres16book/tree/main/database
> 3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)
```shell
\connect thai
```
включаю таймер

```shell
\timing

 id | depart_date |       busstation       | order_place | all_place
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 строк)
Время: 1614,116 мс (00:01,614)
```
> 4. Навесить индексы на внешние ключ
```shell
thai=# create index idx_schedule on book.ride(fkschedule);
CREATE INDEX
Время: 111,463 мс
thai=# create index idx_route on book.schedule(fkroute);
CREATE INDEX
Время: 35,467 мс
thai=# create index idx_busstation on book.busroute(fkbusstationfrom);
CREATE INDEX
Время: 2,829 мс
thai=# create index idx_ride on book.tickets(fkride);
CREATE INDEX
Время: 7367,364 мс (00:07,367)
thai=# create index idx_bus on book.seat(fkbus);
CREATE INDEX
```
> 5. Проверить, помогли ли индексы на внешние ключи ускориться
сначала ничего не поменялось, потом я попробовала использовать составной индекс в book.ride и это немного улучшило ситуацию
(так как объединение происходит по нескольким полям)
```shell
thai=# create index idx_schedule_bus on book.ride(fkschedule, fkbus);
CREATE INDEX
thai=# WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
 id | depart_date |       busstation       | order_place | all_place
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 строк)

Время: 1562,299 мс (00:01,562)
```