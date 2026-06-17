Lab 1: SQL Injection in WHERE Clause Allowing Retrieval of Hidden Data
Objective: Bypass the application's product filter to display unreleased or hidden categories.

Vulnerability Analysis:
The application filters products by category using a SQL query, likely structured as:
SELECT * FROM products WHERE category = '[user_input]' AND released = 1

By injecting a single quotation mark (') into the category parameter, the application returned an Internal Server Error. This indicated a failure in input sanitization, allowing for direct interaction with the backend database.

Exploitation Methodology:

Injection Point: The category parameter in the URL.

Payload Generation: To bypass the released = 1 constraint, I injected a boolean condition that is always true. The payload used was: ' OR 1=1 --

Execution: The injected payload altered the backend query to:
SELECT * FROM products WHERE category = '' OR 1=1 --' AND released = 1

Impact: The comment sequence (--) neutralized the remainder of the query (the released check). Because 1=1 is universally true, the database returned all rows in the products table, effectively exposing unreleased and hidden items to the user interface.
