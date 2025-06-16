# Skill_SQL

## Задача 1

Найдите производителей и модели спортивных мотоциклов с мощностью более 150 л.с., ценой до $20,000.

```sql
SELECT  v.maker,
        m.model
FROM    Motorcycle AS m
JOIN    Vehicle    AS v  ON v.model = m.model
WHERE   m.horsepower  > 150
  AND   m.price       < 20000
  AND   m.type        = 'Sport'
ORDER BY m.horsepower DESC;

## Задача 2

Вывести информацию о подходящих автомобилях, мотоциклах и велосипедах, отсортированных по мощности.

```sql
SELECT  v.maker,
        c.model,
        c.horsepower,
        c.engine_capacity,
        'Car'        AS vehicle_type
FROM    Car      AS c
JOIN    Vehicle  AS v  ON v.model = c.model
WHERE   c.horsepower      > 150
  AND   c.engine_capacity < 3
  AND   c.price           < 35000

UNION ALL

SELECT  v.maker,
        m.model,
        m.horsepower,
        m.engine_capacity,
        'Motorcycle' AS vehicle_type
FROM    Motorcycle AS m
JOIN    Vehicle    AS v  ON v.model = m.model
WHERE   m.horsepower      > 150
  AND   m.engine_capacity < 1.5
  AND   m.price           < 20000

UNION ALL

SELECT  v.maker,
        b.model,
        NULL          AS horsepower,
        NULL          AS engine_capacity,
        'Bicycle'     AS vehicle_type
FROM    Bicycle  AS b
JOIN    Vehicle  AS v  ON v.model = b.model
WHERE   b.gear_count  > 18
  AND   b.price       < 4000

ORDER BY horsepower DESC;

