## Title
Reflected Cross-Site Scripting (XSS) - Path-Based Reflection

## Vulnerability Type
Reflected XSS (Path-Based / DOM or Server-Side Path Reflection)

## Summary
The `path-fetch` endpoint on `https://kzlabs.in/subdomains/path-fetch/` reflects the raw URL path segment (rather than a query parameter) directly into the page's HTML response without sanitization or output encoding. This differs from the other findings in this batch, which were all query-parameter based — here the injection point is the path itself, indicating the application reads and reflects `PATH_INFO` (or an equivalent routing segment) unsafely.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/path-fetch/`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/path-fetch/hello'%22%3E%3CImg%20SrC=x%20OnErrOr=confirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload, placed in the URL path rather than a query string, executed via the image `onerror` event handler.

## Payload Used
```
hello'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1917" height="1032" alt="Screenshot 2026-09-08 055554" src="https://github.com/user-attachments/assets/7f241e94-b079-4149-a963-2ee2e1e25d95" />
<img width="1917" height="1032" alt="Screenshot 2026-09-08 055617" src="https://github.com/user-attachments/assets/5f26add5-94df-4ea1-9115-48086600754b" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. Because the injection point is the path rather than a query parameter, this may indicate a distinct code path (e.g. custom routing or `PATH_INFO` handling) from the query-parameter-based findings, so it likely needs a separate fix rather than being covered by the same sanitization patch.

## Recommendations for Fix
Apply context-aware output encoding (e.g. `htmlspecialchars($pathSegment, ENT_QUOTES, 'UTF-8')` in PHP) to any path segment before reflecting it into the page, in addition to encoding query parameters. Add a Content-Security-Policy header as defense-in-depth. Audit the routing/dispatch logic specifically, since path-based reflection often lives in different code than query-string handling and may be missed if only `$_GET` parameters are patched.
