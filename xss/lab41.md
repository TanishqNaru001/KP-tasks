## Title
Stored Cross-Site Scripting (XSS) - Shopify CMS / Article Writing Platform

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Shopify-style CMS application at `https://kzlabs.in/subdomains/cms/` is vulnerable to Stored Cross-Site Scripting (XSS) through the article writing functionality.

After registration and authentication, users can create articles using the **Write Now** feature. The application provides fields including **Title**, **Tag**, and **Body**. While directly entering the payload into the normal fields does not trigger execution, the **Body** editor includes an **HTML mode**.

By switching the Body editor to HTML mode and inserting an XSS payload, the malicious content is saved and later executed when the article is rendered.

## Vulnerable Endpoint

**CMS Application / Article Writing Platform:**

`https://kzlabs.in/subdomains/cms/`

**Injection Point:**

**Write Now → Body → HTML Mode**

**Execution Point:**

The payload executes when the saved article is rendered.

## Vulnerable Fields

| Field | Result |
|---|---|
| Title | Payload does not execute |
| Tag | Payload does not execute |
| Body (Normal Editor) | Payload does not execute |
| Body (HTML Mode) | Stored XSS payload executes after saving |

## Steps to Reproduce

### Step 1 - Register and access the CMS
1. Register and log in to the application.
2. Navigate to:

   `https://kzlabs.in/subdomains/cms/`

### Step 2 - Open the article creation feature
1. Locate and click the **Write Now** option.
2. The article creation/editor page will open.

### Step 3 - Test the normal input fields
1. Enter an XSS payload into the **Title** field.
2. Enter an XSS payload into the **Tag** field.
3. Enter an XSS payload into the normal **Body** editor.
4. Observe that the payload does not execute through these normal input methods.

### Step 4 - Switch the Body editor to HTML mode
1. Locate the **HTML** option in the Body editor.
2. Switch the editor from the normal writing mode to **HTML mode**.
3. Enter the XSS payload into the HTML editor.

### Step 5 - Save the article
1. Save or publish the article.
2. Open or view the saved article.
3. Observe that the payload inserted through the **HTML mode** is rendered and executes.
4. Successful JavaScript execution confirms the presence of Stored XSS.

## Payload Used

```text
tanishq'"><Img SRc=x ONeRRor=confirm(100)>
```

## Proof of Concept

<img width="1862" height="926" alt="image" src="https://github.com/user-attachments/assets/402931b5-07ee-41f8-9c50-3b89bdfd4322" />
<img width="1917" height="1031" alt="image" src="https://github.com/user-attachments/assets/342a1750-726c-4aa6-a91a-034756f9ef73" />


## Impact

Stored XSS in an article-writing or CMS platform can have significant security consequences because malicious content may be stored and executed whenever the affected article is viewed.

- **Any user who views an affected article may execute attacker-controlled JavaScript in their browser.**
- The malicious payload persists until the article is deleted or properly sanitized.
- If articles can be viewed by multiple users, a single malicious article could potentially affect multiple victims.
- Attackers could manipulate the application's user interface or display phishing content.
- Depending on the application's security controls, attackers may potentially perform actions in the context of an authenticated victim.
- If administrators or content moderators review malicious articles, privileged users could also potentially be affected.
- The vulnerability is particularly significant because the application's HTML mode allows potentially dangerous user-controlled HTML to be stored and rendered.

## Recommendations for Fix

1. Do not directly render user-controlled HTML without proper sanitization.

2. If HTML formatting is intentionally supported in the article editor, sanitize all submitted HTML using a well-maintained server-side HTML sanitization library, such as **HTMLPurifier**.

3. Remove dangerous HTML elements and attributes, including:

   - `<script>` tags
   - Event handlers such as `onerror`, `onclick`, and `onload`
   - Dangerous URLs and JavaScript execution contexts

4. Apply context-aware output encoding wherever user-generated content is displayed.

5. Implement a strict **Content-Security-Policy (CSP)** that avoids `unsafe-inline` and restricts unauthorized script execution.

6. Ensure the HTML editor uses a proper allowlist of safe HTML tags and attributes instead of attempting to block known malicious payloads with a denylist.

7. Audit all other CMS and article-related functionality that accepts rich text or HTML input for similar Stored XSS vulnerabilities.

8. Treat all user-controlled HTML content as untrusted, even when it is submitted through an authenticated user's account.
