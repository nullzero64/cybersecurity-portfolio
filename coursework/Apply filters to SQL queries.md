# Apply filters to SQL queries

## Project Description

This project demonstrates how to filter SQL database tables to retrieve specific data relevant to analysis and security. You’ll learn how to use AND, OR, and NOT operators to refine your queries, understand the syntax for each, and see the results of applying these powerful filters. Mastering these filtering techniques will help you efficiently extract and analyze information from large datasets.

## Retrieve after hours failed login attempts

Filter login attempts to show only those that occurred after 6pm and were not successful.

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00' AND success = 0;
```

<img width="975" height="953" alt="image" src="https://github.com/user-attachments/assets/18d2883a-4a19-4449-873d-a1ec6e7bc758" />

## Retrieve login attempts on specific dates

Display login attempts that happened on either May 8 or May 9, regardless of the outcome.

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-08' OR login_date = '2022-05-09';
```

<img width="975" height="1159" alt="image" src="https://github.com/user-attachments/assets/f8b17664-0249-4463-bb0f-93e6690403c2" />

## Retrieve login attempts outside of Mexico

Show all login attempts where the country is not Mexico.

```sql
SELECT *
FROM log_in_attempts
WHERE country != 'MEXICO';
```

<img width="975" height="1281" alt="image" src="https://github.com/user-attachments/assets/658ecca1-ec77-4933-b348-998a9f8d1f16" />

## Retrieve employees in Marketing

```sql
SELECT *
FROM employees
WHERE department = 'Marketing';
```

<img width="856" height="386" alt="image" src="https://github.com/user-attachments/assets/bfe8d59d-cf45-4128-9d12-b30d6876aebf" />

## Retrieve employees in Finance or Sales

```sql
SELECT *
FROM employees
WHERE department = 'Finance' OR department = 'Sales';
```

<img width="880" height="1261" alt="image" src="https://github.com/user-attachments/assets/d6a7f771-5ed9-4898-b71c-f3364f5540c9" />

## Retrieve all employees not in IT

```sql
SELECT *
FROM employees
WHERE department != 'IT';
```

<img width="945" height="861" alt="image" src="https://github.com/user-attachments/assets/ffaba147-98db-409c-b156-d0ec966e7bc1" />

## Summary

Efficiently filtering data is crucial for security analysis and decision-making. By using SQL operators like AND, OR, and NOT, you can quickly isolate relevant events—such as failed logins after work hours, activity on specific dates, or actions outside a particular region. These skills empower you to extract actionable insights from your databases and better protect your organization, making your data handling both targeted and effective.
