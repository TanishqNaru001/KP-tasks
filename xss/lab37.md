## Title
Stored Cross-Site Scripting (XSS) - Support Center (DigitalOcean Clone)

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Support Center (DigitalOcean Clone) at `https://kzlabs.in/subdomains/digitalocean/` is vulnerable to Stored Cross-Site Scripting (XSS) in the support ticket creation functionality.

The application accepts user-supplied input in the **Subject** and **Description** fields when creating a support ticket and stores this content without properly sanitizing it. When the submitted ticket is later viewed, the stored content is rendered without appropriate output encoding, causing the injected JavaScript payload to execute in the browser.

Because the payload is persistent, any user who views an affected support ticket may trigger the malicious JavaScript.

## Vulnerable Endpoints

**Registration:**

`https://kzlabs.in/subdomains/digitalocean/index.php?view=register`

**Login:**

`https://kzlabs.in/subdomains/digitalocean/index.php`

**Injection Point - Create Support Ticket:**

`https://kzlabs.in/subdomains/digitalocean/index.php?view=create`

**Execution Point:**

The payload executes when the submitted support ticket is opened and the stored **Subject** and **Description** fields are rendered.

## Vulnerable Fields

| Field | Injection Location | Trigger Location |
|---|---|---|
| Subject | Create Support Ticket page | Submitted ticket view |
| Description | Create Support Ticket page | Submitted ticket view |

## Steps to Reproduce

### Step 1 - Register an account
1. Navigate to:

   `https://kzlabs.in/subdomains/digitalocean/index.php?view=register`

2. Create a new account.

### Step 2 - Log in
1. Navigate to:

   `https://kzlabs.in/subdomains/digitalocean/index.php`

2. Log in using the registered account credentials.

### Step 3 - Create a malicious support ticket
1. Navigate to the Create Support Ticket page:

   `https://kzlabs.in/subdomains/digitalocean/index.php?view=create`

2. In the **Subject** field, enter the following XSS payload:

   ```text
   tanishq'"><ImG SrC=x ONErrOR=confirm(1)>
   ```

3. In the **Description** field, enter the same XSS payload:

   ```text
   tanishq'"><ImG SrC=x ONErrOR=confirm(1)>
   ```

4. Submit the support ticket.

### Step 4 - Verify Stored XSS execution
1. Open or view the newly created support ticket.

2. Observe that the payload stored in the **Subject** field executes when the ticket is displayed.

3. Observe that the payload stored in the **Description** field also executes when the ticket content is displayed.

4. A JavaScript confirm box displaying `1` confirms successful execution of the stored XSS payload.

5. For a stronger Proof of Concept, open the affected ticket from another user session or browser to verify that the payload executes for other users viewing the ticket.

## Payload Used

```text
tanishq'"><ImG SrC=x ONErrOR=confirm(1)>
```

## Proof of Concept

<img width="1917" height="1032" alt="image" src="https://github.com/user-attachments/assets/f7802eb5-d2f4-4308-a86e-ccda26bc497b" />
<img width="1917" height="1035" alt="image" src="https://github.com/user-attachments/assets/f14d8ff2-d0e9-481a-9b4f-9c6e05f0b738" />
<img width="1917" height="988" alt="image" src="https://github.com/user-attachments/assets/8c4d2d29-c051-449a-8454-d132e0444f29" />




## Impact

Stored XSS in a support ticketing platform can have significant security consequences because support tickets may be viewed by multiple users, including support agents, administrators, and other privileged users.

- **Any user who views an affected support ticket may execute attacker-controlled JavaScript in their browser.**
- The malicious payload persists in the application's database until the ticket is deleted or the stored content is properly sanitized.
- An attacker may potentially perform actions in the context of a victim's authenticated session, depending on the application's security controls.
- If support agents or administrators view the malicious ticket, the vulnerability could affect privileged accounts.
- Attackers could manipulate the application's interface and display phishing prompts or misleading content to victims.
- Because the payload is stored server-side, the attacker does not need to send a malicious link repeatedly; the payload can execute whenever the affected ticket is viewed.
- The presence of Stored XSS in both the **Subject** and **Description** fields suggests that other user-controlled fields in the application may also be vulnerable.

## Recommendations for Fix

1. Apply context-aware output encoding to all user-controlled data before rendering it in HTML.

   For PHP applications, use:

   ```php
   htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
   ```

2. Ensure that both the **Subject** and **Description** fields are properly encoded before being displayed anywhere in the application.

3. Implement server-side input validation according to the expected format of each field.

4. If the **Description** field intentionally supports HTML or rich-text formatting, use a trusted server-side HTML sanitization library such as **HTMLPurifier** instead of custom filtering or denylist-based filtering.

5. Implement a strict **Content-Security-Policy (CSP)** that avoids `unsafe-inline` and restricts unauthorized JavaScript execution.

6. Audit all other support ticket fields and user-generated content throughout the application for similar Stored XSS vulnerabilities.

7. Treat all user-supplied input as untrusted and apply appropriate output encoding based on the context in which the data is rendered.
