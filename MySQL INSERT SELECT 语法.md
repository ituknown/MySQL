```sql
INSERT INTO dest_table (column1, column2, column3)
SELECT column1, column2, column3
FROM source_table
WHERE some_condition;
```