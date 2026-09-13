## Title
DOM-Based Cross-Site Scripting (XSS) via URL Fragment (`location.hash`) - MilesWeb Clone

## Vulnerability Type
DOM-Based XSS (URL Fragment / Hash-Based)

## Summary
The MilesWeb clone at `https://kzlabs.in/subdomains/milesweb/` is vulnerable to DOM-based XSS via the URL fragment (`#`). Client-side JavaScript reads the `location.hash` value (intended to scroll to a named section like `#plansSection`) and writes it to the DOM unsafely without sanitization. Injecting an XSS payload after `#` instead of a valid section name causes the payload to execute in the victim's browser. This is a textbook URL fragment DOM XSS — the `#` and everything after it is **never sent to the server**, making this 100% client-side and completely invisible to server-side WAFs, logs, and output encoding patches.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/milesweb/#plansSection`

**Injection point:** URL fragment (`location.hash`) — everything after `#`

**Vulnerable sink:** Client-side JavaScript reading `location.hash` and writing it to the DOM (likely `innerHTML`, `document.write`, or `insertAdjacentHTML`)

## Steps to Reproduce
1. Navigate to the following URL with the payload injected in the URL fragment:
   ```
   https://kzlabs.in/subdomains/milesweb/#'"><SCripT>confirm(1)</SCripT>
   ```
2. Observe that a JavaScript confirm box pops up - confirming the `location.hash` value was read by client-side JS and written to the DOM unsafely.

## Payload Used
```
'"><SCripT>confirm(1)</SCripT>
```
Injected via: `https://kzlabs.in/subdomains/milesweb/#'"><SCripT>confirm(1)</SCripT>`

## Why This Is Definitively DOM XSS
Unlike the other DOM XSS findings (IMDb, AniList) where page source verification is needed to confirm, this finding is **inherently and provably DOM XSS** by nature:

- The URL fragment (`#` and everything after it) is **never included in the HTTP request** sent to the server - it exists only in the browser.
- The server never sees `#plansSection` or the payload - it cannot reflect what it never received.
- Any execution of this payload is therefore 100% client-side, by definition.
- No page source verification is needed - the architecture of HTTP proves this is DOM XSS.

## Proof of Concept

<img width="1917" height="1033" alt="image" src="https://github.com/user-attachments/assets/69c5e234-4568-4e49-9e22-e69dca41e256" />
<img width="1917" height="1042" alt="image" src="https://github.com/user-attachments/assets/7adebad7-d999-428e-aaef-4cd7ba710267" />


## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser simply by sharing a crafted URL:

- Steal session cookies and perform account takeover.
- Redirect victims to phishing or malware pages.
- Perform unauthorized actions on behalf of the victim.
- **No server interaction required** - the payload never touches the server, making it undetectable via server-side logging, WAF rules, or IDS/IPS systems.
- Particularly effective as a phishing vector since the URL appears to point to a legitimate anchor section (`#plansSection` replaced with payload) on a trusted domain.

## Recommendations for Fix
1. Never use `location.hash` directly in unsafe DOM sinks. Replace:
   ```javascript
   // Unsafe
   document.getElementById('section').innerHTML = location.hash.substring(1);

   // Safe — for plain text
   document.getElementById('section').textContent = location.hash.substring(1);
   ```
2. If the hash value is used to scroll to a named section, validate it against an allowlist of known section IDs before using it:
   ```javascript
   const allowedSections = ['plansSection', 'featuresSection', 'pricingSection'];
   const hash = location.hash.substring(1);
   if (allowedSections.includes(hash)) {
       document.getElementById(hash)?.scrollIntoView();
   }
   ```
3. If HTML rendering from the hash is genuinely required, sanitize using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(location.hash.substring(1));
   ```
4. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth - this blocks inline script execution even if hash-based payloads reach a DOM sink.
