# Penetration Testing Report — Mediroza General Hospital

## 1. Engagement overview

| Item | Detail |
|---|---|
| Assessment type | Authorised black-box web application penetration test |
| Target | `https://medirozahospital.com` |
| Target context | Purpose-built training simulation, as stated by the tester |
| Duration | Five-day assignment (exact dates not captured) |
| Testing environment | Kali Linux in VirtualBox; Microsoft Edge |
| Primary objectives | Identify exposed entry points; assess authentication; locate three encrypted laboratory-report PDFs; assess exposed data; document findings |
| Out-of-scope constraints from assignment | No social engineering; no denial-of-service; no testing outside the specified domain |

## 2. Executive summary

The assessment identified a SQL injection condition in the Patient Portal authentication workflow. The supplied screenshot shows a MySQL syntax error displayed by the application after a crafted username value was submitted. The tester reports that SQL injection was then used to bypass the login failure; a later screenshot shows the authenticated “My lab reports” page. This sequence is consistent with an authentication-bypass vulnerability caused by unsafe handling of user input.

The portal displayed three password-protected PDF report entries. A separate legacy SQL backup was accessible at `/old/mediroza_db_backup_2019.sql`. The dump header identifies the database as `mediroza_db`, describes it as an internal database backup, and warns that it contains confidential staff and shareholder records. The screenshots show staff salary fields and shareholder ownership data.

The exposure of staff salary and ownership information creates a significant confidentiality risk. The backup also appears to contain direct identifiers and contact information; those fields are intentionally omitted from this report’s exported tables.

## 3. Scope and methodology

Testing was performed against the specified simulated domain and its publicly reachable application paths. The evidence supplied documents:
- WHOIS and DNS reconnaissance.
- Web technology identification using WhatWeb.
- Inspection of public pages, page source, `robots.txt`, and `sitemap.xml`.
- Inspection of Patient Portal and Staff Login forms.
- Review of the POST-based login workflow and its response.
- SQL injection testing against the Patient Portal login form.
- Confirmation of portal access through the subsequent “My lab reports” page.
- Discovery and review of the legacy SQL database backup.
- Extraction and tabulation of staff salary and shareholder records from the visible SQL dump.

No claim is made here that unshown tests were performed.

## 4. Reconnaissance and application mapping

| Check | Observation | Interpretation |
|---|---|---|
| WHOIS | Registrar Namecheap; created 2026-08-14; updated 2026-08-17; expiry 2027-08-14 | Domain registration metadata collected |
| DNS A record | `199.188.201.16` | Resolved IPv4 address |
| Nameservers | `dns1.namecheaphosting.com`, `dns2.namecheaphosting.com` | Hosting DNS infrastructure |
| MX records | `mx1-hosting.jellyfish.systems`, `mx2-hosting.jellyfish.systems`, `mx3-hosting.jellyfish.systems` | Mail-routing records |
| DMARC TXT | `v=DMARC1; p=none` | Monitoring policy; does not request quarantine/rejection |
| WhatWeb | HTTP 200; OpenResty 1.31.1.1; Cloudflare challenge indication | Technology fingerprint from the observed response |
| Login response headers | LiteSpeed; PHP/8.2.36 | Separate login response identified these headers; do not assume they describe every endpoint |
| Homepage source | `Mediroza CMS 1.4.2` | CMS/version string exposed in source |
| Sitemap | `/index.html`, `/about.html`, `/doctors.html`, `/contact.html` | Four public URLs listed |
| robots.txt | Disallows `/patient/`, `/staff/`, `/old/` | Reveals paths to crawlers; not an access-control mechanism |

### Discovered application paths

| Path | Purpose / observation |
|---|---|
| `/patient/login.php` | Patient Portal login form |
| `/staff/login.php` | Staff login form |
| `/old/` | Listed as disallowed in `robots.txt`; legacy backup path was found here |
| `/old/mediroza_db_backup_2019.sql` | SQL backup file shown in browser |

The login form used `POST` to `login.php` with `username` and `password` fields. The request was observed as form-encoded. The failed-login response displayed “Incorrect password” with HTTP 200. A screenshot also showed the PHP session cookie name `PHPSESSID`; the cookie name alone does not establish whether session handling is secure.

## 5. SQL injection and authentication bypass

### Evidence observed

The supplied screenshot of `/patient/login.php` displays an application error referring to `mysqli_query()` and a MySQL syntax error near an empty string at line 1 after a crafted username value was submitted. This is direct evidence that database error details were exposed to the client and that the input reached a database query without safe handling.

The tester reports that SQL injection was used to bypass the login failure. The subsequent screenshot shows the “My lab reports” page and a logout link, which is evidence of access to the authenticated portal view.

### Impact

- Authentication controls could be bypassed.
- Protected portal content became accessible without the expected valid credentials.
- The error message disclosed database implementation details.
- If the same unsafe query pattern exists elsewhere, additional database records may be exposed or modified.

### Severity

**Critical** — authentication bypass through SQL injection affecting a portal that contains protected records.

## 6. Protected laboratory report listing

The authenticated portal screenshot shows three report entries. The visible labels and metadata are:

| # | Report label | Lab reference | Date shown | File status |
|---:|---|---|---|---|
| 1 | Pathology Report — S. Dlamini | LR-2024-1187 | 2024-11-04 | PDF (encrypted) |
| 2 | Pathology Report — P. Reddy | LR-2024-1192 | 2024-11-05 | PDF (encrypted) |
| 3 | Pathology Report — E. Thompson | LR-2024-1205 | 2024-11-06 | PDF (encrypted) |

The screenshot confirms that the portal listed three encrypted PDF files. It does not independently prove that the PDF contents were decrypted or reviewed; record decryption outcomes separately if supported by additional evidence.

## 7. Legacy database backup exposure

The backup was shown at:

`https://medirozahospital.com/old/mediroza_db_backup_2019.sql`

The visible header identifies:
- Database: `mediroza_db`
- Backup description: internal database backup
- Generator: Mediroza CMS 1.4.2 backup module
- Host: localhost
- Backup date: 2019-08-27 02:14:03
- Warning: contains confidential staff and shareholder records

The dump includes a `staff` table with fields for employee name, job title, department, email, phone, national ID, monthly salary in ZAR, and date joined. It also includes a `shareholders` table with shareholder name, share percentage, shares held, and share class.

### 7.1 Employee salary records

The following table contains the fields needed for the assignment. National IDs, phone numbers, and email addresses are deliberately excluded from this report.

| ID | Employee | Job title | Department | Monthly salary (ZAR) | Date joined |
|---:|---|---|---|---:|---|
| 1 | Dr. Rajesh Naidoo | Chief Pathologist | Diagnostics Lab | 138,000 | 2009-03-16 |
| 2 | Sarah Botha | Chief Financial Officer | Finance | 152,000 | 2011-07-01 |
| 3 | Dr. Johan van der Merwe | Medical Director | Management | 160,000 | 2012-09-10 |
| 4 | Dr. Anita Naicker | Consultant Cardiologist | Cardiology | 128,000 | 2013-02-18 |
| 5 | Dr. Ahmed Kara | Consultant Physician | Internal Medicine | 135,000 | 2011-06-13 |
| 6 | Dr. Yusuf Cassim | Senior Registrar | Emergency & Trauma | 96,000 | 2010-11-15 |
| 7 | Michael Roberts | HR Director | Human Resources | 132,000 | 2010-11-15 |
| 8 | Susan Pretorius | HR Officer | Human Resources | 32,000 | 2016-08-23 |
| 9 | Jameel Malik | IT Systems Administrator | IT | 58,000 | 2015-04-12 |
| 10 | Thabo Molefe | Network Engineer | IT | 46,000 | 2017-10-02 |
| 11 | Nomvula Khumalo | Registered Nurse | Emergency & Trauma | 34,000 | 2016-01-19 |
| 12 | Lerato Mokoena | Registered Nurse | Pediatrics | 33,000 | 2017-06-07 |
| 13 | Bongani Ndlovu | Registered Nurse | Cardiology | 35,000 | 2015-12-01 |
| 14 | Zanele Mahlangu | Nursing Sister | Theatre | 42,000 | 2013-03-25 |
| 15 | Kagiso Sithole | Pharmacist | Pharmacy | 61,000 | 2014-09-08 |
| 16 | Naledi Zulu | Pharmacy Assistant | Pharmacy | 26,000 | 2019-02-14 |
| 17 | Themba Nkosi | Radiographer | Radiology | 44,000 | 2016-07-30 |
| 18 | Palesa Radebe | Radiographer | Radiology | 43,000 | 2017-04-11 |
| 19 | Deepak Pillay | Lab Technologist | Diagnostics Lab | 41,000 | 2015-05-20 |
| 20 | Kavitha Govender | Lab Technician | Diagnostics Lab | 35,000 | 2018-08-06 |
| 21 | Dr. Suresh Moodley | Consultant Radiologist | Radiology | 130,000 | 2012-02-28 |
| 22 | Dr. Fatima Patel | Pediatrician | Pediatrics | 118,000 | 2013-10-17 |
| 23 | Nisha Singh | Physiotherapist | Rehabilitation | 48,000 | 2016-11-09 |
| 24 | Dr. Vikram Chetty | Anaesthetist | Theatre | 135,000 | 2011-06-13 |
| 25 | David Smith | Facilities Manager | Operations | 52,000 | 2014-01-27 |
| 26 | Karen O'Connor | Billing Administrator | Finance | 29,000 | 2018-03-19 |
| 27 | James Wilson | Security Supervisor | Operations | 27,600 | 2019-09-02 |
| 28 | Linda Fourie | Receptionist | Front Office | 19,000 | 2020-02-10 |
| 29 | Peter van Wyk | Procurement Officer | Supply Chain | 38,000 | 2015-08-24 |
| 30 | Andile Mbeki | Ward Clerk | Administration | 21,000 | 2019-11-05 |

**Data quality note:** The salary/date entries were transcribed from angled screenshots of the SQL dump. Rows marked medium confidence in `data/staff_salary_records.csv` contain one or more characters that were less clear in the screenshot and should be checked against the original `.sql` file before formal submission.

### 7.2 Shareholder records

| ID | Shareholder | Shareholding (%) | Shares held | Share class |
|---:|---|---:|---:|---|
| 1 | Dr. Rajesh Naidoo | 18.0% | 180,000 | Ordinary |
| 2 | Cedar Health Holdings (Pty) Ltd | 15.0% | 150,000 | Ordinary |
| 3 | Dr. Johan van der Merwe | 12.0% | 120,000 | Ordinary |
| 4 | Reddy Family Trust | 11.0% | 110,000 | Ordinary |
| 5 | Thabo Molefe | 10.0% | 100,000 | Ordinary |
| 6 | Sarah Botha | 9.0% | 90,000 | Ordinary |
| 7 | Dr. Ahmed Kara | 8.0% | 80,000 | Preferential |
| 8 | Naledi Zulu | 7.0% | 70,000 | Ordinary |
| 9 | Michael Roberts | 6.0% | 60,000 | Ordinary |
| 10 | Dr. Vikram Chetty | 4.0% | 40,000 | Preferential |

**Interpretation:** The visible records total 100% ownership and 1,000,000 shares. The data indicates that the backup contains sensitive internal ownership information.

## 8. Findings register

| ID | Finding | Severity | Evidence | Business impact |
|---|---|---|---|---|
| F-01 | SQL injection in Patient Portal login | Critical | MySQL syntax error shown after crafted input; tester reports successful SQLi bypass; portal page subsequently displayed | Unauthorised access to protected portal functionality and records |
| F-02 | Publicly accessible legacy database backup | Critical | `/old/mediroza_db_backup_2019.sql` opened in browser; dump header and table contents visible | Disclosure of internal database structure, employee salary data, and shareholder information |
| F-03 | Verbose database error disclosure | High | Client-facing MySQL syntax error includes `mysqli_query()` / SQL syntax detail | Helps an attacker identify backend technology and query failure details |
| F-04 | Sensitive records available in portal | High | Authenticated portal lists three encrypted PDF report entries | Potential exposure of sensitive laboratory-report metadata and files |
| F-05 | `robots.txt` reveals restricted path names | Low | Disallow entries for `/patient/`, `/staff/`, `/old/` | Reveals application paths; does not itself grant access |

## 9. Recommendations

### Immediate
1. Remove the SQL backup from the public web directory and verify that no other backups, dumps, archives, or configuration files are publicly reachable.
2. Patch the login query using parameterised statements / prepared statements. Do not concatenate user-controlled input into SQL.
3. Replace verbose database errors with generic client-facing authentication errors; retain detailed diagnostics only in protected server logs.
4. Invalidate exposed sessions and rotate any credentials or secrets present in the backup.
5. Review access logs for requests to the backup and authentication endpoint.

### Short term
1. Enforce authentication and authorisation checks on every protected page and file download, server-side.
2. Ensure successful authentication regenerates the session ID.
3. Configure session cookies with `Secure`, `HttpOnly`, and an appropriate `SameSite` attribute.
4. Store backups outside the web root, encrypt them at rest, and restrict access.
5. Apply least-privilege database permissions.
6. Add automated security tests for SQL injection and authentication bypass.

### Long term
1. Adopt a secure software development lifecycle and code review for database access.
2. Conduct periodic web application security assessments.
3. Monitor for public exposure of backup extensions and sensitive file types.
4. Establish a data-retention and secure-backup policy.

## 10. Conclusion

The supplied evidence supports a critical SQL injection issue in the Patient Portal authentication flow and critical exposure of a legacy SQL database backup. Together, these weaknesses create a pathway to unauthorised access and disclosure of sensitive business records. Remediation should prioritise parameterised database access, removal of public backups, and verification of server-side authorisation.

## 11. Evidence and limitations

The report is based on screenshots and observations supplied in this conversation. The evidence index is in `evidence/EVIDENCE_INDEX.md`. The raw backup file itself was not provided as a downloadable attachment; the table values were transcribed from photographs of the dump. Any uncertain transcription should be validated against the original SQL file. No decrypted contents of the three PDFs are included because the supplied portal screenshot only establishes that the files were listed as encrypted.
