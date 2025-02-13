### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение

Текст запроса:
```
SELECT SUM(data_length) AS "Размер таблиц", SUM(index_length) AS "Размер индексов", (SUM(index_length)/SUM(data_length))*100 AS "Процентное отношение"
FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = 'sakila';
```

Скриншот из DBeaver:
![alt text](https://github.com/masterchoo495/Indexes/blob/main/001.png)

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение

Исходный запрос перегружен условиями и затрагиваемыми таблицами, ввиду чего его выполнение на моем стенде занимает около 3-х секунд и в процессе обрабатывается (!) 642 000 строк. На мой взгляд, учитывая характер получаемых данных, избыточной является информация о названии фильма (f.title), инвентаре (inventory_id) и дате аренды (rental_date).

Вывод EXPLAIN ANALYZE:
```
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=3002..3002 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=3002..3002 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1342..2909 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=1342..1375 rows=642000 loops=1)
                -> Stream results  (cost=22.6e+6 rows=16.5e+6) (actual time=0.352..945 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=22.6e+6 rows=16.5e+6) (actual time=0.335..812 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=20.9e+6 rows=16.5e+6) (actual time=0.331..697 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=19.3e+6 rows=16.5e+6) (actual time=0.326..583 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.314..23.7 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.72 rows=16500) (actual time=0.12..3.3 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.72 rows=16500) (actual time=0.113..2.26 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0266..0.131 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1) (actual time=553e-6..791e-6 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=69.1e-6..83.9e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=73.2e-6..89.7e-6 rows=1 loops=642000)
```

Текст оптимизированного запроса (убрал лишнее из описанного выше):
```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) over (partition by c.customer_id)
FROM payment p, customer c
WHERE date(p.payment_date) = '2005-07-30' and c.customer_id  = p.customer_id
```
Данный запрос возвращает тот же результат (391 строку), но выполняется за 0,007 секунды и обрабатывает всего лишь 634 строки, т.е. в 1000 раз меньше, чем при выполнении исходного запроса.

Вывод EXPLAIN ANALYZE:
```sql
-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4.23..4.26 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4.23..4.23 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=3.69..4.15 rows=634 loops=1)
            -> Sort: c.customer_id  (actual time=3.67..3.69 rows=634 loops=1)
                -> Stream results  (cost=7449 rows=16500) (actual time=0.167..3.6 rows=634 loops=1)
                    -> Nested loop inner join  (cost=7449 rows=16500) (actual time=0.162..3.48 rows=634 loops=1)
                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1674 rows=16500) (actual time=0.15..3.07 rows=634 loops=1)
                            -> Table scan on p  (cost=1674 rows=16500) (actual time=0.143..2.39 rows=16044 loops=1)
                        -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=528e-6..543e-6 rows=1 loops=634)
```

Скриншот из DBeaver:
![alt text](https://github.com/masterchoo495/Indexes/blob/main/002.png)

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
