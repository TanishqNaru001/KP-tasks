## Title
Reflected Cross-Site Scripting (XSS) -  Search Filter Bypass
## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The knowledge base search page on `https://kzlabs.in/subdomains/kb/` reflects the `search` query parameter directly into the page's HTML response, inside the search bar's value attribute. As with other subdomains, the endpoint's filtering does not account for mixed-case tag and attribute names (`ImG`, `SrC`, `OnERRor`), confirming the filter performs case-sensitive string matching rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/kb/?search=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/kb/?search=tanishq%27%22%3E%3CImG+SrC%3Dx+OnERRor%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload executed via the image `onerror` event handler after breaking out of the search bar's input value context.

## Payload Used
```
tanishq'"><ImG SrC=x OnERRor=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1036" alt="Screenshot 2026-09-08 071019" src="https://github.com/user-attachments/assets/9de8db07-e448-490e-b46b-524c0108f63d" />
<img width="1917" height="1037" alt="Screenshot 2026-09-08 071007" src="https://github.com/user-attachments/assets/a24f2299-6458-4cb6-a8fe-30b364e147e0" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `search` parameter (e.g. `htmlspecialchars($search, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the search bar's value attribute, and add a Content-Security-Policy header as defense-in-depth. Case-sensitive tag/attribute blocklisting should be replaced entirely with proper encoding, since it is trivially bypassed via mixed case.
