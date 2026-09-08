## Title
Reflected Cross-Site Scripting (XSS) - Function Name Filter Bypass

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The account page on `https://kzlabs.in/subdomains/account/` reflects the `lname` query parameter directly into the page's HTML response. The `fname` parameter is not vulnerable - it appears to be sanitized correctly. As with the profile portal, this endpoint (or a shared upstream filter) attempts to block specific JavaScript function names such as `confirm`, but this is bypassed by encoding a single character of the function name as a JavaScript unicode escape (`\u0063onfirm` instead of `confirm`), which the browser's JS engine still parses and executes identically. This confirms the same function-name-blocklist weakness seen on the profile portal is shared across multiple subdomains.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/account/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL (the `fname` parameter is included only to match the form's expected fields; the vulnerability triggers via `lname`):
   `https://kzlabs.in/subdomains/account/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3D%5Cu0063onfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming that the `\u0063onfirm` unicode-escaped function name in the `lname` value executed identically to the literal `confirm`, bypassing the function-name string filter in place.

## Payload Used
```
lname: tanishq'"><Img SrC=x OnErrOr=\u0063onfirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1028" alt="Screenshot 2026-09-08 061925" src="https://github.com/user-attachments/assets/78440b68-b911-40a9-b39b-9140b145760b" />
<img width="1917" height="1033" alt="Screenshot 2026-09-08 061918" src="https://github.com/user-attachments/assets/e846d810-7c23-4598-8a2d-67919dab035c" />



## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This is now the second confirmed instance (profile, account) of the identical unicode-escape function-name-filter bypass on `lname`, further reinforcing that the root cause is a single shared, systemically flawed filtering component reused across subdomains rather than an isolated bug.

## Recommendations for Fix
Do not attempt to blocklist specific function names or keywords (`alert`, `confirm`, `eval`, etc.) as a mitigation — this is fundamentally bypassable via unicode escapes, string concatenation, and numerous other JavaScript obfuscation techniques. Instead, apply context-aware output encoding on the `lname` parameter (e.g. `htmlspecialchars($lname, ENT_QUOTES, 'UTF-8')` in PHP) so that user input can never break out of its intended HTML context in the first place. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, and apply the fix to the shared component centrally rather than per-subdomain, since `fname` is already handled correctly and can serve as the reference implementation.
