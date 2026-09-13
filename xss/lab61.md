## Title
DOM-Based Cross-Site Scripting (XSS) via URL Fragment - Send Transaction Anchor on Crypto Wallet Portal

## Vulnerability Type
DOM-Based XSS (URL Fragment / Hash-Based)


## Summary
The MyCrypto wallet clone at `https://kzlabs.in/subdomains/wallet/` is vulnerable to DOM-based XSS via the URL fragment. The page uses hash-based routing (`#send-transaction`) to navigate between wallet sections. Client-side JavaScript reads `location.hash` to determine which view to render and passes it to the DOM unsafely without sanitization. Appending an XSS payload to the `#send-transaction` anchor causes it to execute in the victim's browser. The cryptocurrency wallet context makes this finding significantly more impactful than other DOM XSS instances — a victim tricked into visiting a crafted wallet URL could have their session compromised at the exact moment they are initiating a financial transaction.

## Vulnerable Endpoint
**Legitimate URL:** `https://kzlabs.in/subdomains/wallet/#send-transaction`

**Malicious URL:** `https://kzlabs.in/subdomains/wallet/#send-transaction'"><ImG Src=x OnErROr=confirm(1)>`

**Injection point:** URL fragment (`location.hash`) — everything after `#`

## Steps to Reproduce
1. Navigate to the legitimate wallet page to observe normal behavior:
   `https://kzlabs.in/subdomains/wallet/#send-transaction`
2. Now navigate to the following crafted URL with the payload appended to the fragment:
   `https://kzlabs.in/subdomains/wallet/#send-transaction'%22%3E%3CImG%20Src=x%20OnErROr=confirm(1)%3E`
3. Observe that a JavaScript confirm box pops up - confirming the `location.hash` value was read by client-side JS and written to the DOM unsafely.

## Payload Used
```
'"><ImG Src=x OnErROr=confirm(1)>
```
Appended to legitimate anchor: `#send-transaction'"><ImG Src=x OnErROr=confirm(1)>`

## Why This Is Definitively DOM XSS
- The URL fragment (`#` and everything after it) is **never included in the HTTP request** to the server - this is defined HTTP behaviour.
- The server only receives `GET /subdomains/wallet/` - it never sees `#send-transaction` or the payload.
- Any execution of this payload is 100% client-side, by definition.
- The hash-based routing pattern (`#send-transaction`) confirms the page's JS actively reads and processes `location.hash` to render views - this is the exact source feeding the vulnerable sink.

## Proof of Concept

<img width="1897" height="1033" alt="image" src="https://github.com/user-attachments/assets/6aeafda6-3a24-423d-81a6-aa5fc99b0134" />
<img width="1917" height="1035" alt="image" src="https://github.com/user-attachments/assets/ec03a6d2-2a61-41c7-ba40-68846c225f9d" />


## Impact
This finding carries **elevated impact** compared to other DOM XSS instances due to the cryptocurrency wallet context:

- **Session hijacking at point of transaction** - a victim tricked into opening a crafted `#send-transaction` URL could have their wallet session stolen via `document.cookie` exfiltration at the exact moment they are initiating a crypto transfer.
- **Transaction manipulation** - injected JavaScript could silently modify transaction form values (recipient address, amount) after the page loads, redirecting funds to an attacker-controlled wallet address without the victim noticing.
- **Clipboard hijacking** - crypto wallet users frequently copy/paste wallet addresses; injected JS could intercept clipboard operations or replace a copied address with the attacker's.
- **Phishing at scale** - `#send-transaction` is a legitimate, recognisable wallet URL pattern that a victim would trust; the malicious payload is completely hidden in the fragment.
- **Server-side defences are completely ineffective** - WAF rules, IDS/IPS, server logs, and backend output encoding patches provide zero protection since the payload never reaches the server.

## Recommendations for Fix
1. Never use `location.hash` directly in unsafe DOM sinks. Validate the hash value against an allowlist of known route names before rendering:
   ```javascript
   const allowedRoutes = ['send-transaction', 'receive', 'history', 'settings'];
   const route = location.hash.substring(1).split("'")[0];
   if (allowedRoutes.includes(route)) {
       renderView(route);
   }
   ```
2. If hash content must be rendered as HTML, sanitize using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(location.hash.substring(1));
   ```
3. Implement a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) - blocks inline event-handler execution even if hash-based payloads reach a DOM sink.
4. For a financial application specifically, also implement Subresource Integrity (SRI) on all loaded scripts and restrict `connect-src` in the CSP to prevent exfiltration of session or transaction data to attacker-controlled endpoints.
