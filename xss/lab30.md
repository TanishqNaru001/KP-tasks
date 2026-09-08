## Title
Reflected Cross-Site Scripting (XSS) - Case-Insensitive Filter Bypass
## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The assets page on `https://kzlabs.in/subdomains/assets/` reflects the `p` query parameter directly into the page's HTML response (inside the search bar) without proper sanitization or output encoding. As with other subdomains, the endpoint's filtering does not account for mixed-case tag and attribute names (`IMg`, `SrC`, `OnErroR`), confirming the filter performs case-sensitive string matching rather than proper output encoding.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/assets/?p=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/assets/?p=tanishq%27%22%3E%3CIMg+SrC%3Dx+OnErroR%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload executed via the image `onerror` event handler, breaking out of the search bar's input value context.

## Payload Used
```
tanishq'"><IMg SrC=x OnErroR=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1027" alt="Screenshot 2026-09-08 072333" src="https://github.com/user-attachments/assets/81b320a1-f880-4136-9b1a-f0ea159e8935" />
<img width="1917" height="1030" alt="Screenshot 2026-09-08 072325" src="https://github.com/user-attachments/assets/2a5414c7-7efd-4d6f-94b0-36b423896af3" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `p` parameter (e.g. `htmlspecialchars($p, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the search bar's value attribute, and add a Content-Security-Policy header as defense-in-depth. Case-sensitive tag/attribute blocklisting should be replaced entirely with proper encoding, since it is trivially bypassed via mixed-case variations.
