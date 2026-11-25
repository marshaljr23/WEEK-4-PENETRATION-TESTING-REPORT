<div align="center">

# 🔐 Penetration Testing Report-Mediroza General Hospital

</div>

<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ver-Virtualbox%20v7.2-0070C0?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Kali%20Linux-v2026.2-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Linux-404040?style=flat-square&labelColor=C00000" />
    <img src="https://img.shields.io/badge/Penetration%20Testing-C00000?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Virtualization-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/Kali%20Linux-404040?style=flat-square&labelColor=C00000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Waqas%20Karim%20CCIE-C00000?style=flat-square" />
</p>

| Field | Details |
| :--- | :--- |
| **Client** | Mediroza General Hospital |
| **Engagement** | Black-box Penetration Test |
| **Duration** | 5 Days |
| **Target** | `https://medirozahospital.com` |
| **Classification** | Confidential - Authorized Personnel Only |
| **Date** | 25th November 2026 |
| **Pen-tester (Cybersecurity Intern)** | Olima Emmanuel Oko |
| **Batch** | B084 NETWORKWALKS |
| **LinkedIn** | [Olima Emmanuel Oko](https://www.linkedin.com/in/emmanuel-oko-234244199/) |
| **GitHub** | [marshaljr23](https://github.com/marshaljr23) |
| **Instructor** | Waqas Karim CCIE \| NETWORKWALKS |

---

## 01 Executive Summary

This project is aimed at performing a full black-box penetration test and vulnerability assessment of Mediroza General Hospital, the hospital's public web infrastructure at `https://medirozahospital.com`. The engagement lasted 5 days and was conducted under written authorization, within a strictly defined scope.

The assessment identified four security weaknesses, including a Critical SQL injection flaw that allowed complete unauthenticated bypass of the patient portal, and a publicly exposed database backup containing the salaries of all hospital employees and the full shareholder register. In total, the tester retrieved three confidential patient laboratory reports, decrypted all three encrypted PDF files, and recovered confidential HR and ownership data that should never have been internet-accessible.

| ID | Vulnerability | Severity | Status |
| :--- | :--- | :--- | :--- |
| **F1** | SQL Injection in patient portal authentication | Critical | Exploited |
| **F2** | Unauthenticated access to patient report repository | High | Exploited |
| **F3** | Weak / trivially crackable PDF encryption | Medium | Exploited |
| **F4** | Publicly exposed database backup (staff PII + salaries + shareholders) | Critical | Exploited |

> **Key Findings at a Glance:** Although tracked as a single finding, the exposed database backup (F4) is arguably the highest-impact issue of the engagement because it discloses the complete payroll and the hospital's beneficial ownership to any anonymous internet user.

---

## 02 Scope and Methodology

### 2.1 Scope

Testing was limited to the single in-scope target domain `medirozahospital.com`. No social engineering, no denial-of-service testing, and no testing outside the agreed scope was performed. All testing was non-destructive; no data was modified or deleted on the target.

| Category | Details |
| :--- | :--- |
| **In scope** | `https://medirozahospital.com` and all sub-paths |
| **Out of scope** | Social engineering, DoS/DDoS, third-party services, physical security |
| **Authorization** | Written permission granted by the client (Networkwalks engagement) |
| **Rules of engagement** | Target domain only; no DoS; no out-of-scope testing |

### 2.2 Methodology

The engagement followed a standard five-phase penetration testing lifecycle:

| Phase | Activities Performed |
| :--- | :--- |
| **1. Reconnaissance** | Passive/active foot printing; `sitemap.xml` and `robots.txt` review; directory and file brute-forcing (`dirsearch`, 408-entry wordlist); service and technology fingerprinting. |
| **2. Vulnerability Analysis** | Manual and automated probing of authentication mechanisms, input handling, session management, access controls, and file encryption. |
| **3. Exploitation** | SQL injection to bypass patient portal login; unauthenticated retrieval of patient reports; password cracking of encrypted PDFs (John the Ripper); discovery and download of an exposed database backup. |
| **4. Post-exploitation** | Decryption of PDF metadata (RC4 / PDF Algorithm 2-3), extraction of hidden comments, analysis of the leaked SQL dump for confidential HR and shareholder records. |
| **5. Reporting** | Consolidation of findings, risk rating, evidence capture, and remediation advice. |

### 2.3 Tools Used

* **Recon:** `dirsearch` (directory brute-forcing), `curl`, web browser
* **Vulnerability testing:** Manual HTTP request manipulation, SQL injection payloads
* **Password cracking:** Networkwalks Hash calculator, Hashcracker, John the Ripper 1.9.0-Jumbo (`pdf2john.pl` hash extraction, wordlist + Single rules)
* **PDF analysis:** `pikepdf` (decryption), `PyMuPDF` (text/metadata extraction), custom RC4 / PDF Algorithm 2-3 implementation
* **Data analysis:** Python 3, SQL parsing
* **Website Lookup:** `Whatweb`, `whois`, `curl`, `nslookup`, and `wafw00f`
* **Environment:** Kali Linux

### 2.4 Limitations

The assessment was time-boxed to five days and performed from a single external vantage point. No internal network, wireless, or physical testing was performed. The third encrypted PDF required a dictionary/rule-based attack; a longer brute-force campaign would have been used given more time. Findings represent the security posture at the time of testing only.

---

## 03 Findings and Proof of Exploitation

Each finding below includes its technical detail, the evidence gathered during testing, and the resulting business impact. All findings were confirmed by direct exploitation against the live target.

### F1: SQL Injection in Patient Portal Authentication

* **Location:** `https://medirozahospital.com/patient/login.php`
* **Description:** The patient portal login form concatenates user-supplied input directly into a SQL query. An attacker can inject SQL into the username (or password) field to alter the query logic and bypass authentication entirely, gaining access to the portal without valid credentials. The application accepts the injected input and grants an authenticated session.
* **Evidence / Proof of Exploitation:** A crafted login request with the username `admin'--` caused the application to skip password verification and issue a valid session. The payload neutralizes the password check in the underlying query.
* **Business Impact:** Complete unauthenticated access to the patient portal and all functionality behind it, including the ability to enumerate and download patient laboratory reports. In the worst case, the same flaw permits extraction of the underlying database (patient records, staff accounts) via UNION- or blind-based techniques.


Injection payload used:
  username: admin'--
  password: [Any random password]

Result:
  HTTP 302 redirect to the authenticated portal; authentication bypassed.
## Findings & Proof of Exploitation (Continued)

| File | Password | Method |
| :--- | :--- | :--- |
| `LR-2024-1187_S_Dlamini.pdf` | `123456` | Dictionary / common password |
| `LR-2024-1192_P_Reddy.pdf` | `password` | Dictionary / common password |
| `LR-2024-1205_E_Thompson.pdf` | `!@#$%^&` | John the Ripper - wordlist + Single rules |

> *Note: All PDFs' hashes and passwords were generated and revealed using the Networkwalks PDF Hashcalculator and Hashcracker, further verified using John the Ripper.*

---

### F2: Unauthenticated Access to Patient Report Repository

* **Location:** `https://medirozahospital.com/patient/reports/` \| `/patient/download.php`
* **Description:** The patient report repository and the report download script do not enforce authentication. The directory `/patient/reports/` is browsable and `/patient/download.php` serves report files by parameter without verifying that the requester holds a valid session. Any anonymous visitor can list and retrieve confidential laboratory reports.
* **Evidence / Proof of Exploitation:** Without any login or session cookie, I retrieved the directory index at `/patient/reports/` (HTTP 200) and downloaded the three confidential reports. The report files are also directly enumerable by their predictable names.
* **Business Impact:** Mass disclosure of protected health information (PHI). Patient names, identifiers, dates of birth, and clinical results are exposed to the public internet - a direct breach of patient confidentiality and of data-protection obligations.

---

### F3: Weak PDF Encryption

* **Location:** `LR-2024-1187`, `LR-2024-1192`, `LR-2024-1205` (patient lab reports)
* **Description:** The three retrieved laboratory reports are protected with standard-security (RC4 / 128-bit) PDF encryption using short, human-guessable passwords. All three were recovered using a small wordlist and rule set, demonstrating that the encryption provides no meaningful protection against a determined attacker.
* **Evidence / Proof of Exploitation:** Hashes were extracted with Networkwalks Hash Calculator, `pdf2john.pl`, and cracked with Networkwalks Hashcracker and John the Ripper:

| File | Password | Method |
| :--- | :--- | :--- |
| `LR-2024-1187_S_Dlamini.pdf` | `123456` | Dictionary / common password |
| `LR-2024-1192_P_Reddy.pdf` | `password` | Dictionary / common password |
| `LR-2024-1205_E_Thompson.pdf` | `!@#$%^&` | John the Ripper - wordlist + Single rules |

* **Business Impact:** Password-based PDF protection gives a false sense of security. Because the passwords are weak, any party that obtains a report file (via F2, email interception, or loss of a device) can read its contents almost immediately.

---

### F4: Publicly Exposed Database Backup (Staff PII, Salaries & Shareholders)

* **Location:** `https://medirozahospital.com/old/mediroza_db_backup_2019.sql`
* **Description:** An internal MySQL database backup from 2019 was left publicly accessible in a browsable directory on the web server. The directory listing at `/old/` exposes the file, which can be downloaded by anyone without authentication. The file's own header warns that it contains confidential staff and shareholder records. The exposure was discovered by decrypting the PDF metadata of the third report, whose hidden `/Comments` field read: `'DB backup moved to /old before site migration, do not delete'`.
* **Evidence / Proof of Exploitation:** The backup was downloaded directly from the live server:

* **Business Impact:** Full disclosure of the hospital's payroll and ownership. The file contains the names, job titles, departments, email addresses, phone numbers, national ID numbers, and monthly salaries of all 30 employees, plus the names, share percentages, share counts, and share classes of all 10 shareholders. This is a severe breach of employee privacy and commercially sensitive information.

| Item | Details |
| :--- | :--- |
| **Retrieval command** | `curl -s -L https://medirozahospital.com/old/mediroza_db_backup_2019.sql` |
| **Result** | HTTP 200, 6,346 bytes. Directory index at `/old/` confirmed the file listing. |

---

### 3.1 Confidential Data Uncovered

The leaked backup (database `mediroza_hr`) contained two tables. A summary of the exposed records is shown below:

| Table | Records | Fields Exposed |
| :--- | :--- | :--- |
| **Staff** | 30 | `full_name`, `job_title`, `department`, `email`, `phone`, `national_id`, `monthly_salary_zar`, `date_joined` |
| **Shareholders** | 10 | `shareholder_name`, `share_percent`, `shares_held`, `share_class` |

**Highest-paid employees (monthly salary, ZAR):**

| Name | Job Title | Department | Monthly Salary |
| :--- | :--- | :--- | :--- |
| Dr. Johan van der Merwe | Medical Director | Management | R 160,000 |
| Sarah Botha | Chief Financial Officer | Finance | R 152,000 |
| Dr. Rajesh Naidoo | Chief Pathologist | Diagnostics Lab | R 138,000 |
| Dr. Vikram Chetty | Anaesthetist | Theatre | R 135,000 |
| Dr. Anita Naicker | Consultant Cardiologist | Cardiology | R 132,000 |

---

## 04 Risk Rating

Each vulnerability is rated using the CVSS v3.1 base score and a qualitative severity band. Severity reflects both the technical exploitability and the business impact to Mediroza General Hospital.

| ID | Vulnerability | CVSS | Severity |
| :--- | :--- | :--- | :--- |
| **F1** | SQL Injection - patient portal auth | 9.8 | Critical |
| **F4** | Exposed database backup (PII / salaries / shareholders) | 9.1 | Critical |
| **F2** | Unauthenticated patient report access | 8.1 | High |
| **F3** | Weak PDF encryption | 5.3 | Medium |

### Risk Matrix Summary

| Severity | Count | Findings |
| :--- | :--- | :--- |
| **Critical** | 2 | F1 (SQL Injection), F4 (Exposed DB backup) |
| **High** | 1 | F2 (Unauthenticated report access) |
| **Medium** | 1 | F3 (Weak PDF encryption) |
## 05 Recommendations and Remediation

The following prioritised actions are recommended. Items are ordered by risk: address the Critical findings before all others.

### 5.1 Critical - Immediate Action

1. Remove or restrict the `/old/` directory immediately. Delete the exposed SQL backup from the web root, and audit the server for any other stray backup, export, or configuration files. Verify no sensitive files remain internet-accessible. *(Addresses: F4)*
2. Eliminate SQL injection by using parameterized queries / prepared statements (or an ORM) for every database call. Never concatenate user input into SQL. Apply the principle of least privilege to the database account. *(Addresses: F1)*
3. Deploy a web application firewall (WAF) and/or input validation as defense-in-depth while the code fix is completed. *(Addresses: F1)*

### 5.2 High - Short Term

1. Enforce authentication and authorization on every patient report page and on `/patient/download.php`. Verify the session and the patient's ownership of the requested report server-side. Disable directory listing across the site. *(Addresses: F2)*
2. Serve reports through an authenticated controller that streams files from a location outside the web root, rather than direct file links. *(Addresses: F2)*

### 5.3 Medium - Planned

1. Replace password-based PDF protection with strong, unique passphrases (16 characters or more, random) or, preferably, a secure distribution mechanism (patient portal download after authentication, encrypted email, or digital signatures). Do not rely on weak shared passwords. *(Addresses: F3)*
2. Add PDF metadata hygiene to the document workflow: strip or sanitize hidden metadata/comments before reports are published, so internal notes cannot leak. *(Addresses: F3 / F4)*

### 5.4 General Security Improvements

1. Implement secure session management (`Secure`, `HttpOnly` cookies; session expiry; regeneration on privilege change).
2. Enforce HTTPS everywhere with HSTS and redirect all HTTP traffic.
3. Add security logging and monitoring/alerting for authentication failures, SQL errors, and access to sensitive paths.
4. Conduct regular vulnerability assessments and code reviews, and provide secure development training to staff.
5. Establish a data-retention and secure-backup policy so that old database exports are never published to the web root.
## 06 Evidence Collected

<img width="1920" height="1032" alt="Nmap scan" src="https://github.com/user-attachments/assets/1100f353-cca7-43eb-be5a-7556e524064b" />

<img width="1920" height="1032" alt="Portal Access" src="https://github.com/user-attachments/assets/4bb899bb-9dc7-425a-8013-954836bfa4c8" />

<img width="1920" height="1032" alt="Portal Login-SQLi" src="https://github.com/user-attachments/assets/3af39f02-f8ec-418d-adbd-9ff74cdc77c6" />

<img width="1920" height="1032" alt="PDF 2 Cracked" src="https://github.com/user-attachments/assets/fb349ab1-661f-452a-9809-3a3d22e0bdd9" />
<img width="1920" height="1032" alt="PDF 3 Cracked" src="https://github.com/user-attachments/assets/98af7ed9-b89f-4523-b4c0-921d10033077" />
<img width="1920" height="1032" alt="PDF 1 Unlocked" src="https://github.com/user-attachments/assets/205d9444-e261-42c5-b6e8-1e3143d0f158" />
<img width="1920" height="1032" alt="PDF 2 Unlocked" src="https://github.com/user-attachments/assets/1a7c13b6-0044-43d6-98f1-5b34403f45d9" />
<img width="1920" height="1032" alt="PDF 3 Unlocked" src="https://github.com/user-attachments/assets/da809550-b37c-4661-a9c2-228d4dfccb90" />

---

## 07 Conclusion

Mediroza General Hospital's web application is currently exposed to trivially exploitable flaws that have already disclosed confidential patient reports, the complete staff payroll, and the shareholder register. The Critical findings (F1 and F4) must be remediated immediately. With the recommended controls in place, the hospital can materially reduce its risk and meet its obligations to patients, staff, and regulators.

> **Disclaimer:** This report is confidential and intended only for internship practice, the named client, and authorized personnel. The testing described was conducted in a controlled environment for educational purposes under written authorization; these techniques must never be applied to any system without the owner's explicit written permission.
