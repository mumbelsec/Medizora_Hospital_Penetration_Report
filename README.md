# Medizora_Hospital_Penetration_Test_Report
**Test_type:** Black box penetration

**Target:** medizorahospital.com

**Classification:** Confidential

### **1. Executive Summary**

I was assigned to conduct a black-box penetration test on the web infrastructure of Mediroza General Hospital (https://medirozahospital.com), with written client authorisation. The goal was to identify vulnerabilities, demonstrate their real-world impact, and recommend security improvements.

The assessment uncovered seven vulnerabilities ranging from Medium to Critical severity. The most significant was a SQL injection flaw on the patient portal login page, allowing authentication bypass without credentials. This exposed three confidential patient lab report PDFs. Cracking the PDF encryption revealed metadata pointing to a forgotten database backup in a publicly accessible folder — containing the salaries of all 30 hospital employees and shareholding details of 10 shareholders.

Overall, the target's security posture is poor. Multiple critical vulnerabilities would allow an unauthenticated attacker to access confidential patient data, staff financial records, and corporate ownership information with minimal effort.

### **2.1 Scope**

The assessment was conducted strictly on the target domain. Any testing outside this domain was not performed.

### **2.2 Tools Used**

- **curl** — command line tool for sending HTTP requests and reading web server responses
- **Browser developer tools** — for inspecting page source and login form behaviour
- **NetworkWalks Hash Calculator** — for extracting password hashes from PDF files
- **NetworkWalks Password Cracker** — for cracking PDF password hashes using wordlists
- **qpdf** — for decrypting password-protected PDF files after cracking
- **exiftool** — for reading hidden metadata from PDF files
- **wget** — for downloading files from the web server
- **ChatGPT** — for converting raw SQL data into readable tables
- 
### **3. Findings and Proof of Exploitation**
#### **3.1 Summary Table**

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `patient/login.php` | ![Medium](https://img.shields.io/badge/Medium-blue) |
| 2 | SQL injection login bypass | `patient/login.php` | ![Critical](https://img.shields.io/badge/Critical-red) |
| 3 | Encrypted PDFs accessible after login bypass | `patient/reports/` | ![High](https://img.shields.io/badge/High-yellow) |
| 4 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | ![High](https://img.shields.io/badge/High-yellow) |
| 5 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | ![Medium](https://img.shields.io/badge/Medium-blue) |
| 6 | Forgotten backup folder with directory listing enabled | `old/` | ![Critical](https://img.shields.io/badge/Critical-red) |
| 7 | Confidential staff salaries and shareholder data in plain text | `old/mediroza_db_backup_2019.sql` | ![Critical](https://img.shields.io/badge/Critical-red) |

#### **3.2 Discovery 1 — Username Enumeration**

**Risk Rating:** Medium
**Location:** `patient/login.php`

**Steps Taken**

I opened the patient portal login page and tested the error messages by entering different inputs.

First, I entered a username that I assumed would not exist:
username: bob
password: test123

**Response:** `Username not found`

Then I tried a common default username:
username: admin
password: test123

**Response:** `Incorrect password`

The two different messages confirmed that `admin` is a valid account on the system. I now had the username and only needed to find the password.

#### **3.3 Discovery 2 — SQL Injection Login Bypass**

**Risk Rating:** Critical
**Location:** `patient/login.php`

**Steps Taken**

I tested the username field for SQL injection by typing a single quote:
username: admin’
password: test123

**Response:**

Warning: mysqli_query(): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘’’ at line 1

The database error confirmed the field was injectable. The application was building its SQL query like this behind the scenes
I then crafted the classic SQL injection bypass payload.

username: admin’ --
password: anything

The password check is ignored completely. The query returns the admin row and I am logged in.

#### **3.4 Discovery 3 — Confidential PDFs Accessible After Login Bypass**

**Risk Rating:** High
**Location:** `patient/reports/`

**Steps Taken**

After logging in with the SQL injection payload, I was directed to the patient portal which listed three downloadable PDF files:

patient_report_1.pdf

patient_report_2.pdf

patient_report_3.pdf

I downloaded all three files.

#### **3.5 Discovery 4 — Weak PDF Passwords Crackable with a Wordlist**

**Risk Rating:** High

**Steps Taken**

I used the NetworkWalks Hash Calculator to extract a crackable hash from each PDF, then ran each hash through the NetworkWalks Password Cracker.

Reports 1 and 2 cracked immediately using the built-in 100-word default wordlist. 
Report 3 did not crack with the built-in list. I switched to a larger wordlist (JTR default password list) and ran the attack again.

I then opened each PDF with its cracked password and read the confidential patient medical data inside.

#### **3.6 Discovery 5 — Sensitive Metadata in Patient PDF Files**

**Risk Rating:** Medium

**Steps Taken**

I first decrypted report 3 using `exiftool`, since it can read all metadata fields. A locked PDF only shows the encryption type.

The metadata revealed the author name and a comment:

```
Comments: DB backup moved to /old before site migration, do not delete
```
#### **3.7 Discovery 6 — Forgotten Backup Folder with Directory Listing Enabled**

**Risk Rating:** Critical

**Steps Taken**

During recon, I noted `/old` in the robots.txt file. The metadata clue from Finding 5 confirmed this was the location of the backup. I opened the folder in the browser:https://medirozahospital.com/old/

The directory listing was enabled, and I could see the backup file immediately:
mediroza_db_backup_2019.sql

I downloaded it using `wget`:
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql

#### **3.8 Discovery 7 — Confidential Staff Salaries and Shareholder Data in Plain Text**

**Risk Rating:** Critical

**Steps Taken**

I opened the SQL file in a text editor. The data was in raw SQL format. I copied the staff `INSERT INTO` rows and used ChatGPT to convert them into a readable table:
Present this SQL data as a readable table showing name, job title, department and monthly salary.

Then I did the same for the shareholders:
Present this SQL data as a readable table showing shareholder name, share percentage and share class.

I was able to read the private salary of every employee and the ownership structure of the entire hospital.

I also confirmed that the Author field in the PDF metadata corresponds with the IT Systems Administrator listed in the staff table — connecting all three findings into a single attack chain.

### **4. Full Attack Chain Summary**

The following is the complete path from zero access to full data exposure, showing how each finding led to the next.

- **Step 1:** Robots.txt revealed three hidden folders: `/patient`, `/staff`, and `/old`.
- **Step 2:** The patient login page showed different error messages for wrong username and wrong password, confirming `admin` as a valid account. *(Finding 1)*
- **Step 3:** A single quote in the username field triggered a database error, confirming SQL injection. *(Finding 2)*
- **Step 4:** The payload `admin' --` bypassed the login entirely and gave access to the patient portal. *(Finding 2)*
- **Step 5:** Three confidential patient PDF lab reports were downloaded from the portal. *(Finding 3)*
- **Step 6:** The NetworkWalks Password Cracker cracked the PDF passwords using wordlists. Reports 1 and 2 used the built-in list; Report 3 required the larger JTR list. *(Finding 4)*
- **Step 7:** `exiftool` on the unlocked Report 3 revealed a hidden comment by j.malik pointing to `/old`. *(Finding 5)*
- **Step 8:** The `/old` folder had directory listing enabled, exposing the database backup. *(Finding 6)*
- **Step 9:** The backup contained salaries of 30 employees and shareholding details of 10 shareholders in plain text. *(Finding 7)*

- ### **5. Recommendations and Remediation**

#### **5.1 Fix Username Enumeration**

Change the login page to show a single generic message for any failed login attempt, regardless of whether the username or password was wrong. For example: *"Invalid credentials. Please try again."*

#### **5.2 Fix SQL Injection**

Replace the current login query with a parameterised query or prepared statement. This separates the SQL code from the user input so that no amount of crafted input can change the query structure. **This is the most important fix in this report.**

```php
// Safe example using PHP PDO prepared statement
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);
```
#### **5.3 Fix PDF Access and Password Strength**

Move the PDF files outside the web root so they cannot be served directly by the web server. Enforce access through a server-side script that checks authentication before serving any file. If passwords are used, enforce a minimum length of 12 characters with a mix of uppercase, lowercase, numbers, and symbols.

#### **5.4 Strip PDF Metadata**

Remove all metadata from files before distributing them. Staff notes or internal comments must never be stored inside files that leave the organisation. Use the following command to strip all metadata:

```bash
exiftool -all= patient_report_3.pdf
```

#### **5.5 Fix Directory Listing and Remove the Backup**

Disable directory listing on all folders by adding `Options -Indexes` to the Apache configuration or `.htaccess` file. Delete the database backup from the `/old` folder immediately. Database backups must never be stored inside the web root. Store them in a private, access-controlled location outside the publicly accessible part of the server.

### **6. Conclusion**

This assessment found a complete attack chain from the login page to highly sensitive internal data. No sophisticated tools or knowledge were required. All vulnerabilities found in this report are well-known and have well-established fixes. I recommend the client address all Critical and High findings immediately before the system is used to store or serve real patient data.

### **Author**

**mumbelsec**

BO83/Cybersecurity

WEEK4 FINAL PROJECT

*Organization:* Networkwalks
