## Title
Reflected Cross-Site Scripting (XSS) - Function Name Filter Bypass 
 
## Vulnerability Type
Reflected XSS (Filter Bypass)
 
## Summary
The profile page on `https://kzlabs.in/subdomains/profile/` reflects the `lname` query parameter directly into the page's HTML response. The `fname` parameter is not vulnerable - it appears to be sanitized correctly. The `lname` parameter's payload demonstrates that in addition to tag/attribute filtering, this endpoint (or a shared upstream filter) also attempts to block specific JavaScript function names such as `confirm`. This is bypassed by encoding a single character of the function name as a JavaScript unicode escape (`\u0063onfirm` instead of `confirm`), which the browser's JS engine still parses and executes identically. This confirms the filtering relies on literal string matching of blocked function names rather than safe execution context prevention.
 
## Vulnerable Endpoint
`https://kzlabs.in/subdomains/profile/?fname=&lname=`
 
## Steps to Reproduce
1. Navigate to the following URL (the `fname` parameter is included only to match the form's expected fields; the vulnerability triggers via `lname`):
   `https://kzlabs.in/subdomains/profile/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3D%5Cu0063onfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming that the `\u0063onfirm` unicode-escaped function name in the `lname` value executed identically to the literal `confirm`, bypassing any function-name string filter in place.
## Payload Used
```
lname: tanishq'"><Img SrC=x OnErrOr=\u0063onfirm(1)>
```
 
## Proof of Concept Request
<img width="1917" height="1031" alt="Screenshot 2026-09-08 061426" src="https://github.com/user-attachments/assets/efc2ade6-5ef2-48d9-b00b-9fb4a1fe7301" />
<img width="1917" height="1031" alt="Screenshot 2026-09-08 061242" src="https://github.com/user-attachments/assets/918892ed-dd59-4c33-9edb-f7af6171ca2e" />

## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This finding is notable beyond the standard tag-filter bypasses: it shows that even if the application attempted to blocklist specific dangerous function names (e.g. `alert`, `confirm`, `eval`), that mitigation is trivially bypassable via JavaScript unicode escapes, meaning function-name blocklisting provides no real protection.
 
## Recommendations for Fix
Do not attempt to blocklist specific function names or keywords (`alert`, `confirm`, `eval`, etc.) as a mitigation - this is fundamentally bypassable via unicode escapes, string concatenation, and numerous other JavaScript obfuscation techniques. Instead, apply context-aware output encoding on the `lname` parameter (e.g. `htmlspecialchars($lname, ENT_QUOTES, 'UTF-8')` in PHP) so that user input can never break out of its intended HTML context in the first place. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, since that would block arbitrary inline script/event-handler execution regardless of function name obfuscation. It's also worth confirming why `fname` is correctly sanitized while `lname` is not, since that inconsistency suggests the encoding fix may have been applied inconsistently rather than centrally.
