## Title
Stored Cross-Site Scripting (XSS) - Bug Bounty Platform (HackerOne Clone)

## Vulnerability Type
Stored XSS (Persistent Cross-Site Scripting)

## Summary
The Bug Bounty Platform (HackerOne Clone) at `https://kzlabs.in/subdomains/hackerone/` is vulnerable to Stored Cross-Site Scripting (XSS) in multiple fields used when submitting vulnerability reports.

The application accepts user-controlled input in the **Title**, **Impact**, and **Description** fields and stores the submitted content. The stored data is later rendered in the report dashboard without proper sanitization or context-aware output encoding, allowing injected JavaScript payloads to execute when the submitted report is opened.

Because the payload is persistent, any user who accesses an affected report may trigger the malicious JavaScript in their browser.

## Vulnerable Endpoints

**Registration:**

`https://kzlabs.in/subdomains/hackerone/index.php?view=register`

**Login:**

`https://kzlabs.in/subdomains/hackerone/index.php`

**Injection Point - Submit Vulnerability Report:**

`https://kzlabs.in/subdomains/hackerone/index.php?view=submit`

**Execution Point - View Submitted Report:**

`https://kzlabs.in/subdomains/hackerone/index.php?view=dashboard&report_id=7`

## Vulnerable Fields

| Field | Injection Location | Trigger Location |
|---|---|---|
| Title | Submit Report page | Submitted report page |
| Impact | Submit Report page | Submitted report page |
| Description | Submit Report page | Submitted report page |

## Steps to Reproduce

### Step 1 - Register an account
1. Navigate to:
   `https://kzlabs.in/subdomains/hackerone/index.php?view=register`
2. Create a new account.

### Step 2 - Log in
1. Navigate to:
   `https://kzlabs.in/subdomains/hackerone/index.php`
2. Log in using the registered account credentials.

### Step 3 - Submit a malicious vulnerability report
1. Navigate to the vulnerability report submission page:

   `https://kzlabs.in/subdomains/hackerone/index.php?view=submit`

2. Insert an XSS test payload into the **Title** field.

3. Insert the same or another XSS test payload into the **Impact** field.

4. Insert an XSS test payload into the **Description** field.

5. Submit the vulnerability report.

### Step 4 - Open the submitted report
1. Navigate to the dashboard and open the submitted report.

2. Example report URL:

   `https://kzlabs.in/subdomains/hackerone/index.php?view=dashboard&report_id=7`

3. Observe that the payload stored in the **Title**, **Impact**, and **Description** fields executes when the report content is rendered.

4. This confirms that the application stores attacker-controlled input and later displays it without sufficient output encoding or sanitization.

## Payload Used

```text
tanishq'"><ImG SrC=x ONErrOR=confirm(1)>
```

## Proof of Concept

<img width="1917" height="1040" alt="image" src="https://github.com/user-attachments/assets/3f25df3f-623c-4c4e-9556-90038abbc3c4" />
<img width="1917" height="1042" alt="image" src="https://github.com/user-attachments/assets/131694df-7901-476c-aa7a-1e68641dd53a" />
<img width="1910" height="1040" alt="image" src="https://github.com/user-attachments/assets/e97555ec-1eef-41a7-a34c-8f175c5112b4" />
<img width="1917" height="1041" alt="image" src="https://github.com/user-attachments/assets/18457843-7897-4762-ae45-275c1822b720" />



## Impact

Stored XSS in a bug bounty platform can have significant security consequences because vulnerability reports may be viewed by multiple users, including researchers, organization members, triagers, and administrators.

- **Any user who opens an affected vulnerability report may execute attacker-controlled JavaScript in their browser.**
- The malicious payload persists in the application's database until the report is removed or sanitized.
- An attacker could potentially perform actions in the context of a victim's authenticated session, depending on the application's security controls.
- If privileged users such as administrators or program managers review the malicious report, the impact could be significantly greater.
- Attackers could manipulate the user interface and display fake content or phishing prompts to victims.
- Multiple vulnerable fields suggest that the application may have a broader issue with handling and rendering user-controlled content.
- Since the payload is stored server-side, the attacker does not need to repeatedly distribute a malicious URL; the payload can execute whenever the affected report is viewed.

## Recommendations for Fix

1. Apply context-aware output encoding to all user-controlled data before rendering it in HTML.

   For PHP applications, use appropriate output encoding such as:

   ```php
   htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
   ```

2. Ensure that the **Title**, **Impact**, and **Description** fields are encoded according to the context in which they are displayed.

3. Implement server-side input validation based on the expected format and purpose of each field.

4. If rich-text formatting is intentionally supported in report descriptions, use a trusted server-side HTML sanitization library such as **HTMLPurifier** rather than custom filters or denylist-based approaches.

5. Implement a strict **Content-Security-Policy (CSP)** that avoids `unsafe-inline` and restricts unauthorized JavaScript execution.

6. Review all other user-controlled fields throughout the Bug Bounty Platform for similar vulnerabilities.

7. Treat all user-supplied content as untrusted and apply proper output encoding at every location where the content is rendered.

8. Test the application from different user roles, particularly privileged roles such as administrators and triagers, to determine the full potential impact of the Stored XSS vulnerability.
