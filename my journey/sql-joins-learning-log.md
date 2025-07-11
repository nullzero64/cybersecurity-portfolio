# SQL Joins – My Learning Log

## ❌ What I Did Wrong
```sql
SELECT * 
FROM employees
INNER JOIN username ON employees.username = log_in_attempts.username;
```
I mistakenly tried to JOIN using a column name `username` as if it were a table.

## ✅ How I Fixed It
```sql
SELECT * 
FROM employees
INNER JOIN log_in_attempts ON employees.username = log_in_attempts.username;
```

## 🧠 What I Learned
- Always JOIN on a **table**, not on a column name.
- `INNER JOIN table2 ON table1.column = table2.column` is the correct structure.
- Writing out wrong vs. right helps me retain it better.

## 📅 Date
2025-07-11

## 🗒️ Notes
This was part of my early SQL practice. I was exploring JOINs using TryHackMe and practicing writing out commands in Notepad. It's okay to get things wrong — I’m tracking these to show how I grow from each mistake.
