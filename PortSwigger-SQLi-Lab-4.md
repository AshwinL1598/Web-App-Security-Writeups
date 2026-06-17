Lab 4: SQL Injection UNION Attack, Determining Database Version (MySQL / Microsoft)
Objective: Identify the underlying database technology and extract its exact version number using a UNION-based SQL injection attack.
Vulnerability Analysis: The application's product category filter is vulnerable to SQL injection. Because the application processes the injected input and reflects the database output directly on the frontend, it is an ideal candidate for a UNION-based data exfiltration attack. The goal is to append a secondary query that extracts environmental variables from the database.
Exploitation Methodology:
1. Enumerating Column Count To execute a successful UNION attack, I first needed to determine the exact number of columns returned by the backend SELECT statement. I used the ORDER BY clause to test this incrementally:
' ORDER BY 1# (Executed successfully)
' ORDER BY 2# (Executed successfully)
' ORDER BY 3# (Resulted in an Internal Server Error) This confirmed that the backend query expects exactly two columns.
2. Database Fingerprinting To verify the database type, I tested database-specific comment indicators. The use of the hash symbol (#) successfully commented out the remainder of the backend query without throwing a syntax error, heavily indicating the presence of a MySQL database (as opposed to Oracle, which relies strictly on --).
3. Payload Construction & Data Extraction Knowing the column count is two, and suspecting a MySQL or Microsoft SQL Server environment, I structured a UNION SELECT payload to extract the system version.
To satisfy the UNION operator's requirement for an equal number of columns, I utilized NULL as a placeholder for the second column. NULL ensures there are no data-type mismatch errors when appending the injected query.
Payload Used: ' UNION SELECT @@version, NULL#
4. Impact The application successfully executed the query, dumping the @@version environment variable into the first column of the user interface. This exposed the exact version details of the MySQL/Microsoft database, providing critical reconnaissance data that could be used to research further CVEs or targeted exploits.
