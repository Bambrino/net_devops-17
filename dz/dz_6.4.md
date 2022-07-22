### Задача №1

- \l - список баз
- \с [database_name] - подключение к указанной базе
- \dt - вывод списка таблиц
- \d [table_name] - описание таблицы

    "+" - даст расширенное описание, например, \d+ [table_name] или \dt+
- \q - выход

### Задача №2
```sql
postgres=# create database test_database;
CREATE DATABASE
postgres=# \c test_database 
You are now connected to database "test_database" as user "postgres".
test_database=# \i /media/postgresql/backup/test_dump.sql 
...

test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)

```
```sql
test_database=# select attname, avg_width from pg_stats where tablename = 'orders' order by avg_width desc limit 1;
 attname | avg_width 
---------+-----------
 title   |        16
(1 row)
```

### Задача №3

<details>
  
```sql
test_database=# create table orders_1 (check (price > 499)) inherits (orders);
CREATE TABLE
test_database=# create table orders_2 (check (price <= 499)) inherits (orders);
CREATE TABLE

test_database=# insert into orders_1 select * from orders where price > 499;
INSERT 0 3
test_database=# insert into orders_2 select * from orders where price <= 499;
INSERT 0 5
test_database=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | orders   | table | postgres
 public | orders_1 | table | postgres
 public | orders_2 | table | postgres


test_database=# create rule "orders with price > 499" as on insert to orders where (price > 499) do instead insert into orders_1 values (new.*);
CREATE RULE

test_database=# create rule "orders with price <= 499" as on insert to orders where (price <= 499) do instead insert into orders_2 values (new.*);
CREATE RULE
test_database=# \d+ orders
                                                       Table "public.orders"
 Column |         Type          | Collation | Nullable |              Default               | Storage  | Stats target | Description 
--------+-----------------------+-----------+----------+------------------------------------+----------+--------------+-------------
 id     | integer               |           | not null | nextval('orders_id_seq'::regclass) | plain    |              | 
 title  | character varying(80) |           | not null |                                    | extended |              | 
 price  | integer               |           |          | 0                                  | plain    |              | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Rules:
    "orders with price <= 499" AS
    ON INSERT TO orders
   WHERE new.price <= 499 DO INSTEAD  INSERT INTO orders_2 (id, title, price)
  VALUES (new.id, new.title, new.price)
    "orders with price > 499" AS
    ON INSERT TO orders
   WHERE new.price > 499 DO INSTEAD  INSERT INTO orders_1 (id, title, price)
  VALUES (new.id, new.title, new.price)
Child tables: orders_1,
              orders_2
Access method: heap

test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
  2 | My little database   |   500
  6 | WAL never lies       |   900
  8 | Dbiezdmin            |   501
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(16 rows)


test_database=# create table orders_temp(id int, title varchar(80), price int);
CREATE TABLE

test_database=# select distinct * from orders order by id;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
(8 rows)

test_database=# insert into orders_temp select distinct * from orders order by id;
INSERT 0 8

test_database=# delete from orders;
DELETE 16

test_database=# insert into orders select * from orders_temp;
INSERT 0 0
 
test_database=# drop table orders_temp ;
DROP TABLE

test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  2 | My little database   |   500
  6 | WAL never lies       |   900
  8 | Dbiezdmin            |   501
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(8 rows)

test_database=# select * from orders_1;
 id |       title        | price 
----+--------------------+-------
  2 | My little database |   500
  6 | WAL never lies     |   900
  8 | Dbiezdmin          |   501
(3 rows)

test_database=# select * from orders_2;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(5 rows)

test_database=# insert into orders values (9,'Some strange thing', 999);
INSERT 0 0
test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  2 | My little database   |   500
  6 | WAL never lies       |   900
  8 | Dbiezdmin            |   501
  9 | Some strange thing   |   999
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(9 rows)

test_database=# select * from orders_1;
 id |       title        | price 
----+--------------------+-------
  2 | My little database |   500
  6 | WAL never lies     |   900
  8 | Dbiezdmin          |   501
  9 | Some strange thing |   999
(4 rows)

test_database=# select * from orders_2;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  7 | Me and my bash-pet   |   499
(5 rows)

```
  
</details>
  
 Для исключние ручного шардирования - нужно учесть такое на стадии планирования, ну и, собственно, добавить правила до прода
  
  ### Зачада №4
  
  ```commandline
  root@fb2d8bd923bf:/# pg_dump -U postgres -d test_database > /media/postgresql/backup/test_database.sql 
  ```
  Чтобы добавить требование уникальности для занчения в столбце title, нужно добавить UNIQUE, например:
  ```sql
  CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL UNIQUE,
    price integer DEFAULT 0
);

  ```
  
