## Title
Stored Cross-Site Scripting (XSS) - Twitter Ads / Network Report

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Twitter Ads-style application at `https://kzlabs.in/subdomains/ads/` is vulnerable to Stored Cross-Site Scripting (XSS) through the **Report Name** field in the **New Network Report** functionality.

After registration and authentication, a user can create a new network report. The application accepts attacker-controlled input in the **Report Name** field and stores or renders it without proper output encoding or sanitization. As a result, an injected XSS payload executes when the report name is displayed.

## Vulnerable Endpoint

**Application / Network Reports:**

`https://kzlabs.in/subdomains/ads/`

**Injection Point:**

**New Network Report → Report Name**

## Vulnerable Field

| Field | Injection Location | Trigger Location |
|---|---|---|
| Report Name | New Network Report form | When the report name is rendered/displayed |

## Steps to Reproduce

### Step 1 - Register and access the application
1. Register and log in to the application.
2. Navigate to:

   `https://kzlabs.in/subdomains/ads/`

### Step 2 - Open New Network Report
1. Locate and click the **New Network Report** button.
2. A **New Network Report** popup/form will appear.

### Step 3 - Inject the XSS payload
1. In the **Report Name** field, enter the following payload:

   ```text
   tanishq'"><Img SRc=x ONeRRor=confirm(100)>
   ```

2. Fill in the remaining required fields if necessary.
3. Click **Run & Save Report**.

### Step 4 - Verify payload execution
1. After submitting the report, observe the location where the **Report Name** is displayed.
2. The stored XSS payload executes when the application renders the malicious Report Name.
3. A JavaScript confirm box displaying `100` confirms successful XSS execution.

## Payload Used

```text
tanishq'"><Img SRc=x ONeRRor=confirm(100)>
```

## Proof of Concept

<img width="1917" height="935" alt="image" src="https://github.com/user-attachments/assets/d762f040-d468-4d51-b7b6-fa6cf1032d38" />
<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/20b1fbf7-f1e7-4318-9fd2-f575f22f5420" />


## Impact

Stored XSS in the Network Report functionality can have significant security consequences because malicious content may persist and execute whenever the affected report is viewed.

- **Any user who views an affected report may execute attacker-controlled JavaScript in their browser.**
- The malicious payload can remain stored until the report is deleted or properly sanitized.
- An attacker may potentially perform actions in the context of a victim's authenticated session, depending on the application's security controls.
- Attackers could manipulate the application's user interface or display phishing content to victims.
- If administrators or other privileged users can access submitted network reports, the vulnerability could potentially affect higher-privileged accounts.
- The vulnerability may indicate that other user-controlled fields within the advertising platform are also rendered without proper output encoding.

## Recommendations for Fix

1. Apply context-aware output encoding to the **Report Name** and all other user-controlled fields before rendering them.

   For PHP applications, use:

   ```php
   htmlspecialchars($report_name, ENT_QUOTES, 'UTF-8');
   ```

2. Implement server-side validation for the **Report Name** field based on the expected input format.

3. Ensure user-controlled data is never inserted directly into HTML without proper encoding.

4. Implement a strict **Content-Security-Policy (CSP)** that avoids `unsafe-inline` to provide an additional layer of protection against injected scripts.

5. Audit other report fields and user-controlled functionality throughout the application for similar Stored XSS vulnerabilities.

6. Treat all user-supplied input as untrusted and apply appropriate context-aware encoding wherever the data is displayed.
