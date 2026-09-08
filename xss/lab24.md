## Title
Reflected Cross-Site Scripting (XSS) - Full Unicode-Escaped Script Encoding Bypass 

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The mail page on `https://kzlabs.in/subdomains/mail/` reflects the `id` query parameter directly into the page's HTML response without proper sanitization. Unlike prior findings that used an `<img onerror>` vector with a single unicode-escaped character, this payload uses a literal `<script>` tag (in mixed case, `<ScRiPT>`) with the entire function name `confirm` unicode-escaped character-by-character (`\u0063\u006f\u006e\u0066\u0069\u0072\u006d`). This confirms the filtering component neither reliably blocks case-varied `<script>` tags nor fully-escaped function names, indicating the underlying protection is a shallow, easily-evaded string/pattern filter rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/mail/?id=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/mail/?id=tanishq%27%22%3E%3CScRiPT%3E\u0063\u006f\u006e\u0066\u0069\u0072\u006d(1)%3C/ScRiPT%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming that both the case-varied `<ScRiPT>` tag and the fully unicode-escaped `confirm` function name executed successfully, bypassing the filter on two fronts simultaneously.

## Payload Used
```
tanishq'"><ScRiPT>\u0063\u006f\u006e\u0066\u0069\u0072\u006d(1)</ScRiPT>
```

## Proof of Concept Request
<img width="1917" height="1033" alt="Screenshot 2026-09-08 063618" src="https://github.com/user-attachments/assets/023fbf10-fc1e-4e78-8670-c01c013804b3" />
<img width="1917" height="1037" alt="Screenshot 2026-09-08 063612" src="https://github.com/user-attachments/assets/62afb33f-9cc4-47a7-9c76-b2b724e706cb" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This finding is more severe than the earlier filter-bypass instances in demonstrating that the filter fails against a combined bypass (case variation + full unicode escaping of a `<script>` tag itself, not just an event handler), reinforcing that the mitigation approach across this application is fundamentally broken rather than missing edge cases.

## Recommendations for Fix
Do not attempt to blocklist specific tags, case variants, or function names/keywords as a mitigation — this is fundamentally bypassable via unicode escapes, mixed case, string concatenation, and numerous other encoding techniques, as demonstrated here on both the tag and the function name simultaneously. Instead, apply context-aware output encoding on the `id` parameter (e.g. `htmlspecialchars($id, ENT_QUOTES, 'UTF-8')` in PHP) so that user input can never break out of its intended HTML context in the first place, regardless of case or encoding tricks. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, since this would block inline script execution even if an encoding gap is later reintroduced.
