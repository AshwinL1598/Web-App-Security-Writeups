# Web Application Security & Vulnerability Assessments

Welcome to my security research and portfolio repository. This space is dedicated to documenting hands-on vulnerability assessments, manual penetration testing methodologies, and technical writeups from simulated and lab environments (including PortSwigger Web Security Academy). 

The primary focus of these exercises is to analyze flaws at the root level, understand database-specific exploitation mechanics, and derive actionable remediation strategies essential for Security Operations Center (SOC) analysis and defensive engineering.

## 🛠️ Core Skills & Technologies Demonstrated
* **Analysis Tools:** Burp Suite Professional/Community (Proxy, Repeater, Intruder, Decoder)
* **Database Management Systems (RDBMS):** PostgreSQL, Oracle Database, MySQL, Microsoft SQL Server
* **Vulnerability Vectors:** SQL Injection (In-band, UNION-based, Boolean-based Blind), Authentication Bypass, Schema Enumeration, and Data Exfiltration
* **Methodologies:** Manual boundary testing, fuzzing, automated payload iteration (Intruder Cluster Bomb), and cross-database fingerprinting

---

## 📂 Repository Structure & Writeups

Each file in this repository contains a detailed technical breakdown, including the objective, backend flaw analysis, step-by-step exploitation path, and the ultimate operational impact.

### 🔹 SQL Injection (SQLi) Series
* **[Lab 1: WHERE Clause Hidden Data](./PortSwigger-SQLi-Lab-1.md)**
  * *Concepts:* Input sanitization failure, modifying boolean logic (`OR 1=1`), bypassing business constraints.
* **[Lab 2: Authentication Bypass](./PortSwigger-SQLi-Lab-2.md)**
  * *Concepts:* Query truncation via comments (`--`), exploiting single-row backend expectations to log in as administrative users without passwords.
* **[Lab 3: Oracle UNION Attack - Version Discovery](./PortSwigger-SQLi-Lab-3.md)**
  * *Concepts:* Query geometry mapping (`ORDER BY`), testing column data types, handling Oracle-specific structural constraints (`FROM DUAL`), system table interrogation (`v$version`).
* **[Lab 4: MySQL/MSSQL UNION Attack - Version Discovery](./PortSwigger-SQLi-Lab-4.md)**
  * *Concepts:* Using `NULL` as an RDBMS-neutral data type placeholder to satisfy query geometry, utilizing database-specific global variables (`@@version`).
* **[Lab 5 & 6: Schema Enumeration & Exfiltration (PostgreSQL & Oracle)](./PortSwigger-SQLi-Lab-5.md)**
  * *Concepts:* Blind schema mapping using data dictionaries (`information_schema.tables`/`all_tables`), targeted column discovery, and extracting sensitive tables.
* **[Lab 7 & 8: Query Geometry & Data Type Mapping](./PortSwigger-SQLi-Lab-7.md)**
  * *Concepts:* Systematically isolating string-compatible injection points within structural results using sequential string placement.
* **[Lab 9 & 10: Advanced Data Consolidation & Concatenation](./PortSwigger-SQLi-Lab-10.md)**
  * *Concepts:* Overcoming strict data-type constraints in multi-value queries via cross-database string concatenation (`||` operator) and explicit string delimiters (`_`).
* **[Lab 11: Boolean-Based Blind SQL Injection](./PortSwigger-SQLi-Lab-11.md)**
  * *Concepts:* Inference-based data extraction using an application's conditional text behavior as a binary oracle. Leveraging Burp Suite Intruder (Cluster Bomb) and conditional evaluation (`SUBSTRING()`, `LENGTH()`) to reconstruct administrative passwords character-by-character.

---

## 🛡️ Defensive Perspective (SOC & Incident Response)
Understanding how these flaws are manipulated directly translates to identifying malicious signatures in application traffic logs. Throughout these labs, key detection metrics include:
* Monitoring for anomalous database errors (500 Internal Server Errors) stemming from unhandled SQL syntax.
* Identifying URL/cookie payloads containing heavy administrative keywords, Boolean operators (`AND`, `OR`), system table queries, or database comment structures (`--`, `#`).
* Detecting high-frequency, structured traffic patterns indicative of automated brute-forcing or character extraction tools (e.g., Intruder, SQLmap).

---
*Disclaimer: All assessments and writeups contained within this repository were performed in authorized, sandboxed educational lab environments for training purposes only.*
