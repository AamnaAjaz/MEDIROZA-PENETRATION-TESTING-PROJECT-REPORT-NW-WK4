# MEDIROZA-PENETRATION-TESTING-PROJECT-REPORT-NW-WK4
An authorised black-box penetration testing project documenting SQL injection, authentication bypass, exposed database backup, and sensitive data exposure in the simulated Mediroza Hospital environment.



Penetration Testing Report

Mediroza Hospital – Web Application Security Assessment

Assessment type:     Authorised Black-Box Web Application Penetration Test
Target:              https://medirozahospital.com
Assessment period:   26–27 November 2026
Testing environment: Kali Linux, VirtualBox and Microsoft Edge
Assessment scope: Public website, patient portal, exposed directories, authentication mechanism and accessible database backup
Project: NetworkWalks Cybersecurity Internship Assignment

⸻

1. Executive Summary

An authorised penetration test was conducted against the Mediroza Hospital web application to identify security weaknesses and assess the confidentiality of information accessible through the website.

The assessment identified a critical SQL injection vulnerability in the patient portal login mechanism. By submitting a crafted SQL injection input in the username field, it was possible to bypass the normal login failure and gain access to the patient portal.

Following successful access, the portal displayed a list of three password-protected laboratory reports available for download.

The assessment also identified an exposed directory containing a database backup file:

/old/mediroza_db_backup_2019.sql

The accessible SQL backup contained database definitions and records for hospital staff and shareholders, including employee names, job titles, contact information, monthly salary information and shareholder details.

These findings demonstrate significant weaknesses in authentication security, database protection and sensitive information exposure.

2. Scope and Objectives

The assessment focused on the following objectives:

* Identify publicly accessible directories and files.
* Examine the patient portal authentication mechanism.
* Test for SQL injection vulnerabilities.
* Determine whether authentication controls could be bypassed.
* Identify confidential laboratory reports accessible through the portal.
* Examine the exposed database backup and identify the categories of information disclosed.
* Document findings and recommend appropriate security controls.

Testing was limited to the authorised assessment scope. Social engineering and denial-of-service testing were not performed.

3. Methodology and Tools

The assessment followed a black-box testing approach, examining the application from an external user’s perspective.

Tool / Technique	Purpose
WHOIS	Domain registration and ownership information
DNSRecon	DNS record enumeration
WhatWeb	Web technology identification
Microsoft Edge	Website and portal inspection
Browser Developer Tools	Examination of HTTP requests and responses
Kali Linux	Security testing environment
SQL injection testing	Assessment of login authentication
SQL backup inspection	Identification of exposed database information

4. Reconnaissance and Information Gathering

4.1 WHOIS Enumeration

The following domain information was identified:

Item	Observation
Domain	medirozahospital.com
Registrar	Namecheap, Inc.
Registration date	14 August 2026
Updated date	17 August 2026
Expiry date	14 August 2027
Domain status	Client Transfer Prohibited
DNSSEC	Unsigned
Nameservers	dns1.namecheaphosting.com; dns2.namecheaphosting.com

4.2 DNS Enumeration

DNSRecon identified the following records:

Record	Identified value
A	199.188.201.16
MX	mx1-hosting.jellyfish.systems
MX	mx2-hosting.jellyfish.systems
MX	mx3-hosting.jellyfish.systems
TXT	v=DMARC1; p=none
Additional records	SOA, NS, SRV

The DNS results also included service-discovery records associated with CalDAV, CardDAV, Autodiscover and cPanel.

4.3 Web Technology Identification

WhatWeb identified:

* HTTP response: 200 OK
* IP address: 199.188.201.16
* OpenResty 1.31.1.1 was identified in the initial response.
* A Cloudflare-related browser challenge was observed.
* The login response later identified LiteSpeed and PHP 8.2.36.

The website source also contained the CMS identification string Mediroza CMS 1.4.2.

5. Directory and File Discovery

The website’s robots.txt file contained the following entries:

Path	Observation
/patient/	Disallowed for crawlers
/staff/	Disallowed for crawlers
/old/	Disallowed for crawlers
/sitemap.xml	Sitemap reference

The sitemap listed the following public pages:

* /index.html
* /about.html
* /doctors.html
* /contact.html

During further directory inspection, the /old/ directory was found to expose an index listing containing:

mediroza_db_backup_2019.sql

The backup was accessible through the browser without an additional authentication step.

Security significance: The robots.txt entries were crawler instructions, not access-control restrictions. The exposed backup file represented a significant information-disclosure risk.

6. SQL Injection – Patient Portal Authentication Bypass

Critical finding

6.1 Affected component

Patient Portal login:

https://medirozahospital.com/patient/login.php

6.2 Initial observation

The patient portal presented a username and password form. Inspection of the page showed that the form submitted credentials through an HTTP POST request to login.php.

The normal login attempt returned an authentication failure message:

Incorrect password.

6.3 Vulnerability identification

During testing, a crafted SQL injection input was submitted through the username field.

The observed test input was:

admin'--

The application subsequently displayed a MySQL syntax warning when a related malformed input was tested. The warning exposed database error information, indicating that user-supplied input was being incorporated into a database query without appropriate handling.

6.4 Successful authentication bypass

The SQL injection test was then used to bypass the normal login failure.

The patient portal granted access without the expected successful username-and-password authentication process.

The portal displayed the authenticated user’s report page, titled:

My Lab Reports

This confirmed that the authentication mechanism was vulnerable to SQL injection.

6.5 Evidence

The following evidence was captured:

1. Patient portal login page.
2. SQL injection input entered in the username field.
3. MySQL syntax warning displayed by the application.
4. Successful access to the patient portal.
5. Authenticated laboratory report listing.

6.6 Impact

An attacker could potentially:

* Bypass the patient portal authentication mechanism.
* Access laboratory reports without valid credentials.
* View or download confidential documents.
* Obtain information that should be restricted to authenticated users.

The vulnerability is classified as Critical because it allows authentication bypass and access to confidential information.

7. Confidential Laboratory Reports Discovered

Following successful access to the patient portal, three password-protected PDF reports were displayed.

No.	Report title	Lab reference	Report date	File status
1	Pathology Report – S. Dlamini	LR-2024-1187	04 November 2024	PDF encrypted
2	Pathology Report – P. Reddy	LR-2024-1192	05 November 2024	PDF encrypted
3	Pathology Report – E. Thompson	LR-2024-1205	06 November 2024	PDF encrypted

The portal indicated that the reports were password-protected and that the password would be provided by the laboratory.

Finding: Confidential laboratory documents were accessible after bypassing the portal’s authentication mechanism.

The assessment evidence confirms that the three files were listed and available for download. Their successful decryption or the contents of the reports are not claimed in this report because that evidence was not provided.

8. Exposed Database Backup

Critical finding

8.1 Discovered file

/old/mediroza_db_backup_2019.sql

The directory listing showed the database backup file with an approximate displayed size of 7 KB.

The SQL file contained database creation statements, table structures and inserted records.

The backup identified the database as:

mediroza_db

It also contained the following tables:

* staff
* shareholders

8.2 Staff information exposed

The staff table contained the following fields:

Field	Description
id	Staff record identifier
full_name	Employee name
job_title	Employee designation
department	Department or work area
email	Employee email address
phone	Employee contact number
monthly_salary_zar	Monthly salary in South African rand
date_joined	Employment joining date

The SQL dump included multiple employee records and their associated employment and salary information.

8.3 Shareholder information exposed

The shareholders table contained the following fields:

Field	Description
id	Shareholder record identifier
shareholder_name	Shareholder name
share_percent	Percentage of shares held
shares_held	Number of shares held
share_class	Share classification

The backup contained shareholder records with shareholding percentages, number of shares and share classifications.

8.4 Sensitive information disclosure summary

Information category	Exposure confirmed
Employee names	Yes
Employee job titles	Yes
Departments	Yes
Employee email addresses	Yes
Employee phone numbers	Yes
Monthly employee salaries	Yes
Employment joining dates	Yes
Shareholder names	Yes
Shareholding percentages	Yes
Number of shares held	Yes
Share classifications	Yes

Security impact: An unauthenticated visitor able to access the exposed backup could obtain internal employee and shareholder information. This creates risks involving privacy, financial confidentiality, targeted social engineering and organisational security.

The screenshots establish the presence of these records and fields. Exact salary and shareholding figures should be transcribed from the original SQL file before being included in a detailed numerical appendix.

9. Consolidated Findings and Risk Assessment

ID	Finding	Severity	Evidence
F-01	SQL injection in patient portal login	Critical	Authentication bypass and portal access
F-02	Exposed SQL database backup	Critical	Public /old/ directory listing
F-03	Confidential laboratory reports accessible after bypass	High	Three reports listed in portal
F-04	Database error messages disclosed	Medium	MySQL syntax warning
F-05	Internal employee and shareholder information exposed	Critical	SQL backup table structures and records
F-06	Sensitive directories listed in robots.txt	Informational	robots.txt entries

10. Recommendations

10.1 Prevent SQL Injection

* Use parameterised queries or prepared statements for all database operations.
* Avoid constructing SQL queries by concatenating user input.
* Validate and sanitise input as an additional security measure.
* Use a database account with only the permissions required by the application.
* Conduct security testing of all authentication endpoints.

10.2 Secure Authentication

* Implement secure server-side authentication.
* Use proper password hashing, such as Argon2id or bcrypt.
* Regenerate the session identifier after successful authentication.
* Apply rate limiting and appropriate account protection.
* Ensure authentication checks are enforced on every protected resource.

10.3 Protect Database Backups

* Remove database backups from publicly accessible web directories.
* Store backups outside the web root.
* Restrict access through appropriate server permissions.
* Encrypt backups at rest and protect encryption keys.
* Review historical backups for sensitive information.

10.4 Protect Confidential Documents

* Enforce server-side authorisation before allowing report downloads.
* Ensure users can access only their own authorised documents.
* Store confidential PDFs outside publicly accessible directories.
* Use secure, time-limited download links where appropriate.
* Review access logs for unauthorised document retrieval.

10.5 Improve Error Handling

* Disable detailed database error messages in production.
* Log technical errors securely on the server.
* Display generic authentication and application errors to users.

11. Conclusion

The assessment identified serious security weaknesses in the Mediroza Hospital web application.

The most significant issue was an SQL injection vulnerability in the patient portal login mechanism. The vulnerability allowed the normal authentication failure to be bypassed and provided access to the portal’s laboratory report listing.

The assessment also identified a publicly accessible SQL database backup containing employee and shareholder records, including salary and shareholding information.

These findings demonstrate that the application requires immediate remediation of its authentication and database access controls.

The organisation should prioritise fixing the SQL injection vulnerability, removing exposed database backups, and enforcing server-side access controls for confidential documents.

A follow-up penetration test should be conducted after remediation to confirm that the vulnerabilities have been resolved.

⸻

12. Evidence Register

Evidence ID	Description
E-01	WHOIS domain enumeration results
E-02	DNSRecon output
E-03	WhatWeb technology identification
E-04	robots.txt contents
E-05	sitemap.xml contents
E-06	/old/ directory index showing the SQL backup
E-07	SQL backup showing the staff table and records
E-08	SQL backup showing the shareholders table and records
E-09	Patient portal login form
E-10	SQL injection input and MySQL error message
E-11	Successful patient portal access
E-12	My Lab Reports page listing three encrypted PDFs

13. Assessment Limitations

This report is based on the screenshots and observations supplied during the assessment.

The following items have not been independently verified from the available evidence:

* Successful decryption of the three PDF reports.
* The complete numerical salary list for every employee.
* The complete numerical shareholder holdings list.
* Remediation or retesting of the identified vulnerabilities.

No unverified figures or successful outcomes have been added.

Report status: Final assessment report based on the evidence supplied.



