# Methodology and test log

This file records only activities supported by the supplied project notes and screenshots.

| Phase | Activity | Result |
|---|---|---|
| Reconnaissance | WHOIS lookup | Namecheap registration metadata collected |
| Reconnaissance | DNS enumeration | A, NS, MX, TXT and SRV records observed |
| Fingerprinting | WhatWeb | HTTP 200; OpenResty and Cloudflare challenge indication observed |
| Enumeration | Public website and source review | `Mediroza CMS 1.4.2` string observed |
| Enumeration | `robots.txt` and sitemap review | `/patient/`, `/staff/`, `/old/` listed as disallowed; four public sitemap URLs |
| Application review | Patient and Staff login pages | POST-based username/password forms observed |
| Authentication testing | SQL injection test on Patient Portal | MySQL syntax error observed; tester reports successful bypass |
| Post-authentication review | Patient Portal | Three encrypted PDF report entries listed |
| Data exposure review | Legacy SQL backup | Staff salary and shareholder tables observed |

## Constraints
- The target was described by the tester as a purpose-built training simulation.
- Testing was limited to the named target domain.
- No social engineering or denial-of-service activity is documented.
- No additional tests are claimed unless evidenced in the report.

## Notes for final evidence capture
For a professional submission, retain screenshots showing:
1. The SQL error after the crafted login input.
2. The portal report listing after access.
3. The backup URL and SQL dump header.
4. The staff table structure and salary rows.
5. The shareholder table structure and rows.
6. Any separate evidence of PDF decryption, if performed.
