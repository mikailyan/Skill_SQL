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
    ORDER BY avg_pos, car_name
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
    HAVING  COUNT(*) >= 2  
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
# Бронирвоание отелей
## Задача 1

```sql
SELECT
    c.name,
    c.email,
    c.phone,
    COUNT(*) AS total_bookings,
    GROUP_CONCAT(DISTINCT h.name ORDER BY h.name SEPARATOR ', ') AS hotels_list,
    ROUND(AVG(DATEDIFF(b.check_out_date, b.check_in_date)), 4) AS avg_stay_days
FROM   Customer  AS c
JOIN   Booking   AS b  ON b.ID_customer = c.ID_customer
JOIN   Room      AS r  ON r.ID_room     = b.ID_room
JOIN   Hotel     AS h  ON h.ID_hotel    = r.ID_hotel
GROUP  BY c.ID_customer, c.name, c.email, c.phone
HAVING COUNT(*) > 2
   AND COUNT(DISTINCT h.ID_hotel) > 1
ORDER  BY total_bookings DESC, c.name;

```
## Задача 2

```sql
SELECT
    c.ID_customer,
    c.name,
    COUNT(*)                                              AS total_bookings,
    ROUND(SUM(r.price), 2)                                AS total_spent,
    COUNT(DISTINCT h.ID_hotel)                            AS unique_hotels
FROM   Customer  AS c
JOIN   Booking   AS b  ON b.ID_customer = c.ID_customer
JOIN   Room      AS r  ON r.ID_room     = b.ID_room
JOIN   Hotel     AS h  ON h.ID_hotel    = r.ID_hotel
GROUP  BY c.ID_customer, c.name
HAVING COUNT(*)                   > 2 
   AND COUNT(DISTINCT h.ID_hotel) > 1  
   AND SUM(r.price)               > 500
ORDER  BY total_spent;

```
## Задача 3

```sql
WITH hotel_cat AS (
    SELECT
        h.ID_hotel,
        CASE
            WHEN AVG(r.price) < 175          THEN 'Дешевый'
            WHEN AVG(r.price) <= 300         THEN 'Средний'
            ELSE                                   'Дорогой'
        END AS hotel_type
    FROM   Hotel h
    JOIN   Room  r  ON r.ID_hotel = h.ID_hotel
    GROUP  BY h.ID_hotel
),
client_visits AS (
    SELECT
        c.ID_customer,
        c.name,
        GROUP_CONCAT(DISTINCT h.name ORDER BY h.name SEPARATOR ',') AS visited_hotels,
        MAX(hc.hotel_type = 'Дорогой')  AS has_expensive,
        MAX(hc.hotel_type = 'Средний')  AS has_medium,
        MAX(hc.hotel_type = 'Дешевый')  AS has_cheap
    FROM   Customer   c
    JOIN   Booking    b   ON b.ID_customer = c.ID_customer
    JOIN   Room       r   ON r.ID_room     = b.ID_room
    JOIN   Hotel      h   ON h.ID_hotel    = r.ID_hotel
    JOIN   hotel_cat  hc  ON hc.ID_hotel   = h.ID_hotel
    GROUP  BY c.ID_customer, c.name
)
SELECT
    ID_customer,
    name,
    CASE
        WHEN has_expensive = 1 THEN 'Дорогой'
        WHEN has_medium    = 1 THEN 'Средний'
        ELSE                      'Дешевый'
    END AS preferred_hotel_type,
    visited_hotels
FROM   client_visits
ORDER BY FIELD(
           CASE
               WHEN has_expensive = 1 THEN 'Дорогой'
               WHEN has_medium    = 1 THEN 'Средний'
               ELSE                      'Дешевый'
           END,
           'Дешевый', 'Средний', 'Дорогой'
         ),
         ID_customer;

```

#Структура организации

## Задача 1

```sql
WITH RECURSIVE subordinates AS (
    SELECT  EmployeeID
    FROM    Employees
    WHERE   EmployeeID = 1 
    UNION ALL
    SELECT  e.EmployeeID
    FROM    Employees e
    JOIN    subordinates s  ON e.ManagerID = s.EmployeeID
)
SELECT
    e.EmployeeID,
    e.Name                        AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    ( SELECT GROUP_CONCAT(DISTINCT p.ProjectName
                          ORDER BY p.ProjectName SEPARATOR ', ')
      FROM   Projects p
      WHERE  p.DepartmentID = e.DepartmentID )  AS ProjectNames,
    ( SELECT GROUP_CONCAT(t.TaskName
                          ORDER BY t.TaskID SEPARATOR ', ')
      FROM   Tasks t
      WHERE  t.AssignedTo = e.EmployeeID )      AS TaskNames
FROM        Employees   e
JOIN        subordinates s  ON s.EmployeeID = e.EmployeeID
LEFT JOIN   Departments  d  ON d.DepartmentID = e.DepartmentID
LEFT JOIN   Roles        r  ON r.RoleID       = e.RoleID
ORDER BY    e.Name;

```

## Задача 2

```sql
WITH RECURSIVE subordinates AS (
    SELECT  EmployeeID
    FROM    Employees
    WHERE   EmployeeID = 1
    UNION ALL
    SELECT  e.EmployeeID
    FROM    Employees e
    JOIN    subordinates s  ON e.ManagerID = s.EmployeeID
)
SELECT
    e.EmployeeID,
    e.Name                          AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    ( SELECT GROUP_CONCAT(DISTINCT p.ProjectName
                          ORDER BY p.ProjectName SEPARATOR ', ')
      FROM   Projects p
      WHERE  p.DepartmentID = e.DepartmentID )  AS ProjectNames,
    ( SELECT GROUP_CONCAT(t.TaskName
                          ORDER BY t.TaskID SEPARATOR ', ')
      FROM   Tasks t
      WHERE  t.AssignedTo = e.EmployeeID )      AS TaskNames,
    ( SELECT COUNT(*)
      FROM   Tasks t
      WHERE  t.AssignedTo = e.EmployeeID )      AS TotalTasks,
    ( SELECT COUNT(*)
      FROM   Employees x
      WHERE  x.ManagerID = e.EmployeeID )       AS TotalSubordinates
FROM        Employees   e
JOIN        subordinates s  ON s.EmployeeID = e.EmployeeID
LEFT JOIN   Departments  d  ON d.DepartmentID = e.DepartmentID
LEFT JOIN   Roles        r  ON r.RoleID       = e.RoleID
ORDER BY    e.Name;

```

## Задача 3

```sql
WITH RECURSIVE hierarchy AS (
    SELECT  ManagerID AS boss,
            EmployeeID AS sub
    FROM    Employees
    WHERE   ManagerID IS NOT NULL
    UNION ALL
    SELECT  h.boss,
            e.EmployeeID
    FROM    hierarchy h
    JOIN    Employees  e  ON e.ManagerID = h.sub
),
sub_counts AS ( 
    SELECT  boss            AS EmployeeID,
            COUNT(DISTINCT sub) AS TotalSubordinates
    FROM    hierarchy
    GROUP BY boss
)
SELECT
    e.EmployeeID,
    e.Name                        AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    ( SELECT GROUP_CONCAT(DISTINCT p.ProjectName
                          ORDER BY p.ProjectName SEPARATOR ', ')
      FROM   Projects p
      WHERE  p.DepartmentID = e.DepartmentID )  AS ProjectNames,
    ( SELECT GROUP_CONCAT(t.TaskName
                          ORDER BY t.TaskID SEPARATOR ', ')
      FROM   Tasks t
      WHERE  t.AssignedTo = e.EmployeeID )      AS TaskNames,
    sc.TotalSubordinates
FROM        Employees   e
JOIN        Roles       r   ON r.RoleID       = e.RoleID
                             AND r.RoleName   = 'Менеджер'
JOIN        sub_counts  sc  ON sc.EmployeeID  = e.EmployeeID 
LEFT JOIN   Departments d   ON d.DepartmentID = e.DepartmentID
ORDER BY    e.Name;

```
