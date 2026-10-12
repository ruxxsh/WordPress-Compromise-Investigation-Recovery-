# WordPress Compromise Investigation & Recovery

*Incident response on a real client site: a persistent attacker returned repeatedly after a credential leak. Persistence was removed across the entire hosting account and the site was rebuilt on clean WordPress core.*

| | |
|---|---|
| **Type** | Real client engagement |
| **Role** | Incident responder (sole investigator) |
| **Environment** | WordPress on shared cPanel hosting, PHP 8.1 |
| **Status** | Resolved, no reinfection as of 5 Oct 2026 |

> No live malware samples are published. Indicators are defanged.

## 1. Executive Summary
The client reported that her hosting and WordPress admin credentials had been leaked. Shortly afterwards her site was compromised, and after an initial cleanup the attacker returned, which indicated persistence. I investigated the whole hosting account instead of only the visible site, found and removed the backdoor and a rogue admin account, rebuilt the site on a fresh WordPress core with scanned content, rotated credentials, and gave hardening recommendations.

**Impact:** SEO spam, redirects to a malicious URL, and search-result hijacking. No data loss was confirmed.

## 2. Scope
- **Affected site:** hxxp://wohprague[.]cz
- **Additional sites/directories in the same hosting account:** 3, all reviewed

## 3. Detection and Initial Report
- **How it was noticed:** client complaint about unexpected redirects
- **Reported leaked credentials:** hosting account and WordPress admin
- *The source and date of the leak were not independently verified.*

## 4. Timeline
| Date | Event |
|---|---|
| 3 Jun 2026 | Initial compromise symptoms observed |
| 3 Jun 2026 | Client reported credential leak |
| 3 Jun 2026 | First cleanup performed |
| 4 Jun 2026 | Attacker returned; full-account investigation started |
| 4 Jun 2026 | Persistence found and removed |
| 5 Jun 2026 | Rebuild completed, monitoring started |
| 5 Oct 2026 | No reinfection observed |

## 5. Investigation
**Why the attacker kept coming back:** the first cleanup only covered the visible site. Persistence elsewhere in the account let the attacker re-enter.

**Method**
- Reviewed all other directories in the hosting account, not just the affected site
- Compared files against clean WordPress core (checksum verification and diff against a fresh download)
- Scanned wp-content (plugins, themes, uploads) with Wordfence Security

**Findings**
| # | Finding | Location | Notes |
|---|---|---|---|
| 1 | Backdoor web shell | `public_html/` | Base64-obfuscated PHP payload |
| 2 | Rogue administrator account | WordPress users: `admin000` | Unauthorized admin created for persistent access |
| 3 | Modified `index.php` | Site root | Injected code redirecting visitors to a malicious URL and hijacking search results (SEO spam) |
| 4 | Immutable attribute on malicious file(s) | Site root | File permissions altered so the infected file could not be edited or deleted normally |

## 6. Indicators of Compromise (defanged)
| Type | Indicator |
|---|---|
| Rogue user | `admin000` |
| Modified file | `index.php` (malicious redirect and SEO code) |
| Web shell | Base64-obfuscated PHP backdoor in `public_html/` |
| File attribute | Immutable flag set on infected file(s) |
| Affected paths | `public_html/` and sibling domain folders |
| Malicious file | AMP.php 

## 7. Root Cause
- **Likely initial access:** leaked credentials (per client report). Not confirmed in logs.
- **Possible contributing factors (not verified):** no 2FA, reused password, outdated plugins.

## 8. Remediation
1. Backed up `wp-content`, the database, and `wp-config.php`.
2. Replaced all other files with a fresh WordPress download.
3. Scanned the preserved data and removed the obfuscated malicious code found.
4. Cleared the immutable attribute on the affected file(s) and restored them from clean copies.
5. Removed the rogue admin account.
6. Repeated the process for the 3 other directories in the account.
7. Rotated credentials: hosting, WordPress admins, database, and security keys/salts.

## 9. Recovery and Verification
- Rebuilt from clean files from the official WordPress repository
- Monitored from 5 Jun 2026; **no reinfection observed as of 5 Oct 2026**

## 10. MITRE ATT&CK Mapping
| Tactic | Technique | Evidence |
|---|---|---|
| Initial Access | T1078 Valid Accounts (*likely*) | Client-reported credential leak; not confirmed in logs |
| Persistence | T1505.003 Server Software Component: Web Shell | Base64-obfuscated backdoor in `public_html/` |
| Defense Evasion | T1222.002 File and Directory Permissions Modification | Immutable attribute set to block removal of infected files |

## 11. Recommendations
- Enforce 2FA on WordPress and the hosting panel
- Use unique strong passwords with a password manager
- Isolate each site in its own hosting account
- Keep core, plugins, and themes updated; remove unused ones
- Restrict file permissions and disable the dashboard file editor
- Keep off-site backups and file-integrity monitoring
- Consider cyber insurance and an incident-support arrangement

## 12. Lessons Learned
- **Scope matters:** cleaning one site fails when other sites share the same account or credentials.
- **Rotate credentials early:** if the attacker still has valid access, cleanup alone won't hold.
- **Check file attributes:** a file that can't be removed may be immutable, not a permissions problem.
- **Monitoring is part of recovery:** continuous alerts catch reinfection sooner.

## Tools Used
Wordfence Security, fresh WordPress core files, FTP, SSH, Cpanel, VirusTotal.
