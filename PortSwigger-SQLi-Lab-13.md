# PortSwigger Web Security Academy

## Lab 13: Visible Error-Based SQL Injection

### Objective
Exploit a visible error-based SQL injection vulnerability to force a database data type conversion failure, systematically leaking sensitive administrative credentials directly through the application's frontend error messages.

---

### Backend Flaw Analysis
The application utilizes an unparameterized SQL query to process a tracking analytics cookie (`TrackingId`). Unlike blind SQL injection environments, this application is configured with verbose error handling enabled in production. When a database query encounters a runtime exception or syntax failure, the application extracts the raw relational database management system (RDBMS) error message and prints it directly into the HTTP response. 

An attacker can exploit this behavior by constructing a query that forces a data type conversion error using the `CAST()` function. If the database attempts to convert an incompatible data type (such as converting a text string into an integer), it will fail and print the value it failed to convert inside the error logs, effectively leaking sensitive data to the user interface.

---

### Exploitation Methodology

#### 1. Input Verification & Error Information Leakage
Initial fuzzing was conducted by injecting a single quote (`'`) into the tracking cookie value. The application responded with a detailed HTTP 500 error message:

Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'iJhVhWQRO04nyKPo''. Expected char

This error message provided critical structural insights:

It confirmed the backend query format: SELECT * FROM tracking WHERE id = 'INPUT'.

It verified that input is directly concatenated into the query string without sanitization.

It proved that verbose error messages were reflected in the user interface.

Injecting a SQL comment ('--) resolved the syntax error and returned an HTTP 200 OK, confirming that trailing query logic could be safely neutralized.

### 2. Profiling Database Constraints via Type Testing
To determine how the database processes logical expressions, sequential boolean and data-type functions were injected:

Payload: ' AND SELECT 1=1--

Result: ERROR: syntax error at or near "SELECT". This indicated an invalid syntax layout.

Payload: ' AND CAST((select 1) as int) --

Result: ERROR: argument of AND must be type boolean, not type integer.

This specific error message provided key environmental data. It confirmed that the database enforces strict type safety (indicative of PostgreSQL or similar engines), where the AND operator expects an explicit boolean evaluation (True/False) rather than an integer value.

To satisfy this constraint, the payload was re-engineered to perform a logical comparison against an integer index:

Payload: ' AND 1=CAST((SELECT 1) as int) --

Result: HTTP 200 OK. This confirmed the data type conversion syntax was fully functional and structurally aligned with the backend query.

### 3. Forcing Runtime Conversion Errors to Exfiltrate Data
With the CAST mechanism validated, the subquery was modified to target data from the users table instead of a static integer.

Payload: ' AND 1=CAST((SELECT username FROM users) as int) --

Result: ERROR: more than one row returned by a subquery used as an expression.

The database successfully processed the query but crashed because the subquery returned an entire array of usernames rather than a single value. To fix this structural constraint, the LIMIT 1 clause was introduced to restrict the database output to exactly one record.

Payload: ```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) as int) --

### Backend Execution Mechanics:
The database resolves the inner subquery first, extracting the string value "administrator" from the users table.

It then attempts to evaluate the outer function: CAST('administrator' AS int).

Because the alphabetic string "administrator" cannot be parsed into a numeric integer data type, the database engine encounters a fatal type conversion error and halts execution.

The web server catches the unhandled exception and prints the raw database error message to the screen:

ERROR: invalid input syntax for type integer: "administrator"

The administrative username was successfully extracted via the error string.

### 4. Credential Harvesting
The same data-type coercion methodology was applied to target the password column within the users table:

Payload: ```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) as int) --

Resulting Error Message: ERROR: invalid input syntax for type integer: "cy7gjtyi1wjzq5yg6ry5"

The database wrapper successfully leaked the cleartext password (cy7gjtyi1wjzq5yg6ry5) associated with the first row of the user table.

### Operational Impact
By taking advantage of verbose database error messages, the constraints of standard data extraction were bypassed entirely. Rather than executing time-intensive brute-force or boolean-inference attacks, sensitive administrative credentials were leaked instantaneously through data type coercion errors, leading to a complete compromise of the application portal.
