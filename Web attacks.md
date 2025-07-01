# Introduction to Web Attacks

As web applications become more common and complex, protecting them from attacks is critical. Modern web apps present a vast attack surface, making them prime targets for attackers. Compromising a web app can lead to internal network breaches, data theft, service disruption, and financial loss. Even internal apps and APIs are at risk. This module covers three essential web attacks: how to detect, exploit, and prevent them.

---

## Web Attacks

### 1. HTTP Verb Tampering
#### What is HTTP Verb Tampering?
HTTP works by accepting various HTTP methods (verbs) at the start of a request. While most apps use GET and POST, web servers may accept other methods (HEAD, PUT, DELETE, etc.). If the server or app is not configured to restrict these, attackers can exploit this to bypass security controls or gain unauthorized access.

#### Common HTTP Verbs
| Verb    | Description                                                        |
|---------|--------------------------------------------------------------------|
| GET     | Retrieve data                                                      |
| POST    | Submit data                                                        |
| HEAD    | Like GET, but only returns headers                                 |
| PUT     | Write/upload data to a resource                                    |
| DELETE  | Delete a resource                                                  |
| OPTIONS | Show supported HTTP methods                                        |
| PATCH   | Apply partial modifications to a resource                          |

Some methods (PUT, DELETE) can be very sensitive if not properly restricted.

#### Two Main Types of HTTP Verb Tampering

1. **Insecure Configuration**
   - Caused by web server configs that only protect certain methods (e.g., GET/POST) but leave others (e.g., HEAD) unprotected.
   - **Example:**
     ```xml
     <Limit GET POST>
         Require valid-user
     </Limit>
     ```
     An attacker can use HEAD to bypass authentication and access restricted pages.

2. **Insecure Coding**
   - Caused by developers applying security checks (like input validation) only to certain methods, not all.
   - **Example:**
     ```php
     $pattern = "/^[A-Za-z\s]+$/";
     if(preg_match($pattern, $_GET["code"])) {
         $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
         ...SNIP...
     }
     ```
     Here, only GET input is validated, but the query uses $_REQUEST (GET/POST). An attacker can use POST to bypass the filter and exploit vulnerabilities like SQL injection.

**Key Point:**
- Restrict allowed HTTP methods in both server configuration and application logic.
- Apply security controls and input validation to all HTTP methods.

#### Example: Bypassing Basic Authentication
Let's look at a real-world example of exploiting insecure configuration HTTP Verb Tampering.

**Scenario:**
- File Manager web app with a restricted `/admin` directory
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

**Result:**
- HEAD request executes reset function without authentication
- No response body (HEAD method characteristic)
- Files successfully deleted, confirming the bypass

**Why it Works:**
- Server config only protects GET/POST:
```xml
<Limit GET POST>
    Require valid-user
</Limit>
```
- HEAD method falls outside these limits
- Function executes despite empty response

**Prevention:**
- Protect all HTTP methods:
```xml
<LimitExcept GET POST HEAD>
    Deny from all
</LimitExcept>
```
- Or restrict to only needed methods

#### Example: Bypassing Command Injection Protection
Let's look at another real-world example where HTTP Verb Tampering bypasses security filters.

**Scenario:**
- File Manager web app with file creation functionality
- Security filter blocks special characters in filenames
- Filter only checks POST parameters, not GET

**Step 1: Identify**
- Normal POST request gets blocked:
```http
POST /create.php HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 21

filename=test;ls;file2
```
Response: "Malicious character detected!"

**Step 2: Method Manipulation**
1. In Burp Suite:
   - Intercept the request
   - Right-click → "Change request method"
   - Burp converts POST to GET automatically:
```http
GET /create.php?filename=test;ls;file2 HTTP/1.1
Host: example.com
```

**Note:** When changing from POST to GET:
- POST body parameters move to URL query string
- Content-Type and Content-Length headers are removed
- URL-encoding may need adjustment

**Step 3: Command Injection Test**
```http
GET /create.php?filename=file1;touch%20file2; HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```
Result:
- Two files created: `file1` and `file2`
- Confirms successful filter bypass and command injection

**Why it Works:**
- Security filter checks only `$_POST['filename']`
- GET request uses `$_GET['filename']`
- Back-end still executes the command without filtering

**Prevention:**
```php
// Check all request methods
if (preg_match('/[;&|]/', $_REQUEST['filename'])) {
    die('Malicious character detected!');
}
```

---

### 2. Insecure Direct Object References (IDOR)
- **What is it?**
  - Occurs when apps expose direct references (IDs, filenames) to internal objects without proper access control.
  - Attackers manipulate these references to access unauthorized data (e.g., changing user ID in a URL).
- **Why is it dangerous?**
  - Allows attackers to access or modify other users' data.
- **Key Point:**
  - Enforce strong access controls on all object references.

---

### 3. XML External Entity (XXE) Injection
- **What is it?**
  - Exploits XML parsers that process untrusted input and allow external entities.
  - Attackers send malicious XML to read server files, leak sensitive data, or execute code.
- **Why is it dangerous?**
  - Can disclose sensitive files, credentials, or enable remote code execution.
- **Key Point:**
  - Disable external entity processing in XML parsers and use secure libraries.

---

Let's get started by discussing the first of these attacks in the next section.
