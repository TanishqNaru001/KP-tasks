## Title
DOM-Based Cross-Site Scripting (XSS) via Search Parameter - Event Handler Injection on Censys Clone

## Vulnerability Type
DOM-Based XSS

## Summary
The Censys clone at `https://kzlabs.in/subdomains/censys/` is vulnerable to DOM-based XSS via the search bar. Client-side JavaScript reads the search input and writes it to the DOM unsafely. This instance is notable because the payload uses an `<img onerror>` event handler rather than a `<script>` tag — confirming the vulnerable DOM sink renders full HTML (not just script tags), meaning any HTML injection including event-handler attributes executes. This is the fourth DOM XSS instance found in this assessment, and the first to confirm event-handler execution within a DOM sink.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/censys/`

**Vulnerable sink:** Client-side JavaScript reading the search input and passing it to an HTML-rendering sink (`innerHTML`, `document.write`, or similar).

## Steps to Reproduce
1. Navigate to `https://kzlabs.in/subdomains/censys/`
2. In the search bar, enter the following payload:
   ```
   '"><ImG Src=x OnErROr=confirm(1)>
   ```
3. Submit the search or observe as-you-type.
4. Observe that a JavaScript confirm box pops up - confirming the payload was written to the DOM and the `onerror` event handler executed client-side.

## Payload Used
```
'"><ImG Src=x OnErROr=confirm(1)>
```
Mixed-case `ImG`, `Src`, `OnErROr` used to bypass any client-side string filter checking for lowercase tag/attribute names.

## How to Confirm It Is DOM XSS (Not Reflected)
1. Submit the payload and observe the confirm dialog firing.
2. **View raw page source** (`Ctrl+U`) - payload should **not** appear in the server's HTML response.
3. **DevTools → Elements tab** - payload **will** appear in the rendered DOM, injected by client-side JS.

## Proof of Concept

<img width="1917" height="1032" alt="image" src="https://github.com/user-attachments/assets/541f1254-1490-4bf9-af97-312578d71aad" />
<img width="1916" height="1032" alt="image" src="https://github.com/user-attachments/assets/120c0b84-a3b9-4e25-87d6-bf1c16377b7c" />
<img width="1917" height="1037" alt="image" src="https://github.com/user-attachments/assets/b339a338-40ae-4c63-a161-daeddc5d2f08" />



## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser by crafting a malicious URL or search input:

- Steal session cookies and perform account takeover.
- Redirect victims to phishing or malware pages.
- Perform unauthorized actions on behalf of the victim.
- Exfiltrate sensitive data visible on the page.

The `<img onerror>` vector additionally confirms the DOM sink renders arbitrary HTML - not just `<script>` tags — meaning the entire range of HTML-based XSS payloads (SVG, input, body handlers, etc.) is available to an attacker, significantly broadening the exploitable attack surface beyond what a `<script>`-only finding would imply.

## Recommendations for Fix
1. Replace unsafe DOM sinks with safe alternatives:
   - Use `textContent` instead of `innerHTML` for plain text output.
   - Use `createElement` + `appendChild` instead of `document.write`.
2. If HTML rendering is required, sanitize client-side using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(userInput);
   ```
3. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth.
4. Since this is the fourth DOM XSS instance across this lab (IMDb, AniList, MilesWeb, Censys), the root cause is likely a shared client-side search component — apply the fix centrally to that shared JS module rather than per-subdomain.
