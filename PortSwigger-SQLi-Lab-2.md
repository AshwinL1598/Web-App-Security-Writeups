Lab 2: SQL Injection Authentication Bypass
Objective: Bypass the authentication mechanism and log in to the application as the administrator user.

Vulnerability Analysis:
The login portal is vulnerable to SQL injection within the username field. The backend authentication query is assumed to be:
SELECT * FROM users WHERE username = '[user_input]' AND password = '[password_input]'

Exploitation Methodology:

Reconnaissance: Initial testing with ' and -- characters in the login fields generated an internal server error, confirming the presence of a SQL injection vulnerability.

Initial Attempt: I initially tested the payload administrator' OR 1=1 --. This resulted in an error, likely because the OR 1=1 condition returned multiple user records, breaking the application's expectation of a single user object.

Refined Payload: To exclusively target the administrator account, I modified the payload to simply terminate the query after the username string: administrator' --

Execution: By submitting this payload into the username field (with any arbitrary password), the backend query was manipulated to:
SELECT * FROM users WHERE username = 'administrator' --' AND password = '...'

Impact: The -- sequence commented out the password verification logic entirely. The database returned the administrator record, and the application granted full administrative access without requiring a valid password.

