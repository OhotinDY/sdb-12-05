# Домашнее задание к занятию «Индексы»

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```sql
SELECT SUM(index_length) / SUM(data_length) * 100 AS percent_ratio
FROM information_schema.tables
WHERE table_schema = 'sakila';
```
![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index1.jpg)


### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

---

- Выполнение EXPLAIN ANALYZE запроса из задания:
![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index2.jpg)

- Запрос:
![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index3.jpg)

- Оптимизированный запрос:

```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) OVER(PARTITION BY c.customer_id)
FROM payment p, customer c
WHERE DATE(p.payment_date) = '2005-07-30' AND p.customer_id = c.customer_id;
```

![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index4.jpg)

![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index5.jpg)

- Проверка существующих индексов:

![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index6.jpg)

- Создание индекса idx_payment_date для поля payment_date таблицы payment:

```sql
CREATE INDEX idx_payment_date ON payment(payment_date);
```

![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index7.jpg)



![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index8.jpg)

- Скорректированный запрос, его выполнение, проверка использования индекса помощью EXPLAIN и выполнение EXPLAIN ANALYZE:

```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) OVER(PARTITION BY c.customer_id)
FROM payment p USE INDEX (idx_payment_date), customer c 
WHERE payment_date >= '2005-07-30' and payment_date < '2005-07-31' AND p.customer_id = c.customer_id;
```

![sql3](https://github.com/OhotinDY/sdb-12-05/blob/main/index.jpg)
