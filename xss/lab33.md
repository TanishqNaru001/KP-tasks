## Title
Reflected Cross-Site Scripting (XSS) - Deep API Route Path Injection

## Vulnerability Type
Reflected XSS (Path-Based Injection)

## Summary
The widgets portal at `https://kzlabs.in/subdomains/widgets/index.php` reflects a URL path segment directly into the page's HTML response without proper sanitization or output encoding. The injection point sits deep within a multi-segment REST-style API route that mimics a Reddit API structure (`/svc/shreddit/api/comments/askreddit/{INJECT}/messages/t1_i5sxroa`), where a subreddit post identifier or thread slug is reflected unsanitized. As with other subdomains, the case-sensitive filter is bypassed via mixed-case tag and attribute names (`ImG`, `SrC`, `OnErroR`). The depth and apparent legitimacy of the surrounding URL structure makes this a particularly convincing phishing vector.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/{INJECT}/messages/t1_i5sxroa`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/tanishq%27%22%3E%3CImG%20SrC=x%20OnErroR=confirm(1)%3E/messages/t1_i5sxroa`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the injected payload embedded within the deep API route path executed via the image `onerror` event handler.

## Payload Used
```
tanishq'"><ImG SrC=x OnErroR=confirm(1)>
```
Injected as: `/svc/shreddit/api/comments/askreddit/{payload}/messages/t1_i5sxroa`

## Proof of Concept Request
<img width="1917" height="1036" alt="Screenshot 2026-09-08 073442" src="https://github.com/user-attachments/assets/27d9761b-e303-475f-a7f8-8d55b90ab45b" />
<img width="1856" height="1037" alt="Screenshot 2026-09-08 073436" src="https://github.com/user-attachments/assets/a31878e5-2215-4a68-92b5-d234ccaf0faf" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. This instance is particularly effective as a social engineering vector: the surrounding multi-segment URL (`/svc/shreddit/api/comments/askreddit/.../messages/t1_i5sxroa`) closely mimics a legitimate Reddit API route, making the crafted URL appear highly credible to a victim who sees it in a link - the payload is buried within what looks like a normal thread/comment identifier. The path-based injection also confirms the routing layer reflects unsanitized path segments regardless of their depth within the route structure.

## Recommendations for Fix
Apply context-aware output encoding (e.g. `htmlspecialchars($segment, ENT_QUOTES, 'UTF-8')` in PHP) to every path segment extracted from the route before reflecting it into the HTML response - not just the first or last segment, but all intermediate ones. This is a distinct code path from query-string parameter handling, so patching `$_GET` parameters alone will not fix this. Audit all route patterns that extract and reflect path segments, paying particular attention to deep nested routes where individual segments may be overlooked, and add a Content-Security-Policy header as defense-in-depth.
