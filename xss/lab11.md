## Title
Reflected Cross-Site Scripting (XSS) — Filter Evasion on Feedback Portal
## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The feedback form on `https://kzlabs.in/subdomains/feedback/` reflects the `fname` and `lname` query parameters directly into the page's HTML response. This endpoint appears to block or strip `<script>` tags, but does not filter other HTML event-handler vectors. Using an `<img onerror>` payload instead of `<script>` bypasses the existing filter and achieves script execution, confirming the filtering logic is a denylist rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/feedback/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/feedback/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed via the image `onerror` event handler, bypassing whatever `<script>` filtering is in place.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1915" height="1040" alt="image" src="https://github.com/user-attachments/assets/72392734-fa1e-493b-9a1d-469aceddfe2d" />
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/56568df8-d85c-4961-869e-67d91e8cc212" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. The fact that an existing `<script>` filter is bypassable also indicates the current mitigation approach is fundamentally insufficient, not just incomplete.

## Recommendations for Fix
Replace denylist-based filtering (blocking specific tags like `<script>`) with proper context-aware output encoding on both the `fname` and `lname` parameters (e.g. `htmlspecialchars($fname, ENT_QUOTES, 'UTF-8')` in PHP), which neutralizes all HTML metacharacters regardless of tag or attribute used. Add a Content-Security-Policy header as defense-in-depth so that even if encoding is missed somewhere, inline script/event-handler execution is blocked by the browser.
