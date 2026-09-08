## Title
Reflected Cross-Site Scripting (XSS) - Script & Img Tag Filter Evasion

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The support page on `https://kzlabs.in/subdomains/support/` reflects both the `fname` and `lname` query parameters directly into the page's HTML response. This endpoint appears to filter `<script>` tags, but the `<img onerror>` vector still succeeds — confirming the filtering logic targets specific tag names rather than performing proper output encoding, and that even `<img>`-aware filtering (if present) can be bypassed via case variation or attribute placement.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/support/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/support/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed via the image `onerror` event handler in both the `fname` and `lname` contexts, bypassing the existing tag filter.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1032" alt="Screenshot 2026-09-08 054601" src="https://github.com/user-attachments/assets/3a364595-0477-40db-96ba-cffd350aa915" />
<img width="1917" height="1033" alt="Screenshot 2026-09-08 054612" src="https://github.com/user-attachments/assets/0a72b14a-3f85-493a-b788-5696e747a512" />

*(attach screenshot here)*

## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This is now the third confirmed instance (feedback, cookbook, support) of the same filter-bypass pattern, reinforcing that the root cause is a shared, systemically flawed filtering component rather than an isolated per-page issue.

## Recommendations for Fix
Replace denylist-based filtering (blocking specific tags like `<script>` or `<img>`) with proper context-aware output encoding on both the `fname` and `lname` parameters (e.g. `htmlspecialchars($fname, ENT_QUOTES, 'UTF-8')` in PHP), which neutralizes all HTML metacharacters regardless of tag or attribute used. Add a Content-Security-Policy header as defense-in-depth, and prioritize auditing the shared component behind these parameters since patching individual subdomains will not resolve the underlying issue.
