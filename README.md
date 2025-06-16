# Skill_SQL

# Транспортные средства
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
```
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
```

# Автомобильные гонки

## Задача 1

```sql
WITH car_stats AS (
    SELECT  c.name          AS car_name,
            c.class         AS car_class,
            AVG(r.position) AS avg_pos,
            COUNT(*)        AS race_count
    FROM    Cars     AS c
    JOIN    Results  AS r  ON r.car = c.name
    GROUP BY c.name, c.class
),
ranked AS (
    SELECT  cs.*,
            ROW_NUMBER() OVER (PARTITION BY cs.car_class
                               ORDER BY cs.avg_pos, cs.car_name) AS rn
    FROM    car_stats cs
)
SELECT  car_name,
        car_class,
        ROUND(avg_pos, 4)  AS average_position,
        race_count
FROM    ranked
WHERE   rn = 1
ORDER BY average_position;

```
## Задача 2

```sql
WITH car_stats AS (
    SELECT  c.name          AS car_name,
            c.class         AS car_class,
            AVG(r.position) AS avg_pos,
            COUNT(*)        AS race_count
    FROM    Cars     AS c
    JOIN    Results  AS r  ON r.car = c.name
    GROUP BY c.name, c.class
),
best_overall AS (
    SELECT  *
    FROM    car_stats
    ORDER BY avg_pos, car_name      -- если ничья, берём по алфавиту
    LIMIT 1
)
SELECT  bo.car_name,
        bo.car_class,
        ROUND(bo.avg_pos, 4) AS average_position,
        bo.race_count,
        cl.country           AS car_country
FROM    best_overall bo
JOIN    Classes      cl ON cl.class = bo.car_class;

```
## Задача 3

```sql
WITH car_stats AS (
    SELECT  c.name          AS car_name,
            c.class         AS car_class,
            AVG(r.position) AS avg_pos,
            COUNT(*)        AS race_count
    FROM    Cars     AS c
    JOIN    Results  AS r  ON r.car = c.name
    GROUP BY c.name, c.class
),
class_avg AS (
    SELECT  car_class,
            AVG(avg_pos)      AS class_avg_pos,
            SUM(race_count)   AS total_races
    FROM    car_stats
    GROUP BY car_class
),
min_val AS (
    SELECT  MIN(class_avg_pos) AS best_avg
    FROM    class_avg
),
best_classes AS (
    SELECT  ca.*
    FROM    class_avg ca
    JOIN    min_val  mv ON mv.best_avg = ca.class_avg_pos
)
SELECT  cs.car_name,
        cs.car_class,
        ROUND(cs.avg_pos, 4) AS average_position,
        cs.race_count,
        cl.country           AS car_country,
        bc.total_races
FROM    car_stats  cs
JOIN    best_classes bc ON bc.car_class = cs.car_class
JOIN    Classes     cl ON cl.class     = cs.car_class
ORDER BY cs.car_class, cs.car_name;

```
## Задача 4

```sql
WITH car_stats AS (
    SELECT  c.name          AS car_name,
            c.class         AS car_class,
            AVG(r.position) AS avg_pos,
            COUNT(*)        AS race_count
    FROM    Cars     AS c
    JOIN    Results  AS r  ON r.car = c.name
    GROUP BY c.name, c.class
),
class_metrics AS (
    SELECT  car_class,
            AVG(avg_pos) AS class_avg,
            COUNT(*)     AS car_cnt
    FROM    car_stats
    GROUP BY car_class
    HAVING  COUNT(*) >= 2          -- учитываем только классы с ≥2 авто
)
SELECT  cs.car_name,
        cs.car_class,
        ROUND(cs.avg_pos, 4) AS average_position,
        cs.race_count,
        cl.country           AS car_country
FROM    car_stats cs
JOIN    class_metrics cm ON cm.car_class = cs.car_class
JOIN    Classes       cl ON cl.class     = cs.car_class
WHERE   cs.avg_pos < cm.class_avg
ORDER BY cs.car_class, cs.avg_pos;

```
## Задача 5

```sql
WITH car_stats AS (
    SELECT  c.name          AS car_name,
            c.class         AS car_class,
            AVG(r.position) AS avg_pos,
            COUNT(*)        AS race_count
    FROM    Cars     AS c
    JOIN    Results  AS r  ON r.car = c.name
    GROUP BY c.name, c.class
),
low_pos_cars AS (
    -- Автомобили с низкой (т.е. высокой численно) средней позицией
    SELECT * 
    FROM   car_stats
    WHERE  avg_pos > 3.0
),
low_counts AS (
    SELECT  car_class,
            COUNT(*) AS low_position_count
    FROM    low_pos_cars
    GROUP BY car_class
),
total_races AS (
    SELECT  car_class,
            SUM(race_count) AS total_races
    FROM    car_stats
    GROUP BY car_class
)
SELECT  lpc.car_name,
        lpc.car_class,
        ROUND(lpc.avg_pos, 4)  AS average_position,
        lpc.race_count,
        cl.country             AS car_country,
        tr.total_races,
        lc.low_position_count
FROM        low_pos_cars lpc
JOIN        low_counts   lc ON lc.car_class = lpc.car_class
JOIN        total_races  tr ON tr.car_class = lpc.car_class
JOIN        Classes      cl ON cl.class     = lpc.car_class
ORDER BY    lc.low_position_count DESC,
            lpc.car_class,
            lpc.car_name;

```
