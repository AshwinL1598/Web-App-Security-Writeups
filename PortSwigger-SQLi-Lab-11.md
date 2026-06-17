
Lab 11: Blind SQL Injection with Conditional Responses
Objective: Exploit a boolean-based blind SQL injection vulnerability to systematically enumerate the database, determine password length, and extract the administrator’s password character-by-character to achieve account compromise.
Vulnerability Analysis: The application utilizes a tracking cookie (TrackingId) that is processed by the backend database without proper sanitization. Unlike in-band SQL injection, the results of the query are not reflected on the frontend. However, the application exhibits a conditional response: if the SQL query evaluates to True, the page displays a "Welcome back" message. If the query evaluates to False, the message disappears. This behavioral difference acts as a boolean oracle, allowing for the exfiltration of data through a series of true/false inferences.
Exploitation Methodology:
1. Verifying the Injection Point & Boolean Oracle To confirm the vulnerability, I manipulated the TrackingId cookie using basic boolean conditions:
TrackingId=xyz' AND 1=1 -- (Evaluated to True -> "Welcome back" message displayed)
TrackingId=xyz' AND 1=0 -- (Evaluated to False -> Message disappeared) This confirmed that I could ask the database true/false questions.
2. Database Schema Verification Before attempting data extraction, I verified the existence of the target table and user.
Checking for the users table: ' AND (SELECT 'x' FROM users LIMIT 1)='x' -- (Result: True)
Checking for the administrator account: ' AND (SELECT username FROM users WHERE username='administrator')='administrator' -- (Result: True)
3. Determining Password Length To configure an automated extraction attack, I first needed the exact length of the administrator's password. I used the LENGTH() function alongside Burp Suite Intruder (Sniper attack) to test incremental values.
Payload: ' AND (SELECT username FROM users WHERE username='administrator' AND LENGTH(password)>[§1§])='administrator' --
Result: By analyzing the HTTP responses, I determined the query evaluated to False at length > 20, confirming the password is exactly 20 characters long.
4. Data Exfiltration & Response Analysis With the password length confirmed, I utilized the SUBSTRING() function to extract the password one character at a time. I configured Burp Suite Intruder with a Cluster Bomb attack type to iterate through both the character position and the potential character values.
Payload: ' AND (SELECT SUBSTRING(password,[§position§],1) FROM users WHERE username='administrator')='[§character§]' --
Payload Set 1 (Position): Numbers 1 - 20.
Payload Set 2 (Character): Alphanumeric characters (a-z, 0-9).
Result Analysis: After launching the brute-force attack, the vast majority of the payloads resulted in a False query, meaning the application did not display the conditional response. By sorting the Intruder results and specifically filtering for HTTP responses that contained the "Welcome back" string, I was able to identify the True conditions. Each response containing this string confirmed the correct character for that specific position.
Outcome: By extracting the characters from these successful responses one by one, I successfully reassembled the complete 20-character password: 4ruwmpyjz2oek1ddxyby.
Impact: Utilizing the exfiltrated credentials, I successfully authenticated into the application as the administrator. This boolean-inference technique bypassed the limitations of a blind SQL environment, resulting in a complete account takeover. 
