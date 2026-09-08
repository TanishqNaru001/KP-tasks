## Title
Reflected Cross-Site Scripting (XSS) - Function Name Filter Bypass via Unicode Escape 

## Vulnerability Type
Reflected XSS (Filter Bypass)

## Summary
The portal page on `https://kzlabs.in/subdomains/portal/` reflects the `cat` query parameter directly into the page's HTML response without proper sanitization. Parameter mining with `x8` identified 7 reflecting parameters on this endpoint (`cat`, `fname`, `id`, `lname`, `number`, `page`, `page.id`). The `cat` parameter was confirmed exploitable using a fully unicode-escaped `alert` function name inside an `<img onerror>` handler, demonstrating the same function-name-blocklist weakness seen on other subdomains extends to the `alert` function as well, not just `confirm`.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/portal/?cat=`

*(Note: `x8` parameter discovery indicates this endpoint also reflects `fname`, `id`, `lname`, `number`, `page`, and `page.id` - worth confirming exploitability on each rather than assuming.)*

## Steps to Reproduce
1. Discover reflecting parameters using a parameter-mining tool:
   ```
   x8 -u https://kzlabs.in/subdomains/portal/ -X GET --disable-progress-bar --reflected-only -w params.txt
   ```
   This returned 7 reflecting parameter names, including `cat`.
2. Navigate to the following URL:
   `https://kzlabs.in/subdomains/portal/?cat=tanishq%27%22%3E%3CImg%20SrC=x%20OnErrOr=\u0061\u006c\u0065\u0072\u0074(1)%3E`
3. Observe that a JavaScript alert box pops up displaying `1` - confirming the fully unicode-escaped `alert` function name executed via the image `onerror` handler, bypassing the function-name filter.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=\u0061\u006c\u0065\u0072\u0074(1)>
```

## Proof of Concept Request
<img width="1916" height="1028" alt="Screenshot 2026-09-08 070621" src="https://github.com/user-attachments/assets/11f3bda1-8b1c-4b8d-8d81-8d140b27765d" />
<img width="1917" height="1036" alt="Screenshot 2026-09-08 070614" src="https://github.com/user-attachments/assets/bce72e5e-5ff3-46fd-8474-21e794259ac8" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. This confirms the unicode-escape bypass works against multiple blocked function names (`confirm`, `alert`), not just one, meaning the underlying filter blocklists specific keywords individually rather than preventing script execution at a structural level — any function name not explicitly on the blocklist, or any blocklisted name obfuscated via unicode escaping, will bypass it.

## Recommendations for Fix
Do not attempt to blocklist specific function names or keywords (`alert`, `confirm`, `eval`, etc.) as a mitigation — this is fundamentally bypassable via unicode escapes and applies to any function name, not just the ones tested here. Instead, apply context-aware output encoding on the `cat` parameter (e.g. `htmlspecialchars($cat, ENT_QUOTES, 'UTF-8')` in PHP) so that user input can never break out of its intended HTML context in the first place. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, and audit the other reflecting parameters found on this endpoint (`fname`, `id`, `lname`, `number`, `page`, `page.id`) for the same issue.
