## Title
Reflected Cross-Site Scripting (XSS) - Case-Insensitive Filter Bypass

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The directory page on `https://kzlabs.in/subdomains/directory/` reflects both the `fname` and `lname` query parameters directly into the page's HTML response. As with the feedback, cookbook, and support subdomains, this endpoint's filtering is bypassed using mixed-case tag/attribute names (`Img`, `SrC`, `OnErrOr`), confirming the filter performs a case-sensitive string match rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/directory/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/directory/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed via the image `onerror` event handler in both the `fname` and `lname` contexts, bypassing the existing case-sensitive filter.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request

<img width="1917" height="1038" alt="Screenshot 2026-09-08 054838" src="https://github.com/user-attachments/assets/e7742567-7cd1-4112-99d6-65c3c2f6a1f6" />
<img width="1917" height="1038" alt="Screenshot 2026-09-08 054848" src="https://github.com/user-attachments/assets/e86e275a-17bd-44a3-87a7-3d07022e61a5" />

*(attach screenshot here)*

## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This is now the fourth confirmed instance (feedback, cookbook, support, directory) of the identical case-sensitive filter-bypass pattern, further confirming the root cause is a single shared, systemically flawed filtering component reused across many subdomains.

## Recommendations for Fix
Replace denylist-based, case-sensitive tag filtering with proper context-aware output encoding on both the `fname` and `lname` parameters (e.g. `htmlspecialchars($fname, ENT_QUOTES, 'UTF-8')` in PHP), which is inherently case- and tag-agnostic. Add a Content-Security-Policy header as defense-in-depth, and treat this as one root-cause fix to apply centrally across the shared component rather than patching each subdomain individually.
