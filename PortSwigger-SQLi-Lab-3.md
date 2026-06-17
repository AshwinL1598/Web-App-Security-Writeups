Lab 3: SQL Injection UNION Attack, Determining Database Version on Oracle
Objective: Perform a UNION-based SQL injection to extract the underlying database version and banner details.

Vulnerability Analysis:
The application contains a SQL injection vulnerability in the category filter. Because the results of the query are reflected directly on the page, a UNION-based attack can be leveraged to retrieve data from other database tables.

Exploitation Methodology:

Determining Column Count: I used the ORDER BY clause to incrementally test the number of columns returned by the original query.

' ORDER BY 1 -- (Success)

' ORDER BY 2 -- (Success)

' ORDER BY 3 -- (Error)
This confirmed the original query returns exactly two columns.

Identifying Database Constraints & Data Types:
I attempted a standard UNION attack to verify data types: ' UNION SELECT 'a', 'a' --. This resulted in an error. Recognizing that the target environment was an Oracle database, I adjusted the syntax. Oracle databases require a FROM clause in all SELECT queries.

Refined Payload: ' UNION SELECT 'a', 'a' FROM DUAL --

The successful execution of this payload confirmed both columns accept string data types.

Extracting the Database Version:
With the column count and types verified, I queried the v$version view, which contains banner information for Oracle databases.

Final Payload: ' UNION SELECT banner, NULL FROM v$version --

Impact: The query successfully appended the database version details to the application's response, exposing the internal system architecture.
