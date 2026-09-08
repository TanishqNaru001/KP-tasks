## Title
Reflected Cross-Site Scripting (XSS) - Category Filter Bypass 

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The shop page on `https://kzlabs.in/subdomains/shop/index.php` reflects both the `category` and `sort` query parameters directly into the page's HTML response without proper sanitization. As with other subdomains, the endpoint's filtering does not account for mixed-case tag and attribute names (`ImG`, `Src`, `OnERRor`), confirming the filter performs case-sensitive string matching rather than proper output encoding, and that the issue affects multiple parameters on this endpoint simultaneously.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/shop/index.php?category=&sort=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/shop/index.php?category=tanishq%27%22%3E%3CImG%20Src=x%20OnERRor=confirm(1)%3E&sort=tanishq%27%22%3E%3CImG%20Src=x%20OnERRor=confirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload executed via the image `onerror` event handler, in both the `category` and `sort` reflection contexts.

## Payload Used
```
tanishq'"><ImG Src=x OnERRor=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1036" alt="Screenshot 2026-09-08 071420" src="https://github.com/user-attachments/assets/bb26c9f9-79f0-4fca-a62e-13f1d704e263" />
<img width="1917" height="1031" alt="Screenshot 2026-09-08 071414" src="https://github.com/user-attachments/assets/7cd28e4e-4fc1-4b23-842d-8fafd66f9440" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. Both `category` and `sort` being independently exploitable with the identical bypass suggests the shared filtering component is applied uniformly across parameters on this endpoint, rather than the vulnerability being specific to one field.

## Recommendations for Fix
Apply context-aware output encoding on both the `category` and `sort` parameters (e.g. `htmlspecialchars($category, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting them into the page, and add a Content-Security-Policy header as defense-in-depth. Case-sensitive tag/attribute blocklisting should be replaced entirely with proper encoding, since it is trivially bypassed via mixed case, and the fix should be applied to all parameters on this endpoint rather than just the two confirmed here.
