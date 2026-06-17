Lab 9: SQL Injection UNION Attack, Retrieving Data from Other Tables
Objective: Leverage a UNION-based SQL injection vulnerability to enumerate the database schema, locate user credentials, and extract the administrator’s password to achieve account takeover.
Vulnerability Analysis: The application’s product category filter lacks proper input sanitization, allowing for direct interaction with the backend SQL queries. Because the application dynamically reflects query results on the frontend, an attacker can append secondary SELECT statements using the UNION operator to extract sensitive records from entirely separate database tables.
Exploitation Methodology:
1. Injection Validation & Query Geometry To verify the vulnerability, I injected a single quote ('), which resulted in a 500 Internal Server Error, indicating a break in the SQL syntax. I successfully restored the query structure using a SQL comment ('--). Next, I determined the backend query's column count using the ORDER BY technique:
' ORDER BY 1 -- (Success)
' ORDER BY 2 -- (Success)
' ORDER BY 3 -- (Error) This confirmed the query geometry expects exactly two columns.
2. Data Type Mapping & Database Fingerprinting I utilized a UNION SELECT statement with static string values to verify data type compatibility:
' UNION SELECT 'a', 'a' -- (Success) The successful execution without the FROM DUAL clause ruled out an Oracle database and confirmed both columns accept string input. To identify the specific database engine, I tested version extraction payloads:
' UNION SELECT @@version, NULL -- (Error - Rules out MySQL/MSSQL)
' UNION SELECT version(), NULL -- (Success) The database returned a PostgreSQL 12.22 banner, confirming the target environment.
3. Schema Enumeration (Tables & Columns) To locate the credentials, I interrogated the PostgreSQL information_schema rather than relying on default or assumed table names.
Targeting Tables: I queried information_schema.tables, successfully identifying the users table.
Targeting Columns: I queried information_schema.columns with a WHERE clause targeting the users table, successfully extracting the exact column names: username and password.
4. Data Exfiltration Having successfully mapped the target database structure, I constructed the final UNION payload to dump the contents of the users table into the application's user interface.
Final Payload: ' UNION SELECT username, password FROM users --
Impact: The payload successfully extracted and displayed the user database, including the administrator credentials (administrator : 1wgv1bdhhwfpxhfzugtg). Utilizing these credentials at the application's login portal resulted in a complete authentication bypass and full administrative compromise of the platform.
