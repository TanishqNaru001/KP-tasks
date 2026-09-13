## Title
DOM-Based Cross-Site Scripting (XSS) via Bare URL Fragment - Gallery Portal / ForeScout Clone

## Vulnerability Type
DOM-Based XSS (URL Fragment / Hash-Based)


## Summary
The ForeScout clone gallery page at `https://kzlabs.in/subdomains/gallery/` is vulnerable to DOM-based XSS via the URL fragment. Unlike the Careers and Wallet findings where the payload was appended after a legitimate anchor name (`#open-roles`, `#send-transaction`), here the payload is injected directly after a bare `#` with no preceding anchor name - confirming that `location.hash` is read and written to the DOM unsafely regardless of whether a valid anchor value precedes the payload. The mixed-case `<SCripT>` tag confirms the client-side filter is case-sensitive and trivially bypassed.

## Vulnerable Endpoint
**Legitimate URL:** `https://kzlabs.in/subdomains/gallery/#`

**Malicious URL:** `https://kzlabs.in/subdomains/gallery/#'"><SCripT>confirm(1)</SCripT>`

**Injection point:** URL fragment (`location.hash`) - directly after bare `#`

## Steps to Reproduce
1. Navigate to the legitimate gallery page:
   `https://kzlabs.in/subdomains/gallery/#`
2. Now navigate to the crafted URL with the payload injected directly after `#`:
   `https://kzlabs.in/subdomains/gallery/#'%22%3E%3CSCripT%3Econfirm(1)%3C/SCripT%3E`
3. Observe that a JavaScript confirm box pops up - confirming the bare `location.hash` value was read by client-side JS and written to the DOM unsafely.

## Payload Used
```
'"><SCripT>confirm(1)</SCripT>
```
Injected as: `#'"><SCripT>confirm(1)</SCripT>`

## Why This Is Definitively DOM XSS
- The URL fragment (`#` and everything after it) is **never sent to the server** - defined HTTP behaviour.
- The server only receives `GET /subdomains/gallery/` - it never sees the `#` value or the payload.
- The bare `#` (no preceding anchor name) confirms the JS reads the raw `location.hash` value without first validating it against a set of known anchor names - the entire hash content is passed directly to the sink.

## Proof of Concept

<img width="1917" height="1041" alt="image" src="https://github.com/user-attachments/assets/230e4ad1-c866-4d20-89f8-779e815111fb" />
<img width="1917" height="1037" alt="image" src="https://github.com/user-attachments/assets/ab6e5a62-64c8-452b-8cb5-562cf20f77f9" />


## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser simply by sharing a crafted URL:

- Steal session cookies and perform account takeover.
- Redirect victims to phishing or malware pages.
- Perform unauthorized actions on behalf of the victim.
- The bare `#` injection (no anchor name required) is a broader attack surface than anchor-appended variants - any URL on this page with a `#` can carry the payload, not just specific named anchors.
- **Server-side defences are completely ineffective** - WAF rules, IDS/IPS, server logs, and backend encoding patches provide zero protection since the payload never reaches the server.

## Recommendations for Fix
1. Never use `location.hash` directly in unsafe DOM sinks. If the hash is only used for scroll/navigation, validate it against an allowlist before use:
   ```javascript
   const allowedAnchors = ['section1', 'gallery', 'contact'];
   const hash = location.hash.substring(1);
   if (allowedAnchors.includes(hash)) {
       document.getElementById(hash)?.scrollIntoView();
   }
   ```
2. If the hash must be rendered as HTML, sanitize using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(location.hash.substring(1));
   ```
3. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth.
4. The bare `#` vulnerability (no required anchor prefix) means this page has a wider attack surface than anchor-specific findings — audit all `location.hash` usage in the page's JS files to ensure no other sinks exist.
