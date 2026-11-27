# Mediroza General Hospital
## External Black-Box Web Application Penetration Testing

![Assessment](https://img.shields.io/badge/Assessment-Black--Box%20Penetration%20Testing-blue)
![Risk](https://img.shields.io/badge/Overall%20Risk-CRITICAL-red)
![Program](https://img.shields.io/badge/Networkwalks-Batch%20B084-orange)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-purple)

---

## 1. Project Overview

This repository documents an authorized external black-box web application penetration test conducted against the Mediroza General Hospital website as part of the Networkwalks Cybersecurity Internship — Batch B084, Week 4.

The assessment focused on identifying security weaknesses in the patient portal, publicly accessible resources, database backup exposure, authentication mechanisms, and encrypted PDF documents.

### Assessment Information

| Assessment | Details |
|---|---|
| Client | Mediroza General Hospital |
| Assessment Type | External Black-Box Web Application Penetration Test |
| Target | https://medirozahospital.com |
| Penetration Tester | Uwes Kulabkar |
| Program | Networkwalks Cybersecurity Internship |
| Batch | B084 — Week 4 |
| Testing Environment | Kali Linux, Firefox |
| Overall Risk | **CRITICAL** |

## 2. Executive Summary

The assessment identified multiple security weaknesses affecting the Mediroza General Hospital web application.

The most significant findings included:

- SQL injection in the patient login functionality, resulting in authentication bypass.
- Unauthenticated access to a publicly exposed SQL database backup.
- Weak password protection on three encrypted pathology reports.
- Sensitive information disclosure through PDF metadata.
- Username enumeration through inconsistent authentication responses.
- Directory listing on a legacy application directory.

The exposed database backup contained 30 staff records and 10 shareholder records. The assessment also identified internal identifiers and legacy backup references within PDF metadata.

These weaknesses demonstrate how multiple security issues can combine to increase the risk of unauthorized access and sensitive information disclosure.

## 3. Findings Overview

| ID | Vulnerability | Severity | CWE |
|---|---|---|---|
| F-01 | SQL Injection — Patient Portal Authentication Bypass | Critical | CWE-89 |
| F-02 | Unauthenticated Database Backup Disclosure | High | CWE-538 |
| F-03 | Weak Password Protection on Patient PDFs | High | CWE-521 |
| F-04 | Sensitive PDF Metadata Disclosure | Medium | CWE-200 |
| F-05 | Username Enumeration | Low | CWE-204 |
| F-06 | Directory Listing on Legacy Directory | Low | CWE-548 |

**Overall Assessment Risk: CRITICAL**

## 4. Scope and Methodology

### Scope

The assessment covered the assigned Mediroza General Hospital web application and directly associated files encountered during the authorized testing exercise.

### Testing Methodology

The assessment followed these phases:

1. Reconnaissance
2. Scanning and Enumeration
3. Initial Access and Authentication Testing
4. PDF Password Security Assessment
5. PDF Metadata Analysis
6. Evidence Collection and Reporting

### Tools and Technologies

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration and DNS intelligence |
| WhatWeb | Web technology fingerprinting |
| cURL | HTTP inspection and response analysis |
| Wget | File retrieval and exposure verification |
| Firefox Developer Tools | Manual application testing |
| grep and awk | Database inspection and record correlation |
| Networkwalks Hash Calculator | PDF password hash extraction |
| Networkwalks Password Cracker | Dictionary-based password testing |
| pdf2john | PDF hash extraction |
| John the Ripper | Offline password auditing |
| RockYou Wordlist | Dictionary-based password auditing |
| qpdf | PDF decryption and encryption analysis |
| ExifTool | PDF metadata examination |

## 5. Technical Assessment

### Phase 1 — Reconnaissance

**Objective:** Identify domain information, web technologies, server configuration and publicly disclosed application paths.

Activities:
- WHOIS domain enumeration.
- Web server and technology fingerprinting.
- HTTP response header inspection.
- robots.txt analysis.

**Key observations:**
- NameCheap was identified as the domain registrar.
- DNSSEC was reported as unsigned.
- LiteSpeed was identified through web technology fingerprinting.
- WhatWeb received an HTTP 403 response, while a separate cURL request returned HTTP/2 200.
- robots.txt disclosed the following paths:
  - `/patient/`
  - `/staff/`
  - `/old/`

**Evidence:** WHOIS information, WhatWeb output and robots.txt discovery.

### Phase 2 — Scanning and Enumeration

**Objective:** Investigate discovered application paths and identify publicly accessible files.

#### F-06: Directory Listing

The legacy `/old/` directory returned a directory index that exposed the following file:

`mediroza_db_backup_2019.sql`

**Observation:** Directory indexing allowed visitors to discover legacy application files.

#### F-02: Unauthenticated Database Backup Disclosure

The SQL backup was accessible without authentication.

**Observed response:**
- HTTP Status: 200 OK
- Content Type: text/x-sql
- File Size: 6,346 bytes

**Database inspection identified:**
- Staff table: 30 records.
- Shareholders table: 10 records.

Exposed information included names, contact information, national identifiers, salary information and ownership details.

**Impact:** Unauthorized access to sensitive organizational and financial information.

### Phase 3 — Initial Access and SQL Injection

**Objective:** Evaluate authentication responses, identify unsafe SQL input handling and verify patient portal access.

#### F-05: Username Enumeration

Testing identified different responses for existing and nonexistent usernames.

| Test Case | Observed Response |
|---|---|
| Nonexistent username | Username not found |
| Existing username | Incorrect password |

**Impact:** An attacker could identify valid usernames through authentication error messages.

#### F-01: SQL Injection — Authentication Bypass

Testing of the patient login identified a MySQL `mysqli_query()` syntax warning following malformed username input.

The observed behavior indicated unsafe SQL input handling.

**Observed test input:**

`admin' --`

**Result:** The login test provided access to the patient portal without a valid password.

The portal displayed the My Lab Reports section and three downloadable encrypted pathology reports.

**Technical explanation:**

The observed behavior was consistent with SQL injection caused by dynamically constructed SQL statements. A single quote can terminate a SQL string literal, while SQL comment syntax can alter the interpretation of the remaining query.

The server-side source code was not available, so the exact SQL implementation was not independently inspected.

**Impact:** Authentication bypass exposed protected patient portal functionality and access to the three encrypted reports.

### Phase 4 — PDF Password Security

**Objective:** Examine PDF encryption and evaluate the strength of password-based protection.

#### F-03: Weak Password Protection on Patient PDFs

Three pathology reports were downloaded from the patient portal and examined.

**Encryption observations:**

- PDF Version: 1.4
- Encryption: Standard V2.3
- Key Length: 128-bit
- Password Recovery: Successful on all three reports

The assessment used PDF hash extraction, dictionary-based password auditing and offline verification.

Tools included:
- Networkwalks Hash Calculator
- Networkwalks Password Cracker
- pdf2john
- John the Ripper
- RockYou Wordlist
- qpdf
- ExifTool

**Results:**

| Document | Encryption | Password Recovery |
|---|---|---|
| patient_report_1.pdf | Standard V2.3 (128-bit) | Successful |
| patient_report_2.pdf | Standard V2.3 (128-bit) | Successful |
| patient_report_3.pdf | Standard V2.3 (128-bit) | Successful |

**Impact:** Dictionary-based password recovery demonstrated that the passwords did not provide adequate protection against offline password guessing.

### Phase 5 — PDF Metadata Analysis

**Objective:** Examine document metadata and correlate internal identifiers with the exposed database records.

#### F-04: Sensitive PDF Metadata Disclosure

Metadata was extracted from all three decrypted pathology reports.

**Observed metadata:**

| Source | Finding |
|---|---|
| Report 1 | Author: Mediroza Diagnostics Lab |
| Report 2 | Author: Mediroza Diagnostics Lab |
| Report 3 | Author: j.malik |
| All reports | Creator: Mediroza CMS 1.4.2 |
| All reports | Producer: Mediroza Lab Reporting Module |
| Report 3 | Comment referenced the legacy `/old/` backup directory |

**Database correlation:**

The identifier `j.malik` matched Jameel Malik, identified as an IT Systems Administrator in the exposed staff table.

The metadata comment also referenced the legacy backup directory discovered during reconnaissance.

**Impact:** Document metadata disclosed internal identifiers, software information and legacy infrastructure references.

## 6. Evidence and Screenshots

The following evidence categories document the assessment activities.

| Figure | Evidence Description |
|---|---|
| 01 | WHOIS, WhatWeb and robots.txt discovery |
| 02 | Legacy directory listing and SQL backup retrieval |
| 03 | Staff table structure and records |
| 04 | Shareholder table structure and records |
| 05 | Username enumeration |
| 06 | SQL syntax error during login testing |
| 07 | Patient portal access |
| 08 | PDF hash extraction |
| 09 | Report 1 password recovery |
| 10 | Report 2 password recovery |
| 11 | Report 3 password recovery |
| 12–14 | Opened pathology report evidence |
| 15 | PDF metadata and matching staff record |

**Evidence directory:** `Screenshots/`

Evidence should be organized by assessment phase and figure number.

## 7. Security Findings and Remediation

| Finding | Recommended Remediation |
|---|---|
| F-01 — SQL Injection | Implement parameterized queries, separate authentication checks and suppress database error details. |
| F-02 — Database Backup Exposure | Remove SQL backups from public directories and restrict access to backup storage. |
| F-03 — Weak PDF Passwords | Enforce strong, unique passwords and authenticated, user-specific document access. |
| F-04 — PDF Metadata Disclosure | Remove internal identifiers, comments and unnecessary metadata before distribution. |
| F-05 — Username Enumeration | Use consistent authentication error messages, rate limiting and monitoring. |
| F-06 — Directory Listing | Disable directory indexing and restrict access to legacy directories. |

### Remediation Priority

1. **Immediate:** Fix SQL injection and patient portal authentication bypass.
2. **Immediate:** Remove publicly accessible database backups.
3. **High:** Improve PDF password protection and document access controls.
4. **Medium:** Sanitize PDF metadata.
5. **Low:** Prevent username enumeration and disable directory listing.

## 8. Conclusion

The assessment demonstrated how vulnerabilities in authentication, file exposure and document security can combine to create significant risks for a web application.

The most critical issue was SQL injection in the patient portal, which resulted in authentication bypass. The publicly accessible SQL backup also exposed sensitive staff and shareholder information.

The assessment highlighted the importance of secure database queries, proper authentication, restricted backup storage, strong document protection and metadata sanitization.

This project provided practical experience in external black-box penetration testing, vulnerability identification, evidence collection, password security assessment and professional security reporting.

## 9. Learning Outcomes

Through this assessment, the following practical skills were developed:

- Domain reconnaissance and information gathering.
- Web technology fingerprinting.
- HTTP response and header analysis.
- Directory and file enumeration.
- SQL injection identification.
- Authentication and authorization testing.
- Database backup exposure assessment.
- PDF encryption and password auditing.
- PDF metadata extraction and analysis.
- Evidence documentation and vulnerability reporting.
- Security remediation recommendations.

## 10. Project Information

**Project:** Mediroza General Hospital — Web Application Penetration Testing

**Program:** Networkwalks Cybersecurity Internship

**Batch:** B084

**Week:** 4

**Assessment:** External Black-Box Web Application Penetration Test

**Penetration Tester:** Uwes Kulabkar

---

## Disclaimer

This repository is intended for educational and authorized cybersecurity assessment documentation.

All testing activities described in this report were conducted as part of the assigned internship assessment. The findings are documented for vulnerability awareness, security analysis and remediation purposes.

No unauthorized testing or access is intended or encouraged.

**© 2026 — Networkwalks Cybersecurity Internship, Batch B084**















































































































































































































































































































































































































































































































































































































































