## Title
Stored Cross-Site Scripting (XSS) - Events & Polls Activity Center

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Events & Polls Activity Center at `https://kzlabs.in/subdomains/articles/` is vulnerable to Stored Cross-Site Scripting (XSS) in multiple user-controlled input fields.

The application stores user-supplied input without properly sanitizing it and renders the stored content without appropriate output encoding. As a result, malicious JavaScript payloads injected into fields such as **Username**, **Full Name**, **Event Title**, **Event Description**, and **Poll-related fields** can be stored by the application and later executed when the affected content is displayed.

The payload persists in the application and can execute whenever a user accesses the relevant page or activity containing the malicious content.

## Vulnerable Endpoints

**Registration:**
`https://kzlabs.in/subdomains/articles/index.php?view=register`

**Login:**
`https://kzlabs.in/subdomains/articles/index.php`

**Application / Payload Execution:**
`https://kzlabs.in/subdomains/articles/index.php`

## Vulnerable Fields

| Feature | Vulnerable Field | Trigger Location |
|---|---|---|
| User Registration | Username | Application page after login |
| User Registration | Full Name | Application page after login |
| Add Event | Event Title | Events section |
| Add Event | Event Description | Events section |
| Poll | Poll-related input fields | Poll section |

## Steps to Reproduce

### Step 1 - Register an account
1. Navigate to:
   `https://kzlabs.in/subdomains/articles/index.php?view=register`
2. Create a new account.
3. Insert an XSS payload into the **Username** and/or **Full Name** fields.
4. Complete the registration process.

### Step 2 - Log in and verify stored payload execution
1. Navigate to:
   `https://kzlabs.in/subdomains/articles/index.php`
2. Log in using the newly created account.
3. After successful authentication, observe that the stored payload inserted during registration executes when the affected **Username** or **Full Name** value is rendered.

### Step 3 - Inject payload into Event Title
1. After logging in, navigate to the **Add Event** functionality.
2. Enter an XSS payload into the **Event Title** field.
3. Submit/create the event.
4. Navigate to the location where the created event is displayed.
5. Observe that the stored payload executes when the Event Title is rendered.

### Step 4 - Inject payload into Event Description
1. Navigate to the **Add Event** functionality.
2. Enter an XSS payload into the **Event Description** field.
3. Submit/create the event.
4. Open or view the created event.
5. Observe that the stored payload executes when the Event Description is rendered.

### Step 5 - Verify Stored XSS in Polls
1. Navigate to the **Poll** functionality.
2. Insert an XSS payload into the affected poll input field.
3. Submit/create the poll.
4. Navigate to the location where the poll is displayed.
5. Observe that the stored payload executes when the stored poll content is rendered.

## Payload Used

```text
[Insert the exact XSS payload used during testing]
```

## Proof of Concept

<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/c9eca045-4370-4e77-bace-81f4604a2293" />
<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/46552901-6b4d-4939-93f4-79f81ce0112f" />
<img width="1917" height="970" alt="image" src="https://github.com/user-attachments/assets/ade9901c-4669-4cf0-bc01-5361fc51b1c4" />
<img width="1917" height="981" alt="image" src="https://github.com/user-attachments/assets/5d80822a-a19c-4a1e-a6f1-d0367c4efbeb" />
<img width="1917" height="967" alt="image" src="https://github.com/user-attachments/assets/60ece5c1-de4f-4a43-a60d-2f73fc30589b" />


## Impact

Stored XSS vulnerabilities in an Events & Polls Activity Center can have significant security impact because the malicious payload is stored by the application and can execute in the browsers of users who view the affected content.

- **Any user who views affected events, polls, or user-generated content may have JavaScript executed in their browser.**
- Because the payload is persistent, the attacker does not need to repeatedly send a malicious link to victims.
- An attacker may perform actions in the context of a victim's authenticated session, depending on application protections and session configuration.
- Malicious content in event titles or descriptions could affect multiple users who browse the Events section.
- Malicious payloads stored in polls could execute for users interacting with or viewing the affected poll.
- Stored XSS can also be used for phishing attacks, user-interface manipulation, or unauthorized actions performed through the victim's browser.
- Multiple vulnerable fields indicate that the application may have a broader lack of proper output encoding across user-generated content.

## Recommendations for Fix

1. Apply context-aware output encoding to all user-controlled data before rendering it in HTML. For PHP applications, use appropriate encoding such as:

   ```php
   htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
   ```

2. Ensure that all user-controlled fields, including **Username**, **Full Name**, **Event Title**, **Event Description**, and **Poll fields**, are properly encoded before being displayed.

3. Implement server-side input validation based on the expected format of each field.

4. If Event Descriptions or Poll content intentionally support HTML or rich text formatting, use a well-maintained server-side HTML sanitization library such as **HTMLPurifier** instead of custom filtering or denylist-based approaches.

5. Implement a strong **Content-Security-Policy (CSP)** that avoids `unsafe-inline` and restricts unauthorized script execution.

6. Audit all other user-generated input fields throughout the application for the same vulnerability, as multiple affected locations suggest a shared issue with output handling.

7. Use secure development practices that treat all user-controlled input as untrusted and apply output encoding according to the context in which the data is rendered.
