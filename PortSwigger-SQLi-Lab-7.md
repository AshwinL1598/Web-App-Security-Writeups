Lab 7: SQL Injection UNION Attack, Determining Column Count
Objective: Determine the exact number of columns returned by the backend database query to prepare for a UNION-based data exfiltration attack.
Vulnerability Analysis: The application contains a SQL injection vulnerability within the product category filter. To successfully append a UNION SELECT payload, the injected query must return the exact same number of columns as the original backend query. Finding this integer is the mandatory first step of the exploitation chain.
Exploitation Methodology:
Initial Testing: I tested the category parameter by appending a single quote and a SQL comment ('--). This manipulated the query logic and generated an internal server error, confirming the parameter was improperly sanitized and vulnerable to injection.
To determine the query geometry, I utilized two distinct enumeration techniques:
Method 1: The ORDER BY Technique I injected an ORDER BY clause to sort the results by a specific column index. By incrementally increasing the index number until the database threw an error, I mapped the column boundaries.
' ORDER BY 1 -- (Executed successfully)
' ORDER BY 2 -- (Executed successfully)
' ORDER BY 3 -- (Executed successfully)
' ORDER BY 4 -- (Resulted in an error) The error on index 4 confirms that the backend SELECT statement requests exactly three columns.
Method 2: The UNION SELECT NULL Technique To verify that the UNION operator was not restricted and to corroborate the column count, I submitted a series of payloads using NULL values as placeholders. NULL is utilized because it satisfies the requirement for data type compatibility across all columns.
' UNION SELECT NULL -- (Error: Column count mismatch)
' UNION SELECT NULL, NULL -- (Error: Column count mismatch)
' UNION SELECT NULL, NULL, NULL -- (Executed successfully)
Impact: Both methodologies successfully confirmed the backend query returns exactly three columns. With the query geometry mapped, the vulnerability can now be escalated to extract sensitive data from other tables within the database.
