# PortSwigger Web Security Academy: SQL Injection (SQLi) Lab 12

## Lab 12: Blind SQL Injection with Conditional Errors

### Objective
Exfiltrate sensitive account data from a completely blind database environment where standard conditional text changes are absent, by forcing the database to trigger a runtime error (`500 Internal Server Error`) when a specific boolean condition evaluates to true.

---

### Backend Flaw Analysis
The application processes a tracking cookie (`TrackingId`) within a backend SQL query loop. Unlike traditional blind SQL vulnerabilities, the application's frontend content remains entirely static regardless of whether the query evaluates to true or false. 

To infer data in this restrictive environment, an attacker must leverage an **Error-Based Logic Oracle**. By injecting a database-specific conditional statement (such as a `CASE` block) that intentionally triggers a runtime exception (e.g., a division-by-zero math error), the database can be forced to halt execution. This causes the web server to return an HTTP 500 Internal Server Error, providing a clear binary indicator (200 OK vs 500 Internal Server Error) to verify assertions.

---

### Exploitation Methodology

#### 1. Injection Validation & Database Fingerprinting
Initial boundary testing was conducted by inserting a single quote (`'`) into the cookie value, yielding an HTTP 500 error. Re-balancing the string boundaries with a pair of quotes (`''`) restored the page to an HTTP 200 OK, confirming the parameter was unsanitized and vulnerable to query manipulation.

To identify the database management system (DBMS) without breaking the trailing application logic, string concatenation operators were used instead of query truncation comments (`--`). Truncating the query prematurely can destroy syntax boundaries required by the application logic, causing persistent errors. By using concatenation, the payload is seamlessly injected into the middle of the existing query string.

* **Payload:** `' || (SELECT '') || '`  
  *Result:* HTTP 500 Error. This syntax failure ruled out PostgreSQL and MySQL.
* **Payload:** `' || (SELECT '' FROM dual) || '`  
  *Result:* HTTP 200 OK. The application's acceptance of the dummy table `dual` explicitly fingerprinted the backend as an **Oracle Database**.

The `||` operator acts as the standard string concatenation mechanism in Oracle, effectively gluing the injected subquery to the original tracking ID value. The statement `SELECT ''` simply instructs the database to return an empty string placeholder, verifying that arbitrary query processing is possible.

#### 2. Verifying Table and Object Existence
Before constructing an error oracle, the existence of the targeted data structures had to be verified. Testing a direct query against a table like `users` (e.g., `' || (SELECT '' FROM users) || '`) fails with an error because a query returning multiple rows cannot be concatenated directly into a single string parameter.

To resolve this constraint, the query must be restricted to a single row. Because **Oracle databases do not support the `LIMIT` keyword**, the built-in `rownum` parameter must be used instead to control row allocation:

* **Payload:** `' || (SELECT '' FROM users WHERE rownum=1) || '`  
  *Result:* HTTP 200 OK. This confirmed the precise existence of the `users` table.
* **Payload:** `' || (SELECT '' FROM users WHERE username='administrator') || '`  
  *Result:* HTTP 200 OK. This confirmed that a record matching the username `administrator` existed within the database.

#### 3. Engineering the Conditional Error Oracle
Because a simple boolean statement does not change the page content, a runtime error was engineered using an Oracle-compatible `CASE` structure combined with an unhandled mathematical exception (`1/0` division by zero).

* **Payload:** ```sql
  ' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || '

### 4. Enumerating Password Length

Using the validated error oracle, a length constraint was appended to the query logic. Burp Suite Intruder was used to iterate through prospective integer values:
Payload Template: ```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND LENGTH(password)=[§index§]) || '
Result: The application returned an HTTP 200 OK for indices 1 through 19. When the payload index reached exactly **20**
the condition evaluated to true, triggering the division-by-zero exception and returning an HTTP 500 Internal Server Error. This confirmed the administrative password was exactly 20 characters long.
Automated Data Exfiltration via Response Analysis
With the password structure mapped, Oracle's SUBSTR() function was introduced into the error-based CASE framework to isolate and test individual string characters. A Cluster Bomb attack was executed via Burp Suite Intruder to iterate through both the character positions and alphanumeric character sets:

Payload Template: ```sql
' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND SUBSTR(password,[§position§],1)='[§character§]') || '

### 5. Attack Analysis & Reassembly:

After executing the automated brute-force attack, the resulting data matrix was sorted exclusively by HTTP Status Code. Because the oracle dictates that a True condition results in a database crash, any response yielding an HTTP 500 Internal Server Error indicated a successful character match for that specific index position. All incorrect guesses evaluated to false, skipped the calculation, and yielded an HTTP 200 OK.

By extracting the specific characters linked to the HTTP 500 status codes sequentially from position 1 to 20, the full cleartext administrative password was successfully reassembled.

### 6. Operational Impact

By engineering a sophisticated conditional error framework, the blind database constraints were entirely bypassed. This manual enumeration resulted in the extraction of highly sensitive administrative credentials, leading to total authentication bypass and full takeover of the target application workspace.
