## Title
Reflected Cross-Site Scripting (XSS) — BigBasket Online Supermarket Catalog
## Vulnerability Type
Reflected XSS

## Summary
The search functionality on `https://kzlabs.in/subdomains/bigbasket/index.php` reflects the `q` query parameter directly into the page's HTML response without proper sanitization or output encoding. This allows an attacker to inject arbitrary JavaScript that executes in the victim's browser when they visit a crafted URL.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/bigbasket/index.php?q=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/bigbasket/index.php?q=tanishq%27%22%3E%3Cscript%3Econfirm%281%29%3C%2Fscript%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected script executed in the page context.

## Payload Used
```
tanishq'"><script>confirm(1)</script>
```

## Proof of Concept Request
<img width="1917" height="1040" alt="image" src="https://github.com/user-attachments/assets/73a8cf47-dba4-4949-80e1-a9a6ea06fac9" />
<img width="1916" height="1037" alt="image" src="https://github.com/user-attachments/assets/fed38077-78d3-4824-9af1-a1735f2a6393" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `q` parameter (e.g. `htmlspecialchars($q, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page, and add a Content-Security-Policy header as defense-in-depth. A WAF (e.g. Cloudflare) can help block common payloads as an additional layer.
