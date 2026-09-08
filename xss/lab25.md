## Title
Reflected Cross-Site Scripting (XSS) - Mixed Security Parameters 

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The settings page on `https://kzlabs.in/subdomains/settings/` reflects a large number of arbitrary GET parameters directly into the page's HTML response, discovered via parameter-mining with `x8` (34 reflecting parameters identified, including `academy`, `password`, `token`, `csrf_token`, `api_key`, and `unsubscribe_token`). The `academy` parameter specifically was confirmed exploitable using a fully unicode-escaped `confirm` function name inside an `<img onerror>` handler, bypassing the same function-name filter seen on other subdomains.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/settings/?academy=`

*(Note: parameter discovery indicates this endpoint reflects any of 34+ arbitrary parameter names - see Impact/Recommendations for scope implications.)*

## Steps to Reproduce
1. Discover reflecting parameters using a parameter-mining tool:
   ```
   x8 -u https://kzlabs.in/subdomains/settings/ -X GET --disable-progress-bar --reflected-only -w params.txt
   ```
   This returned 34 reflecting parameter names, including `academy`.
2. Navigate to the following URL:
   `https://kzlabs.in/subdomains/settings/?academy=tanishq%27%22%3E%3CImg%20SrC=x%20OnErrOr=\u0063\u006f\u006e\u0066\u0069\u0072\u006d(1)%3E`
3. Observe that a JavaScript confirm box pops up displaying `1` - confirming the fully unicode-escaped function name executed via the image `onerror` handler.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=\u0063\u006f\u006e\u0066\u0069\u0072\u006d(1)>
```

## Proof of Concept Request
<img width="1917" height="1037" alt="Screenshot 2026-09-08 064611" src="https://github.com/user-attachments/assets/2b677b66-8d52-4db1-b7a7-27cbbc2e3422" />
<img width="1917" height="1032" alt="Screenshot 2026-09-08 064603" src="https://github.com/user-attachments/assets/5f204402-3442-4342-896e-d51a67ec2d31" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. More significantly, this endpoint reflects an unusually large set of arbitrary parameter names (34 identified via parameter mining), which suggests the reflection is not tied to specific expected form fields but to any unrecognized GET parameter - a pattern often seen with debug/error-reporting code or a generic "echo unknown params" handler left enabled. Several of the reflecting parameter names (`password`, `token`, `csrf_token`, `api_key`, `unsubscribe_token`) suggest genuinely sensitive data may pass through this same code path elsewhere in the application, which would warrant separate, careful investigation.

## Recommendations for Fix
Apply context-aware output encoding on all reflected parameters on this endpoint (e.g. `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')` in PHP), rather than fixing `academy` in isolation, since the underlying issue reflects arbitrary parameter names generically. Identify and remove or properly guard whatever generic "echo any unrecognized parameter" logic is causing 34+ parameter names to reflect on a single page, as this is unusual behavior worth investigating at the code level rather than patching per-parameter. Add a strict Content-Security-Policy header as defense-in-depth.
