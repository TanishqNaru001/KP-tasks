## Title
Reflected Cross-Site Scripting (XSS)— Multi-Parameter Filter Evasion on Cookbook Portal
## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The cookbook page on `https://kzlabs.in/subdomains/cookbook/` reflects both the `fname` and `lname` query parameters directly into the page's HTML response. As with the feedback portal, this endpoint appears to filter `<script>` tags but not other HTML event-handler vectors, and the bypass is reproducible across both parameters simultaneously — confirming the underlying filtering logic is shared and consistently denylist-based rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/cookbook/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/cookbook/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed via the image `onerror` event handler in both the `fname` and `lname` contexts, bypassing whatever `<script>` filtering is in place.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/de73eb52-1be7-41e9-bc9e-603537dec75c" />
<img width="1917" height="1031" alt="image" src="https://github.com/user-attachments/assets/cf9ba4d6-528e-44ba-a2bb-ccd7797584f0" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. That the same `<img onerror>` bypass works identically on both `fname` and `lname` across multiple subdomains indicates a shared, systemically flawed filtering component rather than isolated per-page bugs.

## Recommendations for Fix
Replace denylist-based filtering (blocking specific tags like `<script>`) with proper context-aware output encoding on both the `fname` and `lname` parameters (e.g. `htmlspecialchars($fname, ENT_QUOTES, 'UTF-8')` in PHP), which neutralizes all HTML metacharacters regardless of tag or attribute used. Add a Content-Security-Policy header as defense-in-depth, and audit whether other subdomains share this same filtering component so the fix can be applied centrally rather than page-by-page.
