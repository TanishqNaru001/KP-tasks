## Title
Reflected Cross-Site Scripting (XSS) - Path-Based Injection Between Route Segments

## Vulnerability Type
Reflected XSS (Path-Based Injection)

## Summary
The media portal at `https://kzlabs.in/subdomains/media/index.php` reflects a URL path segment directly into the page's HTML response without proper sanitization or output encoding. Unlike query-parameter-based reflections, the injection point here sits between two route segments (`/account/` and `/messages`), meaning the application reads and reflects the middle path component (likely a username or account identifier) without encoding it. As with other subdomains, the case-sensitive filter is bypassed via mixed-case tag and attribute names (`ImG`, `SrC`, `OnErroR`).

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/media/index.php/account/{INJECT}/messages`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/media/index.php/account/tanishq%27%22%3E%3CImG%20SrC=x%20OnErroR=confirm(1)%3E/messages`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload in the middle path segment executed via the image `onerror` event handler.

## Payload Used
```
tanishq'"><ImG SrC=x OnErroR=confirm(1)>
```
Injected as: `/account/{payload}/messages`

## Proof of Concept Request
<img width="1917" height="1031" alt="Screenshot 2026-09-08 073124" src="https://github.com/user-attachments/assets/4a90bcea-8b42-42df-b93a-63eccaa3ed70" />
<img width="1917" height="1032" alt="Screenshot 2026-09-08 073117" src="https://github.com/user-attachments/assets/f29c88db-b6e1-46ad-acc5-93b339165394" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. Path-segment injection is distinct from query-parameter injection in two important ways: first, it suggests the routing/dispatch layer itself is reflecting unencoded path components rather than a specific form input handler; second, URLs with path-based payloads can appear more legitimate to victims since the malicious content is embedded within what looks like a normal URL structure (`/account/.../messages`) rather than an obvious query string.

## Recommendations for Fix
Apply context-aware output encoding (e.g. `htmlspecialchars($segment, ENT_QUOTES, 'UTF-8')` in PHP) to any path segment before reflecting it into the HTML response - specifically the account identifier segment extracted from the `/account/{id}/messages` route. This is a distinct code path from query-string parameter handling, so patching `$_GET` parameters alone will not fix this. Audit the routing layer for any other route patterns that extract and reflect path segments, and add a Content-Security-Policy header as defense-in-depth.
