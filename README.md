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

Текст запроса:
```
SELECT SUM(data_length) AS "Размер таблиц", SUM(index_length) AS "Размер индексов", (SUM(index_length)/SUM(data_length))*100 AS "Процентное отношение"
FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = 'sakila';
```

Скриншот из DBeaver:
![alt text](https://github.com/masterchoo495/Indexes/blob/main/002.png)

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
