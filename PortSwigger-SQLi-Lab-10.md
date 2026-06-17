Lab 10: SQL Injection UNION Attack, Retrieving Multiple Values in a Single Column
Objective: Exfiltrate multiple sensitive data fields (username and password) through a restrictive UNION-based SQL injection where only a single column supports string data.
Vulnerability Analysis: The application’s product category filter is vulnerable to SQL injection. While the backend query allows for UNION-based data extraction, the schema enforces strict data-type constraints. To extract multiple string values (credentials) when only one text-compatible column is available, the attacker must utilize database-specific string concatenation techniques.
Exploitation Methodology:
1. Query Geometry & Data Type Mapping Initial testing with an injected single quote (') resulted in an Internal Server Error, confirming the injection point. I then utilized the ORDER BY technique to enumerate the backend query's column count:
' ORDER BY 1 -- (Success)
' ORDER BY 2 -- (Success)
' ORDER BY 3 -- (Error) This established a strict two-column requirement. Next, I tested data-type compatibility using NULL placeholders:
' UNION SELECT 'a', NULL -- (Error)
' UNION SELECT NULL, 'a' -- (Success) This critical step confirmed that only the second column accepts string data, meaning a standard extraction payload (SELECT username, password) would fail due to a type-mismatch in the first column.
2. Database Fingerprinting To determine the correct syntax for string concatenation, I first fingerprinted the RDBMS:
' UNION SELECT NULL, @@version -- (Error - Rules out MySQL/MSSQL)
' UNION SELECT NULL, version() -- (Success) The successful execution confirmed a PostgreSQL environment.
3. Data Exfiltration via Concatenation Because PostgreSQL utilizes the || operator for string concatenation, I constructed a payload to merge the username and password columns from the users table into the single available text column. To prevent the data from blending together into an unreadable format, I injected a literal string delimiter ('_') between the fields.
Final Payload: ' UNION SELECT NULL, username || '_' || password FROM users --
Impact: The application successfully executed the concatenated query, outputting the credentials directly into the user interface in a highly readable format:
administrator_z6rnb4w2cyr2bqgzlvlh
carlos_30pq71pn8e3evquafv9h
wiener_8957d44xdyuu8xugwu3s
By parsing out the delimiter, I successfully extracted the administrator's password and authenticated to the application, achieving total account takeover.
