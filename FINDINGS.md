# Findings register

| ID | Title | Severity | Affected component | Status |
|---|---|---|---|---|
| F-01 | SQL injection enabling authentication bypass | Critical | `/patient/login.php` | Observed / tester-reported successful bypass |
| F-02 | Publicly accessible database backup | Critical | `/old/mediroza_db_backup_2019.sql` | Observed |
| F-03 | Verbose database error disclosure | High | Patient Portal login | Observed |
| F-04 | Sensitive report listing available after portal access | High | Patient Portal | Observed |
| F-05 | Restricted path names disclosed by robots.txt | Low | `/robots.txt` | Observed |

## F-01 — SQL injection enabling authentication bypass
**Evidence:** MySQL syntax error shown after crafted username input; tester reports using SQL injection to bypass login failure; subsequent screenshot shows the portal's authenticated report listing.  
**Impact:** Unauthorised access to portal functionality and protected records.  
**Recommendation:** Parameterised queries, server-side credential verification, generic errors, and regression tests.

## F-02 — Publicly accessible database backup
**Evidence:** Browser screenshot of `/old/mediroza_db_backup_2019.sql`; visible SQL dump header and staff/shareholders table definitions and inserts.  
**Impact:** Disclosure of employee salary and shareholder records, plus database schema and other identifiers.  
**Recommendation:** Remove backup from document root; store encrypted backups outside the web root with restrictive access controls.

## F-03 — Verbose database error disclosure
**Evidence:** Client-facing MySQL syntax error and `mysqli_query()` detail.  
**Impact:** Reveals backend implementation and query processing details.  
**Recommendation:** Suppress detailed errors from clients; log them securely server-side.

## F-04 — Sensitive report listing available after portal access
**Evidence:** Portal screenshot listing three encrypted PDF pathology reports.  
**Impact:** Exposure of report metadata and potentially report contents if file-level authorisation is also weak.  
**Recommendation:** Enforce per-user authorisation on every report request and download.

## F-05 — Path disclosure through robots.txt
**Evidence:** `Disallow` entries for `/patient/`, `/staff/`, and `/old/`.  
**Impact:** Low; crawler directives reveal route names but do not provide access control.  
**Recommendation:** Do not rely on robots.txt to protect sensitive paths; enforce access control and remove exposed artefacts.
