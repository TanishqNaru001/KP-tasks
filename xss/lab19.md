## Title
Reflected Cross-Site Scripting (XSS) - Page Heading
## Vulnerability Type
Reflected XSS

## Summary
The docs page on `https://kzlabs.in/subdomains/docs/` reflects the `q` query parameter directly into the page's HTML response (in the page heading) without proper sanitization or output encoding. This allows an attacker to inject arbitrary HTML/JavaScript that executes in the victim's browser when they visit a crafted URL. As with the earlier `q`/`term` findings (RefSeek, PubMed), this uses an `<img onerror>` payload.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/docs/?q=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/docs/?q=tanishq%27%22%3E%3CImg%20SrC=x%20OnErrOr=confirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed in the page context via the image `onerror` event handler.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1038" alt="Screenshot 2026-09-08 060854" src="https://github.com/user-attachments/assets/5839ad4a-e341-43e5-a0e4-af81c24cf808" />
<img width="1917" height="1035" alt="Screenshot 2026-09-08 060848" src="https://github.com/user-attachments/assets/1e52011c-8478-4774-9b17-96676c6ff561" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `q` parameter (e.g. `htmlspecialchars($q, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page heading, and add a Content-Security-Policy header as defense-in-depth. This is another instance of the same `q`/`term`-parameter reflection pattern seen on RefSeek and PubMed — likely worth fixing centrally rather than per-subdomain.
