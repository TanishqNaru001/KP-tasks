# Time-Based Blind SQL Injection in `id` GET Parameter on Courses Subdomain

## Vulnerability Type
SQL Injection (Blind, Time-Based) - CWE-89

## Summary
The `id` GET parameter on the `/subdomains/courses/` endpoint is not properly sanitized before being used in a backend SQL query. This allows an attacker to inject conditional SQL logic that triggers a measurable response delay, confirming the parameter is directly concatenated into a SQL statement without parameterization. An attacker can exploit this to blindly extract database contents (schema names, table data, credentials, etc.) without any visible output on the page.

## Vulnerable Endpoint
```
GET https://kzlabs.in/subdomains/courses/?id=1
```

## Steps to Reproduce
1. Intercept the request to `/subdomains/courses/?id=1` using Burp Suite.
2. Send the request to Intruder, marking the `id` parameter value as the injection point.
3. Load a time-based blind SQLi payload list and start the attack.
4. Observe that one payload causes a consistent ~10 second response delay compared to the baseline (~sub-second) response time for `id=1`.
5. Re-send the identified payload directly to confirm the delay is reproducible and not a network fluke.

## Payload Used
```
1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
```

## Proof of Concept Request
<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/6f28514d-0d0e-49da-a54a-af968c2e7611" />
<img width="1912" height="1076" alt="image" src="https://github.com/user-attachments/assets/b9085f7d-9b63-4ff2-8297-318c0e681d95" />


## Impact
- **Confidentiality:** An attacker can perform blind data extraction - database name, table/column names, and potentially sensitive data (user credentials, PII) - one bit/character at a time using conditional time delays.
- **Availability:** Because the payload can control sleep duration, an attacker could craft heavy/nested sleep conditions to cause resource exhaustion or denial-of-service on the database layer.
- **Severity:** High - unauthenticated, exploitable via a simple GET parameter, no user interaction required.

## Recommendations for Fix
1. **Use parameterized queries / prepared statements** for all database calls involving the `id` parameter - never concatenate user input directly into SQL.
2. **Input validation** - enforce that `id` is strictly numeric (e.g., integer cast/whitelist) before it reaches the query layer.
3. **Least privilege DB account** - ensure the application's DB user does not have unnecessary privileges (e.g., `FILE`, admin rights) that could escalate impact.
4. **WAF rule** as a short-term mitigation to block common SQLi patterns (`XOR`, `SLEEP`, `IF(`, `UNION SELECT`) on this parameter while the code fix is deployed.
5. **Regular SAST/DAST scanning** on this subdomain to catch similar injectable parameters elsewhere in the app.
