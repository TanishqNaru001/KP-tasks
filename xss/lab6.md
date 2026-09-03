## Title
Reflected Cross-Site Scripting (XSS) — GIGW Government Website Guidelines Portal

## Vulnerability Type
Reflected XSS

## Summary
The search functionality on `https://kzlabs.in/subdomains/guidelines/index.php` reflects the `s` query parameter directly into the page's HTML response without proper sanitization or output encoding. This allows an attacker to inject arbitrary JavaScript that executes in the victim's browser when they visit a crafted URL.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/guidelines/index.php?s=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/guidelines/index.php?s=tanishq%27%22%3E%3Cscript%3Econfirm(1)%3C%2Fscript%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected script executed in the page context.

## Payload Used
```
tanishq'"><script>confirm(1)</script>
```

## Proof of Concept Request
<img width="1917" height="1032" alt="Screenshot 2026-09-03 070554" src="https://github.com/user-attachments/assets/451d7801-5465-4634-ad76-04518606a6a2" />
<img width="1917" height="1041" alt="Screenshot 2026-09-03 070051" src="https://github.com/user-attachments/assets/8085a285-0e55-4713-87d7-df795575c8eb" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `s` parameter (e.g. `htmlspecialchars($s, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page, and add a Content-Security-Policy header as defense-in-depth. A WAF (e.g. Cloudflare) can help block common payloads as an additional layer.
