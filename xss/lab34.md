## Title
Stored Cross-Site Scripting (XSS) - Community Hub Forum 

## Vulnerability Type
Stored XSS (Persistent XSS)

## Severity
High

## Summary
The Community Hub registration form at `https://kzlabs.in/subdomains/community/` accepts and stores user-supplied input across multiple fields without sanitization or output encoding. Payloads injected during registration are persisted to the database and later executed in the browser of any user who views the attacker's profile or reads their comments - without any further interaction from the attacker. This is fundamentally more severe than the Reflected XSS findings in this lab: a victim does not need to click a crafted link; simply browsing to an affected page is sufficient to trigger execution.

## Vulnerable Endpoint
**Registration (injection point):** `https://kzlabs.in/subdomains/community/`

**Trigger points (execution context):**
- `https://kzlabs.in/subdomains/community/index.php?view=profile` - Organization/Institution payload triggers on profile view
- `https://kzlabs.in/subdomains/community/index.php` - First Name, Middle Name, Last Name, and Affiliation/Bio payloads trigger when a comment is posted

## Vulnerable Fields
| Field | Trigger Location |
|---|---|
| First Name | Comment section (any page) |
| Middle Name | Comment section (any page) |
| Last Name | Comment section (any page) |
| Username | To be confirmed |
| Organization / Institution | Profile page |
| Affiliation / Bio | Comment section (any page) |

## Steps to Reproduce

### Step 1 - Register with payloads in all fields
1. Navigate to: `https://kzlabs.in/subdomains/community/`
2. Fill the registration form with the following payloads:
   - **First Name:** `tanishq'"><ImG SrC=x OnErrOr=confirm(1)>`
   - **Middle Name:** `tanishq'"><ImG SrC=x OnErrOr=confirm(2)>`
   - **Last Name:** `tanishq'"><ImG SrC=x OnErrOr=confirm(3)>`
   - **Username:** `tanishq'"><ImG SrC=x OnErrOr=confirm(4)>`
   - **Organization / Institution:** `tanishq'"><ImG SrC=x OnErrOr=confirm(5)>`
   - **Affiliation / Bio:** `tanishq'"><ImG SrC=x OnErrOr=confirm(6)>`
   - Use valid placeholder values for Email, Phone, and Password fields.
3. Submit the registration form.

### Step 2 - Trigger via profile view
1. Log in as any user (including the attacker's own account or a separate victim account).
2. Navigate to: `https://kzlabs.in/subdomains/community/index.php?view=profile`
3. Observe confirm dialog(s) firing - confirming stored payloads in the Organization/Institution field execute when the profile page is rendered.

### Step 3 - Trigger via comment
1. While logged in as the attacker account, post any comment on: `https://kzlabs.in/subdomains/community/index.php`
2. Observe confirm dialogs firing - confirming stored payloads in the First Name, Middle Name, Last Name, and Affiliation/Bio fields execute in the browser of any user viewing that comment thread, including other logged-in users.

## Payload Used
```
tanishq'"><ImG SrC=x OnErrOr=confirm(N)>
```
*(Unique `confirm(N)` values used per field to confirm each field's vulnerability independently)*

## Proof of Concept

<img width="1918" height="1006" alt="1" src="https://github.com/user-attachments/assets/4736f057-e354-4474-ada5-2378ec458592" />
<img width="1920" height="1004" alt="2" src="https://github.com/user-attachments/assets/090e228a-9c5c-4e0f-a4e4-b9895b7e238d" />
<img width="1920" height="1000" alt="3" src="https://github.com/user-attachments/assets/c0ce2d21-e6e6-4fb1-8304-509d5a275157" />
<img width="1918" height="958" alt="4" src="https://github.com/user-attachments/assets/c1e1004a-6db8-46dc-874a-67d3437a8b47" />
<img width="1918" height="953" alt="5" src="https://github.com/user-attachments/assets/8d2bad54-4968-4651-85e0-36a16b52b07d" />
<img width="1916" height="958" alt="6" src="https://github.com/user-attachments/assets/23eb6512-a0cb-459d-99d4-b13280610faf" />





## Impact
Stored XSS carries significantly higher impact than Reflected XSS since no victim interaction beyond normal browsing is required:

- **Any authenticated user who views the attacker's profile or reads their comments has JavaScript executed in their browser context** - without clicking any crafted link.
- An attacker can steal session cookies (`document.cookie`) from every victim who views the affected pages, potentially leading to mass account takeover across all forum users.
- The payload can be used to silently perform actions on behalf of victims (post content, change settings, exfiltrate data) while they browse normally.
- With multiple fields triggering on the high-traffic comment listing page, a single registered attacker account can affect every active user of the forum simultaneously.
- Unlike Reflected XSS, the payload persists until the attacker's account is deleted or the stored data is sanitized - WAF rules blocking crafted URLs provide no protection since the payload is served from the database.

## Recommendations for Fix
1. Apply context-aware output encoding (e.g. `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')` in PHP) on **all** user-supplied registration fields before rendering them anywhere in the application - profile pages, comment sections, admin panels, and any other display context.
2. Implement server-side input validation at registration time to reject inputs containing HTML metacharacters (`<`, `>`, `"`, `'`) in fields where HTML is not expected (names, usernames, organization).
3. For the Affiliation/Bio field, if rich text is intentionally supported, use a server-side HTML sanitization library (e.g. HTMLPurifier for PHP) rather than a custom filter - never a denylist.
4. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) across the entire community subdomain, which would block inline event-handler execution (`onerror`, `onclick`, etc.) even if stored XSS payloads reach the page.
5. Audit all other stored fields in the application (comments, discussion posts, article submissions) for the same issue, as the root cause is likely a shared rendering layer that does not encode output.
