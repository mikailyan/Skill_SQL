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

