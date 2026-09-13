## Title
DOM-Based Cross-Site Scripting (XSS) via Search Parameter - IMDb Clone Search Bar

## Vulnerability Type
DOM-Based XSS

## Summary
The IMDb clone at `https://kzlabs.in/subdomains/imdb/` is vulnerable to DOM-based XSS via the search bar. Unlike the Reflected XSS findings in this assessment (where the server echoes unsanitized input back in the HTML response), DOM XSS occurs entirely client-side - JavaScript on the page reads the user-supplied search value from the URL or DOM and writes it back to the page unsafely using a sink such as `innerHTML`, `document.write()`, or `eval()`. The payload `'"><SCripT>confirm(1)</SCripT>` using a mixed-case `<script>` tag confirms execution bypasses any client-side filter checking for the lowercase `<script>` string.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/imdb/`

**Vulnerable sink:** Client-side JavaScript reading the search input and writing it to the DOM without sanitization (confirm by checking page JS source for `innerHTML`, `document.write`, `eval`, or `insertAdjacentHTML` handling the search value).

## Steps to Reproduce
1. Navigate to `https://kzlabs.in/subdomains/imdb/`
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
Mixed-case `SCripT` is used to bypass any client-side string filter checking for `<script>` in lowercase.

## How to Confirm It Is DOM XSS (Not Reflected)
1. Submit the payload and observe the confirm dialog firing.
2. **View the raw page source** (`Ctrl+U`) - the payload should **not** appear in the server's HTML response.
3. **Open DevTools → Elements tab** - the payload **will** appear in the rendered DOM, injected by client-side JavaScript.
4. If the payload is absent from page source but present in the DOM → confirmed DOM XSS.
5. Check the page's JavaScript files for sinks like:
   ```javascript
   document.getElementById('results').innerHTML = searchQuery;
   document.write(searchTerm);
   eval(userInput);
   ```

## Proof of Concept

<img width="1917" height="1035" alt="image" src="https://github.com/user-attachments/assets/7e1a32bd-b502-4f09-b625-b6b5bec2fab4" />
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/3bfb8052-4607-4470-8141-0a2825a023d3" />


## Impact
This vulnerability allows an attacker to execute arbitrary JavaScript in a victim's browser by getting them to visit a crafted URL or interact with a malicious search input:

- Steal session cookies and perform account takeover.
- Redirect users to phishing or malware-hosting pages.
- Perform unauthorized actions on behalf of the victim within the application.
- Exfiltrate sensitive data visible on the page.

DOM XSS is particularly notable because it is invisible in the server's HTML response - server-side WAF rules and output encoding patches applied to the backend will **not** fix this vulnerability. The fix must be applied in the client-side JavaScript code itself.

## Recommendations for Fix
1. **Never write unsanitized user input to the DOM** using dangerous sinks (`innerHTML`, `document.write`, `eval`, `setTimeout(string)`, `insertAdjacentHTML`). Replace with safe alternatives:
   - Use `textContent` or `innerText` instead of `innerHTML` when inserting plain text.
   - Use `createElement` + `appendChild` instead of `document.write` when building DOM elements.
2. If HTML rendering is required, sanitize client-side using a trusted library such as **DOMPurify**:
   ```javascript
   element.innerHTML = DOMPurify.sanitize(userInput);
   ```
3. Add a strict Content-Security-Policy header (avoiding `unsafe-inline` and `unsafe-eval`) as defense-in-depth — this prevents inline script execution even if DOM XSS payloads reach a sink.
4. Audit all JavaScript files on this page for any other instances of unsafe sink usage reading from `location.search`, `location.hash`, `document.referrer`, or any user-controlled DOM input.
