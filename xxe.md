# XML External Entity (XXE) Injection

XXE attacks target applications that parse XML input. These vulnerabilities occur when XML parsing is configured to process external entity references, which can lead to various security issues.

## Overview
- XML parsing of untrusted input
- External entity processing enabled
- Can lead to file disclosure, SSRF, DoS

## Common Attack Vectors
- File content disclosure
- Server-side request forgery
- Denial of service via entity expansion
- Local system file access

*More detailed content will be added later* 