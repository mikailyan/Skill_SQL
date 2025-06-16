# Skill_SQL

'''
SELECT  v.maker,
        m.model
FROM    Motorcycle AS m
JOIN    Vehicle    AS v  ON v.model = m.model
WHERE   m.horsepower  > 150
  AND   m.price       < 20000
  AND   m.type        = 'Sport'
ORDER BY m.horsepower DESC;
'''
