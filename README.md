# Medizora_Hospital_Penetration_Test_Report
Black box penetration result

**Target:** medizorahospital.com

**Prepared by:** mumbelsec

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

#### **3.3 Finding 2 — SQL Injection Login Bypass**

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

#### **3.4 Finding 3 — Confidential PDFs Accessible After Login Bypass**

**Risk Rating:** High
**Location:** `patient/reports/`

**Steps Taken**

After logging in with the SQL injection payload, I was directed to the patient portal which listed three downloadable PDF files:
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf

I downloaded all three files.
