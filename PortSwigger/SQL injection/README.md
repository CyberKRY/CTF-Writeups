<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/SQL%20injection/Images/common-sql-injection-attacks.webp" alt="SQL" width="800" />
</p>

<h1>What is SQL Injection (SQLi)?</h1>

<b>SQL Injection (SQLi)</b> - is a class of web vulnerability where an application incorporates attacker-controlled input into SQL queries without proper validation or parameterization. This allows an attacker to manipulate database queries, leading to unauthorized data access, data modification, or complete compromise of the backend database.

<h2>Main types of SQL Injection</h2>

<b>In-band SQL Injection</b> — attacker receives results directly through the same channel used to inject the payload.

<b>Error-based</b> — database error messages reveal query structure or data.

<b>Union-based</b> — attacker appends additional queries to extract data from other tables.

<b>Blind SQL Injection</b> — the application does not return database errors or data directly, but the attacker infers information through application behavior.

<b>Boolean-based</b> — different responses indicate true/false conditions.

<b>Time-based</b> — response delays are used to infer query execution results.

<b>Out-of-band SQL Injection</b> — data is exfiltrated using a separate channel (e.g., DNS or HTTP requests), typically when in-band techniques are unavailable.



<h2>Injection points and query contexts</h2>

SQLi can occur in various query contexts: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `ORDER` `BY`, `LIMIT`, and even in stored procedures. Common injection points include URL parameters, POST bodies, cookies, HTTP headers, and JSON/XML inputs. Risk increases when dynamic query construction, string concatenation, or ORM misconfiguration is used instead of prepared statements.

<b>Impact</b>

* Unauthorized access to sensitive data (credentials, PII, financial data)

* Authentication bypass and privilege escalation

* Data modification or deletion

* Full database compromise and potential remote code execution (in some DBMS setups)

* Lateral movement to other systems within the infrastructure
