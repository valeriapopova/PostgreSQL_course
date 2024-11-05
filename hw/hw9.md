> 1. Сгенерировать таблицу с 1 млн JSONB документов
```shell
postgres=# INSERT INTO hw9.docs (docs_info)
SELECT jsonb_build_object(
    'phone', '+7' || '9'||  (RANDOM() * 1e8)::bigint::text,
    'email', md5(RANDOM()::text) || '@mail.ru',
    'fullname', 'blabla' || a
)
FROM generate_series(1, 1000000) AS a;
INSERT 0 1000000
postgres=# select * from hw9.docs limit 10;

   id    |                                               docs_info
---------+-------------------------------------------------------------------------------------------------------
 3000001 | {"email": "8271a83437951371c52a226c2d53c5c3@mail.ru", "phone": "+7948526703", "fullname": "blabla1"}
 3000002 | {"email": "bc70c08e7e7cc0326ca62b7ba61c802f@mail.ru", "phone": "+7983413552", "fullname": "blabla2"}
 3000003 | {"email": "d4f0f859e4d5eb2f81f59eaf00e74666@mail.ru", "phone": "+7924288115", "fullname": "blabla3"}
 3000004 | {"email": "cdbe76f62392a3ba461defaa11546482@mail.ru", "phone": "+7952624119", "fullname": "blabla4"}
 3000005 | {"email": "3abdf89e0f63cb71a2363a1c81635597@mail.ru", "phone": "+799868461", "fullname": "blabla5"}
 3000006 | {"email": "2fad8381a24fd0240562a4b3b53ada42@mail.ru", "phone": "+7990526952", "fullname": "blabla6"}
 3000007 | {"email": "8551dcdbf3650f32017c55471fb23482@mail.ru", "phone": "+7998128509", "fullname": "blabla7"}
 3000008 | {"email": "7efeb759cfd1afb9569d1b5fb195c59b@mail.ru", "phone": "+7956587977", "fullname": "blabla8"}
 3000009 | {"email": "c20d4e22e325f73a382e3d7a55535315@mail.ru", "phone": "+7984315454", "fullname": "blabla9"}
```

> 2. Создать индекс
```shell
postgres=# CREATE INDEX idx_docs_name ON hw9.docs USING GIN ((docs_info->'name'));
CREATE INDEX
```
> 3. Обновить 1 из полей в json
долго не получалось обновить поля так, чтобы появился блоатинге, поэтому добавила длинное текстовое поле в jsonb
```shell
postgres=# CREATE OR REPLACE FUNCTION generate_string(length INT)
RETURNS TEXT AS $$
DECLARE
    chars TEXT := 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    result TEXT := '';
BEGIN
    FOR i IN 1..length LOOP
        result := result || substr(chars, (random() * length(chars))::int + 1, 1);
    END LOOP;
    RETURN result;
END;
$$ LANGUAGE plpgsql;
CREATE FUNCTION
postgres=# UPDATE hw9.docs
SET docs_info = jsonb_set(docs_info, '{a}', to_jsonb(generate_string(1024 * 10)));
```
> 4. Убедиться в блоатинге TOAST
```shell
SELECT reltoastrelid::regclass AS toast_table_name
FROM pg_class
WHERE oid = 'hw9.docs'::regclass;
    toast_table_name
-------------------------
 pg_toast.pg_toast_16600
(1 строка)
```
```shell
postgres=# SELECT * FROM pgstattuple('pg_toast.pg_toast_16600');



 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | dead_tuple_percent | free_space | free_percent
-----------+-------------+-----------+---------------+------------------+----------------+--------------------+------------+--------------
 985661440 |           0 |         0 |             0 |           481280 |      870376909 |               88.3 |  109589432 |        11.12
(1 строка)
```
> 5. Придумать методы избавится от него и проверить на практике
```shell
postgres=# VACUUM hw9.docs;
VACUUM
postgres=# REINDEX TABLE hw9.docs;
REINDEX
postgres=# SELECT * FROM pgstattuple('pg_toast.pg_toast_16600');

 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | dead_tuple_percent | free_space | free_percent
-----------+-------------+-----------+---------------+------------------+----------------+--------------------+------------+--------------
         0 |           0 |         0 |             0 |                0 |              0 |                  0 |          0 |            0
(1 строка)
```

> 6. Не забываем про блоатинг индексов*
```shell
REINDEX INDEX idx_docs_name;
```