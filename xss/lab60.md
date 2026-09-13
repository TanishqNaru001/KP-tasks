## Title
DOM-Based Cross-Site Scripting (XSS) via URL Fragment - Lever Job Embed Anchor on Careers Portal

## Vulnerability Type
DOM-Based XSS (URL Fragment / Hash-Based)


## Summary
The Careers portal at `https://kzlabs.in/subdomains/careers/` is vulnerable to DOM-based XSS via the URL fragment. The page uses a Lever-style job listing embed (`?lever-apply#open-roles`) where client-side JavaScript reads `location.hash` to scroll to or render a named section. Appending an XSS payload directly after the legitimate anchor name (`#open-roles`) causes the payload to be read from `location.hash` and written to the DOM unsafely. As with the MilesWeb finding, the URL fragment is **never sent to the server** - this is inherently and provably client-side execution, invisible to all server-side logging and WAF rules.

## Vulnerable Endpoint
**Legitimate URL:** `https://kzlabs.in/subdomains/careers/?lever-apply#open-roles`

**Malicious URL:** `https://kzlabs.in/subdomains/careers/?lever-apply#open-roles'"><ImG Src=x OnErROr=confirm(1)>`

**Injection point:** URL fragment (`location.hash`) - everything after `#`

## Steps to Reproduce
1. Navigate to the legitimate careers page to observe normal behavior:
   `https://kzlabs.in/subdomains/careers/?lever-apply#open-roles`
2. Now navigate to the following crafted URL with the payload appended to the fragment:
   `https://kzlabs.in/subdomains/careers/?lever-apply#open-roles'"><ImG%20Src=x%20OnErROr=confirm(1)>`
3. Observe that a JavaScript confirm box pops up — confirming the `location.hash` value (including the appended payload) was read by client-side JS and written to the DOM unsafely.

## Payload Used
```
'"><ImG Src=x OnErROr=confirm(1)>
```
Appended to legitimate anchor: `#open-roles'"><ImG Src=x OnErROr=confirm(1)>`

## Why This Is Definitively DOM XSS
- The URL fragment (`#` and everything after it) is **never included in the HTTP request** to the server — this is defined HTTP behaviour.
- The server only receives `GET /?lever-apply` — it never sees `#open-roles` or the payload.
- Any execution of this payload is 100% client-side, by definition - no page source check needed.
- The Lever embed pattern (`?lever-apply#open-roles`) confirms the page's JS is actively reading and processing `location.hash` to handle anchor-based navigation, which is the exact source feeding the vulnerable sink.

## Proof of Concept
<img width="1917" height="1040" alt="image" src="https://github.com/user-attachments/assets/f360f02a-8ec0-4a99-8984-a3ed644e6345" />
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/92d3e165-33c5-4609-a114-e6c8df33bc55" />



## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser simply by sharing a crafted URL:

- Steal session cookies and perform account takeover.
- Redirect victims to phishing or malware pages.
- Perform unauthorized actions on behalf of the victim.
- **High phishing potential** - the URL looks highly credible (`/careers/?lever-apply#open-roles...`) mimicking a legitimate Lever-powered job listing URL that candidates routinely share and click, making social engineering trivial.
- **Server-side defences are completely ineffective** - WAF rules, IDS/IPS, server logs, and output encoding patches applied to the backend provide zero protection since the payload never reaches the server.

## Recommendations for Fix
1. Never use `location.hash` directly in unsafe DOM sinks. Validate the hash value against an allowlist of known anchor IDs before using it:
   ```javascript
   const allowedAnchors = ['open-roles', 'engineering', 'design', 'marketing'];
   const hash = location.hash.substring(1).split("'")[0]; // strip anything after valid chars
   if (allowedAnchors.includes(hash)) {
       document.getElementById(hash)?.scrollIntoView();
   }
   ```
2. If the hash value must be rendered as HTML, sanitize using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(location.hash.substring(1));
   ```
3. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth — blocks inline event-handler execution (`onerror`, `onclick`) even if hash-based payloads reach a DOM sink.
4. Audit all other anchor-based navigation logic on this page — any JS that reads `location.hash` and passes it to a rendering function is a potential DOM XSS source.
