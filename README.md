# Medizora_Hospital_Penetration_Test_Report
Black box penetration result

**Target:** medizorahospital.com

**Prepared by:** mumbelsec

**Classification:** Confidential

### **1. Executive Summary**

I was assigned to conduct a black-box penetration test on the web infrastructure of Mediroza General Hospital (https://medirozahospital.com), with written client authorisation. The goal was to identify vulnerabilities, demonstrate their real-world impact, and recommend security improvements.

The assessment uncovered seven vulnerabilities ranging from Medium to Critical severity. The most significant was a SQL injection flaw on the patient portal login page, allowing authentication bypass without credentials. This exposed three confidential patient lab report PDFs. Cracking the PDF encryption revealed metadata pointing to a forgotten database backup in a publicly accessible folder — containing the salaries of all 30 hospital employees and shareholding details of 10 shareholders.

Overall, the target's security posture is poor. Multiple critical vulnerabilities would allow an unauthenticated attacker to access confidential patient data, staff financial records, and corporate ownership information with minimal effort.
