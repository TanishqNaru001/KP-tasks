## Title
Reflected Cross-Site Scripting (XSS) - Function Name Filter Bypass

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The checkout page on `https://kzlabs.in/subdomains/checkout/` reflects the `cat` query parameter directly into the page's HTML response without proper sanitization. As with the profile, account, and tickets portals, this endpoint (or a shared upstream filter) attempts to block specific JavaScript function names such as `confirm`, but this is bypassed by encoding a single character of the function name as a JavaScript unicode escape (`\u0063onfirm` instead of `confirm`), which the browser's JS engine still parses and executes identically. This confirms the same function-name-blocklist weakness extends beyond the `fname`/`lname` parameter pair to other parameters (`cat`) as well.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/checkout/?cat=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/checkout/?cat=tanishq%27%22%3E%3CImg%20SrC=x%20OnErrOr=\u0063onfirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming that the `\u0063onfirm` unicode-escaped function name in the `cat` value executed identically to the literal `confirm`, bypassing the function-name string filter in place.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=\u0063onfirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1037" alt="Screenshot 2026-09-08 062922" src="https://github.com/user-attachments/assets/f01c1789-32a9-4cf8-bd3a-0de4bd57e6f2" />
<img width="1917" height="1027" alt="Screenshot 2026-09-08 062908" src="https://github.com/user-attachments/assets/b27c8853-11de-48fb-839a-0f2f5413ebd3" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This is now the fourth confirmed instance (profile, account, tickets, checkout) of the identical unicode-escape function-name-filter bypass, and the first seen on a parameter other than `lname` — confirming the shared filtering component is applied broadly across different parameter names, not just `fname`/`lname` pairs.

## Recommendations for Fix
Do not attempt to blocklist specific function names or keywords (`alert`, `confirm`, `eval`, etc.) as a mitigation - this is fundamentally bypassable via unicode escapes, string concatenation, and numerous other JavaScript obfuscation techniques. Instead, apply context-aware output encoding on the `cat` parameter (e.g. `htmlspecialchars($cat, ENT_QUOTES, 'UTF-8')` in PHP) so that user input can never break out of its intended HTML context in the first place. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, and apply the fix centrally to the shared filtering component across all parameters and subdomains rather than patching each one individually.
