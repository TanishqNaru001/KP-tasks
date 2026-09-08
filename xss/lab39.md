## Title
Stored Cross-Site Scripting (XSS) - Video Streaming Service (Netflix Clone)

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The Video Streaming Service (Netflix Clone) at `https://kzlabs.in/subdomains/netflix/` is vulnerable to Stored Cross-Site Scripting (XSS) through the **Username** field during user registration.

The application allows a user to register through a popup registration form. User-controlled input supplied in the **Username** field is stored by the application and later rendered without proper sanitization or context-aware output encoding. As a result, an injected XSS payload can persist and execute when the username is displayed after registration.

## Vulnerable Endpoint

**Registration / Application:**

`https://kzlabs.in/subdomains/netflix/`

The registration functionality is accessible through a popup on the main application page.

## Vulnerable Field

| Field | Injection Location | Trigger Location |
|---|---|---|
| Username | Registration popup | Username displayed after registration |

## Steps to Reproduce

### Step 1 - Open the application
1. Navigate to:

   `https://kzlabs.in/subdomains/netflix/`

2. Open the registration popup from the application interface.

### Step 2 - Register with an XSS payload
1. Enter the required registration details.

2. In the **Username** field, enter an XSS test payload.

3. Complete the registration process.

### Step 3 - Verify Stored XSS execution
1. After successful registration, observe the location where the registered **Username** is displayed.

2. The payload stored in the Username field executes when the application renders the stored username.

3. Successful JavaScript execution confirms that the application is vulnerable to Stored XSS.

4. For a stronger Proof of Concept, access the affected account or profile from another session, where applicable, to verify whether the stored payload can execute for other users who view the affected username.

## Payload Used

```text
[Insert the exact XSS payload used during testing]
```

## Proof of Concept

### Stored XSS via Username

<img width="1917" height="937" alt="image" src="https://github.com/user-attachments/assets/302253a4-d080-4b51-9657-1b0c3c1a354b" />
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/8c1dd486-3374-4d31-bc02-12534e162a54" />


## Impact

Stored XSS in a video streaming platform can have security consequences because attacker-controlled content is stored by the application and executed when rendered in a user's browser.

- **The stored payload executes when the vulnerable Username value is rendered.**
- The malicious input persists until the affected account or stored data is removed or sanitized.
- Depending on where usernames are displayed throughout the application, other users may also be exposed to the malicious payload.
- An attacker may potentially manipulate the user interface or perform actions in the context of an affected user's authenticated session, depending on application security controls.
- If usernames are visible to privileged users or administrators, the vulnerability could potentially affect higher-privileged accounts.
- The vulnerability indicates that user-controlled data is not consistently handled with appropriate output encoding.

## Recommendations for Fix

1. Apply context-aware output encoding to all user-controlled data before rendering it in HTML.

   For PHP applications, an appropriate approach is:

   ```php
   htmlspecialchars($username, ENT_QUOTES, 'UTF-8');
   ```

2. Validate the **Username** field server-side and restrict it to an expected character set appropriate for usernames.

3. Do not allow HTML markup or special characters unless explicitly required.

4. Audit all locations where usernames and other user-controlled profile information are displayed.

5. Implement a strict **Content-Security-Policy (CSP)** to provide an additional layer of protection against unauthorized script execution.

6. Review all registration and profile-related fields for similar Stored XSS vulnerabilities.

7. Treat all user-supplied input as untrusted and ensure appropriate output encoding is applied according to the rendering context.
