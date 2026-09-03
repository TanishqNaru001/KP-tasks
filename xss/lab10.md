## Title
Reflected Cross-Site Scripting (XSS) — Global Site Search Portal

## Vulnerability Type
Reflected XSS

## Summary
The search functionality on `https://kzlabs.in/subdomains/search/` reflects the `fname` and `lname` query parameters directly into the page's HTML response without proper sanitization or output encoding. This allows an attacker to inject arbitrary HTML/JavaScript that executes in the victim's browser when they visit a crafted URL. This instance uses an `<img onerror>` based payload, confirming that the application is not filtering event-handler attributes either.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/search/?fname=&lname=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/search/?fname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E&lname=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed in the page context via the image `onerror` event handler.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1047" alt="image" src="https://github.com/user-attachments/assets/b9cdf6ea-63b8-46a2-9a86-40aa4a0d3753" />
<img width="1917" height="1037" alt="image" src="https://github.com/user-attachments/assets/68e91cc3-2821-4f2f-b611-ddf3f7bbe6cd" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on both the `fname` and `lname` parameters (e.g. `htmlspecialchars($fname, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting them into the page, and add a Content-Security-Policy header as defense-in-depth. Note that this payload used an `<img onerror>` vector rather than `<script>` tags, so any fix relying solely on stripping `<script>` tags will remain bypassable — output encoding of all HTML metacharacters is required regardless of tag/attribute used.
