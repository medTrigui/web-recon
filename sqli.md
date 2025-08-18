# SQL Injection (SQLi) Notes

## Overview
- **OWASP Ranking**: 3rd in Top 10 Application Security Risks (A03:2021-Injection)
- **Definition**: Web application vulnerability allowing attackers to manipulate SQL queries between web app and database
- **Impact**: Extends original queries to access normally inaccessible database tables

## Learning Objectives
- SQL Theory Fundamentals
- Different Database Types & Syntax
- Manual SQL Exploitation Techniques
- SQL Attack Automation

---

## SQL Theory & Database Fundamentals

### Web Application Architecture
- **Frontend**: User-facing interface (HTML, CSS, JavaScript)
- **Backend**: Server-side application layer (PHP, Java, Python)
- **Database**: Data storage layer with SQL interface

### Popular Database Types
- MySQL
- Microsoft SQL Server
- PostgreSQL
- Oracle
- *Note*: Each has unique syntax, commands, and functions

---

## Basic SQL Syntax

### SELECT Statement Structure
```sql
SELECT * FROM users WHERE user_name='leon'
```
- `SELECT *`: Retrieve all columns
- `FROM users`: Target table name
- `WHERE user_name='leon'`: Filter condition

### Embedded SQL in PHP (Vulnerable Example)
```php
<?php
$uname = $_POST['uname'];
$passwd = $_POST['password'];

$sql_query = "SELECT * FROM users WHERE user_name= '$uname' AND password='$passwd'";
$result = mysqli_query($con, $sql_query);
?>
```

**Key Vulnerability Points:**
- Direct insertion of user input into SQL query
- No input validation or sanitization
- Variables retrieved from POST request without filtering

---

## SQLi Attack Fundamentals

### How SQLi Works
- User input directly concatenated into SQL queries
- Special characters not filtered or escaped
- Allows modification of intended SQL logic

### Basic Attack Example
**Normal Input:**
- User enters: `leon`
- Query becomes: `SELECT * FROM users WHERE user_name= 'leon'`

**Malicious Input:**
- User enters: `leon'+!@#$`
- Query becomes: `SELECT * FROM users WHERE user_name= 'leon'+!@#$`
- Breaks query syntax and can be exploited

### Attack Capabilities
- **Query**: Extract data from database
- **Insert**: Add malicious data
- **Modify**: Alter existing records
- **Delete**: Remove data
- **OS Commands**: Execute system commands (in some cases)

---

## Key Technical Points

### mysqli_query Function
- `i` stands for "improved" (not injection)
- PHP function for database interaction
- Executes SQL queries against database

### Vulnerability Root Cause
- **Lack of input validation**
- **Direct string concatenation**
- **No parameterized queries**
- **Missing special character filtering**

---

## Attack Methodology Overview
1. **SQL Enumeration**: Identify database structure
2. **Database Fingerprinting**: Determine DB type and version
3. **Manual Exploitation**: Craft custom payloads
4. **Automated Exploitation**: Use tools for systematic testing

---

## Database Types & Syntax Details

### MySQL Database

#### Connection Commands
```bash
# Connect to remote MySQL instance
mysql -u root -p'root' -h 192.168.50.16 -P 3306

# If TLS/SSL error occurs, add:
mysql -u root -p'root' -h 192.168.50.16 -P 3306 --skip-ssl
```

#### Core MySQL Functions & Commands
```sql
-- Get database version
SELECT version();

-- Get current user and hostname
SELECT system_user();

-- List all databases
SHOW databases;

-- Select specific database
USE database_name;

-- Get user password hash
SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';
```

#### MySQL System Information
- **Default Port**: 3306
- **Password Storage**: Caching-SHA-256 algorithm in `authentication_string` field
- **Default Databases**: `information_schema`, `mysql`, `performance_schema`, `sys`
- **Related**: MariaDB (open-source MySQL fork)

#### MySQL Query Structure
```sql
SELECT * FROM users WHERE user_name='leon'
```

---

### Microsoft SQL Server (MSSQL)

#### Connection Commands
```bash
# Connect via Impacket (from Kali Linux)
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

# Windows native connection
sqlcmd -S server_name -U username -P password
```

#### Core MSSQL Functions & Commands
```sql
-- Get OS and SQL Server version
SELECT @@version;

-- List all databases
SELECT name FROM sys.databases;

-- Inspect tables in specific database
SELECT * FROM database_name.information_schema.tables;

-- Query specific table with schema
SELECT * FROM database_name.dbo.table_name;
```

#### MSSQL System Information
- **Default Port**: 1433
- **Protocol**: Tabular Data Stream (TDS)
- **Authentication**: NTLM or Kerberos
- **Default Databases**: `master`, `tempdb`, `model`, `msdb`
- **Schema**: Uses `dbo` (database owner) schema by default

#### MSSQL Query Syntax Differences
- **Command Termination**: Semicolon + `GO` on separate line (for sqlcmd)
- **Remote Queries**: Can omit `GO` statement (not part of TDS protocol)
- **Schema Specification**: `database.schema.table` format (e.g., `offsec.dbo.users`)

#### MSSQL Connection Example Output
```
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
```

#### MSSQL Enumeration Examples
```sql
-- List tables in custom database
SELECT * FROM offsec.information_schema.tables;

-- Query user data with schema
SELECT * FROM offsec.dbo.users;
```

---

### Database Comparison Summary

| Feature | MySQL | MSSQL |
|---------|-------|-------|
| **Default Port** | 3306 | 1433 |
| **Version Function** | `version()` | `@@version` |
| **User Function** | `system_user()` | `USER_NAME()` |
| **List Databases** | `SHOW databases;` | `SELECT name FROM sys.databases;` |
| **Schema Format** | `database.table` | `database.schema.table` |
| **Command Tools** | `mysql` | `sqlcmd`, `impacket-mssqlclient` |
| **Authentication** | Username/Password | NTLM, Kerberos, SQL Auth |

---

### Key Technical Notes

#### MySQL Specifics
- **Connection**: Direct TCP connection on port 3306
- **Error Handling**: `ERROR 2026 (HY000)` for TLS/SSL issues
- **Password Hashing**: Modern versions use Caching-SHA-256
- **Cloud Deployment**: Available on AWS RDS, Google Cloud SQL, Azure Database

#### MSSQL Specifics
- **Windows Integration**: Native Windows ecosystem integration
- **Protocol**: TDS (Tabular Data Stream) for communication
- **Authentication**: Windows Authentication preferred over SQL Authentication
- **Tools**: 
  - **Windows**: `sqlcmd` (built-in)
  - **Linux**: `impacket-mssqlclient` (Impacket framework)
- **Cloud Deployment**: Azure SQL Database, AWS RDS for SQL Server

#### Important Considerations
- **Syntax Variations**: Each database has unique functions and syntax
- **Default Databases**: Always present, contain system information
- **Custom Databases**: Target for actual application data
- **Schema Awareness**: MSSQL requires schema specification (`dbo` is default)
