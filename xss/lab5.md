## Title
Reflected Cross-Site Scripting (XSS) via `q` Parameter — TutorialRepublic Search (kzlabs.in subdomain)

## Vulnerability Type
Reflected XSS

## Summary
The search functionality on `https://kzlabs.in/subdomains/tutorialrepublic/` reflects the `q` query parameter directly into the page's HTML response without proper sanitization or output encoding. This allows an attacker to inject arbitrary JavaScript that executes in the victim's browser when they visit a crafted URL.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/tutorialrepublic/?q=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/tutorialrepublic/?q=tanishq%27%22%3E%3Cscript%3Econfirm%281%29%3C%2Fscript%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected script executed in the page context.

## Payload Used
```
tanishq'"><script>confirm(1)</script>
```

## Proof of Concept Request
**Screenshot 1:** Payload injected into the search parameter — location where the input reflects unsanitized.

**Screenshot 2:** Confirm dialog box firing, showing successful script execution.

*(attach screenshots here)*

## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `q` parameter (e.g. `htmlspecialchars($q, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page, and add a Content-Security-Policy header as defense-in-depth. A WAF (e.g. Cloudflare) can help block common payloads as an additional layer.
