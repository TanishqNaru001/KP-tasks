## Title
DOM-Based Cross-Site Scripting (XSS) via Search Parameter - AniList Clone Search Bar 
## Vulnerability Type
DOM-Based XSS


## Summary
The AniList clone at `https://kzlabs.in/subdomains/anilist/` is vulnerable to DOM-based XSS via the search bar. JavaScript on the page reads the user-supplied search value and writes it back to the DOM unsafely using a sink such as `innerHTML` or `document.write()`, without sanitization. The payload `'"><SCripT>confirm(1)</SCripT>` using a mixed-case `<script>` tag confirms execution and bypasses any client-side filter checking for lowercase `<script>`. This is the second DOM XSS instance confirmed on this lab (alongside the IMDb clone), suggesting the same vulnerable client-side search component is reused across subdomains.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/anilist/`

**Vulnerable sink:** Client-side JavaScript reading the search input and writing it to the DOM without sanitization (confirm by checking page JS for `innerHTML`, `document.write`, `eval`, or `insertAdjacentHTML` handling the search value).

## Steps to Reproduce
1. Navigate to `https://kzlabs.in/subdomains/anilist/`
2. In the search bar, enter the following payload:
   ```
   '"><SCripT>confirm(1)</SCripT>
   ```
3. Submit the search or observe as-you-type.
4. Observe that a JavaScript confirm box pops up - confirming the payload was written to the DOM and executed client-side.

## Payload Used
```
'"><SCripT>confirm(1)</SCripT>
```

## How to Confirm It Is DOM XSS (Not Reflected)
1. Submit the payload and observe the confirm dialog firing.
2. **View raw page source** (`Ctrl+U`) - payload should **not** appear in the server's HTML response.
3. **DevTools → Elements tab** - payload **will** appear in the rendered DOM, injected by client-side JS.
4. Check the page's JavaScript for sinks like:
   ```javascript
   document.getElementById('results').innerHTML = searchQuery;
   document.write(searchTerm);
   ```

## Proof of Concept

<img width="1917" height="1040" alt="image" src="https://github.com/user-attachments/assets/919f1057-5a4b-4f60-a4a5-988181e128a7" />
<img width="1917" height="1037" alt="image" src="https://github.com/user-attachments/assets/fbc361a1-e8d6-4a73-9ccb-9fb6af5ecea6" />


## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser by crafting a malicious URL or search input:

- Steal session cookies and perform account takeover.
- Redirect victims to phishing or malware pages.
- Perform unauthorized actions on behalf of the victim.
- Exfiltrate sensitive data visible on the page.

Server-side WAF rules and backend output encoding patches will **not** fix DOM XSS — the fix must be applied in the client-side JavaScript itself.

## Recommendations for Fix
1. Replace unsafe DOM sinks with safe alternatives:
   - Use `textContent` instead of `innerHTML` for plain text output.
   - Use `createElement` + `appendChild` instead of `document.write`.
2. If HTML rendering is required, sanitize using **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(userInput);
   ```
3. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth.
4. Since the same vulnerable search component appears on both the IMDb and AniList subdomains, the fix should be applied to the shared JS component centrally rather than per-subdomain.
