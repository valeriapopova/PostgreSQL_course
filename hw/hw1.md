1. Развернуть ВМ (Linux) с PostgreSQL

2. Залить Тайские перевозки
https://github.com/aeuge/postgres16book/tree/main/database

3. Посчитать количество поездок - select count(*) from book.tickets; Сдача в формате markdown на github на почту doit@wb.ru


Ответ:

1) Запросила роль для доступа к ВМ WB Cloud
в терминале подключилась через tsh ssh
создала БД, суперюзера

2) wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql;

3)
```shell
select count(*) from book.tickets;
  count
---------
 5185505
(1 строка)
```
