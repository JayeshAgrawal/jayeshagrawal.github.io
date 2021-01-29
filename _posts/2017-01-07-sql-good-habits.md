---
title: "MySQL - Good Habits"
author: "Jayesh Agrawal"
date: 2017-01-07 20:55:00 +0530
categories: [MySQL]
tags: [Database, StoredProcedure, GoodHabits]
---

In this article, we are going to learn few good habits that we can consider important aspect while work with MySQL to improve performance & troubleshoot as following below:

## 1 - Do not use stored procedure & function parameters name same as WHERE clause field name
- It will be responding with all record of query because MySQL interprets field value as parameter value similar like 1=1.

**Example**
```sql
-- Bad  
CREATE PROCEDURE `getPersonById`(IN id INT(10))  
BEGIN  
-- return all record instead  
SELECT id,name FROM person WHERE id = id;  
END  
-- Good  
CREATE PROCEDURE getPersonById(IN personId INT(10))  
BEGIN  
SELECT id,name FROM person WHERE id = personId;  
END   
```

## 2 - Use same data-type in WHERE clause
- It will be impact on performance because MySQL hold extra memory to type conversion.

**Example**
```sql
-- Bad  
SELECT name FROM person WHERE id = '1001';  
-- Good  
SELECT name FROM person WHERE id = 1001;  
```

## 3 - Use EXISTS clause
- It will be improve response time where need logic based on existence of record in MySQL.

**Example**
```sql
-- Bad  
IF(SELECT COUNT(*) FROM person) > 0;  
-- Good  
IF EXISTS(SELECT 1 FROM person);   
```

## 4 - Add indexing to column that used to join table
- MySQL use index to faster querying data. we can use EXPLAIN SELECT statement that shows how MySQL query optimizer will execute the query.

## 5 - Avoid function over indexed column
- Function over indexed column will be defeat purpose of indexing.

**Example**
```sql
-- Bad  
SELECT name FROM person WHERE UPPER(name) LIKE 'J%';  
-- Good  
SELECT name FROM person WHERE name LIKE 'J%';   
```
## 6 - Prefer ENUM over VARCHAR data-type for multi value column(gender, status, state) for large tables
- It will be improve response time.

**Example**
```sql
-- VARCHAR  
CREATE TABLE person(  
id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,  
name VARCHAR(50) NOT NULL,  
gender VARCHAR(50)  
)ENGINE=MyISAM;  
-- ENUM  
CREATE TABLE person(  
id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,  
name VARCHAR(50) NOT NULL,  
gender ENUM('Male','Female')  
)ENGINE=MyISAM;   
```

## 7 - Avoid SELECT *
- As best practice, Always retrieve necessary columns with select statement that improves response time.

## 8 - Avoid use of GROUP BY clause without aggregate function
- It will be always retrieve first record by grouped column. so, that will be differ if we expect all record based on grouped column.

**Example**
```sql
-- Bad  
SELECT id,name FROM person GROUP BY name;  
-- Good  
SELECT name, count(*) as count FROM person GROUP BY name;
```

## Conclusion
- In this article, we have learned basic keyword/approach that can be helped us to improving performance/troubleshoot in MySQL.
