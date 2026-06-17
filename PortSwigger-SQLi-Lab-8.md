Lab 8: SQL Injection UNION Attack, Finding a Column Containing Text
Objective: Map the data types of the backend database query to identify columns capable of accepting string payloads, preparing the vulnerability for text-based data exfiltration.
Vulnerability Analysis: The application's product category filter is vulnerable to a UNION-based SQL injection. To successfully exfiltrate sensitive data (such as usernames, passwords, or session tokens), the attacker must first identify which of the original query's columns utilize a string or text-compatible data type. Attempting to inject text into an integer or boolean column will result in a data-type mismatch error, neutralizing the attack.
Exploitation Methodology:
1. Injection Validation & Query Geometry Initial testing with an injected single quote (') resulted in an Internal Server Error, confirming the injection vector. To perform a UNION attack, I first enumerated the backend query's column count using the ORDER BY technique:
' ORDER BY 1 -- (Success)
' ORDER BY 2 -- (Success)
' ORDER BY 3 -- (Success)
' ORDER BY 4 -- (Error) This confirmed the backend query requests exactly three columns.
2. Data Type Mapping Knowing the geometry consists of three columns, I constructed a UNION SELECT payload using NULL values to bypass initial type-matching errors. I then systematically replaced each NULL with a static string ('a') to test the underlying column's data type acceptance.
Test 1: ' UNION SELECT 'a', NULL, NULL -- (Result: Error - Incompatible type)
Test 2: ' UNION SELECT NULL, 'a', NULL -- (Result: Success - String accepted)
Test 3: ' UNION SELECT NULL, NULL, 'a' -- (Result: Error - Incompatible type)
This enumeration confirmed that only the second column in the backend SELECT statement supports string data.
3. Exploitation / Proof of Concept To validate the findings and complete the proof of concept, I injected the lab's designated target string (bqIlxM) into the identified string-compatible column.
Final Payload: ' UNION SELECT NULL, 'bqIlxM', NULL --
Impact: The application successfully executed the payload, reflecting the injected string on the frontend user interface. By successfully identifying a text-compatible column, the vulnerability is now fully prepared for advanced exfiltration payloads, such as dumping credentials or system version strings.
