## Title
Stored Cross-Site Scripting (XSS) - Acronis Forum / Q&A Platform

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Acronis-style Forum / Q&A platform at `https://kzlabs.in/subdomains/forum/` is vulnerable to Stored Cross-Site Scripting (XSS) in multiple user-controlled fields.

After registration and authentication, users can edit their profile and provide content in the **About Me** and **Signature** fields. The application stores this user-controlled content and renders it without sufficient sanitization or context-aware output encoding.

The application also provides an **Ask a Question** feature containing fields such as **Title**, **Body**, and **Tags**. During testing, the stored malicious content was observed executing when the affected content was rendered.

The **Signature** payload triggers after saving and is displayed in other areas of the application. The **About Me** payload triggers when the affected user profile is opened.

## Vulnerable Endpoint

**Forum Application:**

`https://kzlabs.in/subdomains/forum/`

## Vulnerable Features and Fields

| Feature | Vulnerable Field | Trigger Location |
|---|---|---|
| User Profile | Signature | After saving and when rendered in the application |
| User Profile | About Me | When the user profile is opened |
| Ask a Question | Title | Requires security review/testing |
| Ask a Question | Body | Requires security review/testing |
| Ask a Question | Tags | Requires security review/testing |

## Steps to Reproduce

### Step 1 - Register and access the forum
1. Register for an account on the application.
2. Navigate to:

   `https://kzlabs.in/subdomains/forum/`

3. Log in using the registered account.

### Step 2 - Navigate to the profile
1. Open the user profile section.
2. Locate the profile editing functionality.

### Step 3 - Test the Signature field
1. Locate the **Signature** field.
2. Enter an XSS test payload.
3. Save the profile changes.
4. Observe that the stored payload executes when the Signature is rendered after saving.

### Step 4 - Test the About Me field
1. Locate the **About Me** field in the profile settings.
2. Enter an XSS test payload.
3. Save the profile changes.
4. Open the affected user profile.
5. Observe that the stored payload executes when the **About Me** content is rendered.

### Step 5 - Navigate to Ask a Question
1. Locate and open the **Ask a Question** feature.
2. The question form contains the following fields:

   - Title
   - Body
   - Tags

3. Test each field individually with a harmless XSS proof-of-concept payload where authorized.
4. Save or submit the question and observe how the submitted content is rendered.

### Step 6 - Verify persistent execution
1. After saving the malicious profile content, navigate to other areas where the **Signature** is displayed.
2. Observe whether the stored payload executes when the Signature is rendered.
3. Open the affected user profile.
4. Observe that the stored payload in the **About Me** field executes when the profile content is displayed.

## Payload Used

```text
tanishq'"><Img SRc=x ONeRRor=confirm(N)>
```

## Proof of Concept

<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/37bfb23f-a169-47f2-9244-23e4435f0587" />
<img width="1912" height="1031" alt="image" src="https://github.com/user-attachments/assets/a0619ec7-30f8-42d1-a24b-2055b4e39cb0" />
<img width="1917" height="1027" alt="image" src="https://github.com/user-attachments/assets/f0bc0159-0b12-4b8e-bd38-a7dc8cbcf9a4" />
<img width="1916" height="1037" alt="image" src="https://github.com/user-attachments/assets/9c2c7bc9-feb0-47b6-98d4-a11a30ce0ade" />


## Impact

Stored XSS in a forum and Q&A platform can have significant security consequences because user-generated content may be viewed by multiple users.

- **Any user who views affected content may execute attacker-controlled JavaScript in their browser.**
- The malicious payload persists until the affected content is removed or properly sanitized.
- A malicious Signature could potentially affect users across multiple pages if Signatures are displayed alongside forum posts or questions.
- Malicious content stored in the About Me section can execute when users visit the affected profile.
- Depending on application security controls, an attacker may potentially perform actions in the context of an authenticated victim.
- Attackers could manipulate the user interface or display phishing content to other users.
- If administrators or moderators view affected content, privileged accounts could potentially be exposed.
- Multiple user-controlled content areas suggest that the application may have broader output encoding or HTML sanitization issues.

## Recommendations for Fix

1. Apply context-aware output encoding to all user-controlled content before rendering it.

2. For PHP applications, use appropriate output encoding such as:

   ```php
   htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
   ```

3. Ensure that the **About Me**, **Signature**, **Title**, **Body**, and **Tags** fields are handled according to the context in which they are displayed.

4. If profile fields or question bodies intentionally support HTML or rich-text formatting, sanitize the submitted HTML using a trusted server-side sanitization library such as **HTMLPurifier**.

5. Use an allowlist of safe HTML tags and attributes rather than attempting to block known malicious payloads with a denylist.

6. Remove dangerous event-handler attributes such as `onerror`, `onclick`, and `onload` from any user-controlled HTML.

7. Implement a strict **Content-Security-Policy (CSP)** that avoids `unsafe-inline` and provides an additional layer of protection against injected scripts.

8. Audit all forum posts, questions, comments, profile fields, and other user-generated content for similar Stored XSS vulnerabilities.

9. Treat all user-supplied input as untrusted and apply appropriate encoding or sanitization at every rendering location.
