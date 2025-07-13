SQL Join

```sql
SELECT * 
FROM machines
LEFT JOIN employees ON machines.device_id = employees.device_id;
```

```sql
SELECT * 
FROM machines
RIGHT JOIN employees ON machines.device_id = employees.device_id;
```

```sql
SELECT * 
FROM employees
INNER JOIN username ON employees.username = log_in_attempts.username;
(WRONG)
```

Example:
```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name = table2.column_name;
```

```sql
SELECT *
FROM employees
INNER JOIN log_in_attempts ON employees.username = log_in_attempts.username;
(CORRECT)
```
