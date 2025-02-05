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

### **5. How to Spot Vulnerabilities in Requests**

| **Header/Field**    | **What to Check For**                          | **Potential Vulnerability**                   |
|---------------------|------------------------------------------------|-----------------------------------------------|
| **Host**            | Try changing the Host to another domain.       | **Host Header Injection**, SSRF               |
| **User-Agent**      | Modify to see if filters are bypassed.         | **WAF Bypass**                               |
| **Authorization**   | Test with invalid or missing tokens.           | **Broken Authentication**                    |
| **Referer/Origin**  | Change to see if CSRF protection exists.       | **CSRF Vulnerability**                       |
| **Cookie**          | Modify session cookies to see if access changes.| **Session Hijacking**                        |
| **Content-Type**    | Change from JSON to XML to trigger parsing issues.| **Content-Type Confusion**, **XXE**          |

---

### **6. Status Codes in Responses (What the Server Says Back)**

| **Code** | **Meaning**                  | **What to Do**                              |
|----------|------------------------------|---------------------------------------------|
| **200**  | OK (everything worked fine)  | Normal response. Test for hidden issues.    |
| **301/302** | Redirect                  | Check for **Open Redirects** vulnerabilities.|
| **400**  | Bad Request                  | Input might be wrong. Test with payloads.   |
| **401**  | Unauthorized                 | Needs authentication. Test tokens.          |
| **403**  | Forbidden                    | Access denied. Try bypass techniques.       |
| **404**  | Not Found                    | Check if paths are hidden.                  |
| **500**  | Internal Server Error        | Server-side issue. Might be **SQLi** or **LFI**.|

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

