# Insecure Direct Object References (IDOR)

IDOR vulnerabilities occur when an application exposes a direct reference to an internal object, like database records or files, allowing attackers to manipulate these references to access unauthorized data.

## Overview
- Direct references to internal objects (IDs, filenames)
- No access control checks
- Common in REST APIs and URL parameters

## Example Vulnerabilities
- User profile access by ID manipulation
- File downloads via path manipulation
- Order/transaction access by reference number

*More detailed content will be added later* 