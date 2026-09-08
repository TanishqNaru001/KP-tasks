## Title
Reflected Cross-Site Scripting (XSS) - Case-Insensitive Filter Bypass

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The redirect handler at `https://kzlabs.in/subdomains/go/` reflects the `returnTo` query parameter directly into the page's HTML response without proper sanitization or output encoding. As with other subdomains, the endpoint's filtering does not account for mixed-case tag and attribute names (`ImG`, `SrC`, `OnErroR`), confirming the filter performs case-sensitive string matching rather than proper output encoding. The `returnTo` parameter is particularly noteworthy as it is typically used to redirect users after authentication or navigation flows - making it a high-value social engineering target since a crafted URL carrying this parameter would appear more legitimate to a victim.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/go/?returnTo=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/go/?returnTo=tanishq%27%22%3E%3CImG%20SrC=x%20OnErroR=confirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload executed via the image `onerror` event handler, breaking out of the `returnTo` value's reflection context.

## Payload Used
```
tanishq'"><ImG SrC=x OnErroR=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1031" alt="Screenshot 2026-09-08 073124" src="https://github.com/user-attachments/assets/bc954e8d-8816-4edf-a0fb-65edad12504c" />
<img width="1917" height="1032" alt="Screenshot 2026-09-08 073117" src="https://github.com/user-attachments/assets/364a8026-1653-423d-a86b-5d2fc1ee5d58" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. The `returnTo` parameter context increases real-world exploitability: victims are more likely to trust and click a URL from a domain they recognise that appears to be part of a normal login/redirect flow. Additionally worth investigating whether this parameter is also vulnerable to open redirect, as `returnTo` parameters commonly control post-action navigation.

## Recommendations for Fix
Apply context-aware output encoding on the `returnTo` parameter (e.g. `htmlspecialchars($returnTo, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page, and validate that its value is a permitted relative path or allowlisted URL rather than accepting arbitrary input - this simultaneously addresses both the XSS and any potential open redirect risk. Add a Content-Security-Policy header as defense-in-depth.
