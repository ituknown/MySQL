**单表：使用子查询**

```sql
UPDATE table t1
INNER JOIN (
  SELECT column...
  FROM table
  WHERE some_condition
) AS t2 ON t1.column = t2.column
SET t1.column = t2.column...
```

**多表：使用 JOIN 查询**

```sql
UPDATE table1 t1
INNER JOIN table2 t2 ON t1.column = t2.column
SET t1.column = t2.column...
WHERE some_condition
```
