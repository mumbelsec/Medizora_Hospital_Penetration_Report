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
