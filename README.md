# Mediroza General Hospital — Authorised Penetration Test

**Project type:** Black-box web application penetration test (training simulation)  
**Target:** `https://medirozahospital.com`  
**Assessment window:** Five-day assignment (exact start/end dates not confirmed in the supplied evidence)  
**Environment:** Kali Linux / VirtualBox and Microsoft Edge  
**Status:** Draft report assembled from the screenshots and observations supplied for this project.

> **Training-data notice:** This repository contains information transcribed from the supplied simulated-lab database-backup screenshots. The records are treated as exercise data. Employee national-ID values, personal phone numbers, and email addresses have intentionally been excluded from the report and CSV exports because they are not needed to demonstrate the security impact.

## Repository structure

```text
mediroza-pentest-repo/
├── README.md
├── REPORT.md
├── METHODOLOGY.md
├── FINDINGS.md
├── data/
│   ├── staff_salary_records.csv
│   └── shareholder_records.csv
└── evidence/
    └── EVIDENCE_INDEX.md
```

## Executive summary

The supplied evidence indicates that the Patient Portal login accepted a crafted input that triggered a MySQL syntax error and, in the subsequent test, allowed access to the portal without the expected valid credentials. The portal displayed a “My lab reports” page containing three password-protected PDF report entries. A legacy database backup was then accessible under `/old/`; its contents identified the database as `mediroza_db`, showed a 2019 backup timestamp, and contained staff salary and shareholder tables.

The principal issue is **SQL injection in the authentication flow**, with resulting authentication bypass and exposure of sensitive business records. The legacy backup exposure is a separate high-impact information-disclosure issue. The screenshots show the database dump and the portal report listing; they do not, by themselves, establish that all three PDF files were successfully decrypted.

## Key results

| Area | Evidence-based result |
|---|---|
| Domain registration | WHOIS screenshot notes Namecheap, registration 2026-08-14, update 2026-08-17, expiry 2027-08-14 |
| DNS | A record `199.188.201.16`; Namecheap nameservers; MX hosts under `jellyfish.systems`; DMARC policy `p=none` |
| Web technology | WhatWeb observation: HTTP 200, OpenResty 1.31.1.1, Cloudflare challenge indication; login response headers separately showed LiteSpeed and PHP 8.2.36 |
| CMS clue | Homepage source displayed `Mediroza CMS 1.4.2` |
| Public sitemap | `/index.html`, `/about.html`, `/doctors.html`, `/contact.html` |
| robots.txt | Disallowed `/patient/`, `/staff/`, `/old/`; this is a crawler instruction, not access control |
| Login endpoints | `/patient/login.php` and `/staff/login.php`; form submits by POST to `login.php` |
| SQL injection | Crafted username input produced a visible MySQL syntax error; subsequent portal screenshot shows successful access |
| Exposed backup | `/old/mediroza_db_backup_2019.sql`; dump header describes internal database backup and confidential staff/shareholder records |
| Portal reports | Three encrypted PDF entries shown in the portal (see REPORT.md) |

## Data files

- `data/staff_salary_records.csv` — employee names, roles, departments, monthly salary in ZAR, and join dates.
- `data/shareholder_records.csv` — shareholder names, ownership percentages, share counts, and share class.

**Transcription caveat:** Some rows were photographed at an angle and are not equally legible. The confidence column flags rows that should be checked against the original SQL file before treating them as authoritative. A few ambiguous salary/date values were transcribed from the clearest supplied screenshot; do not silently treat them as independently verified.

## Reproduce the report

Open `REPORT.md` for the full narrative and `FINDINGS.md` for the vulnerability register. The CSV files can be opened in Excel, LibreOffice, or imported into a spreadsheet.

## Recommended remediation

1. Replace dynamic SQL string concatenation with parameterised queries/prepared statements.
2. Enforce server-side authentication and verify credentials before creating an authenticated session.
3. Regenerate the session identifier after successful authentication; use `Secure`, `HttpOnly`, and appropriate `SameSite` cookie attributes over HTTPS.
4. Remove database backups and other sensitive artefacts from web-accessible directories; store backups outside the document root with restrictive permissions.
5. Rotate any credentials, secrets, or identifiers that may have been exposed.
6. Apply least-privilege database permissions and ensure the application account cannot access unrelated tables.
7. Add regression tests for SQL injection, authentication bypass, and direct access to protected resources.
8. Review access logs and determine whether the exposed backup or records were accessed outside the authorised exercise.

## For More

This penetration testing project was limited to the steps mentioned above. However, a full audit would include an analysis of open ports and their potential exploitation. Running services should also be evaluated. Furthermore, various cybersecurity tools—such as OpenVAS or ZAP—could be employed to conduct a more in-depth analysis of the web platform. 



____

👤 Author

Aamna Khatoon
Cybersecurity intern B084

LinkedIn: www.linkedin.com/in/aamna-khatoon-a47a54124

📌 Project Information
Program Name: Cybersecurity at Networkwalks | Week: 04 | Project: Real-world Penetration Testing - Mediroza Hospital | Repository: GitHub
