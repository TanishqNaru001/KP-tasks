## Title
Reflected Cross-Site Scripting (XSS) via `title` Parameter — HTML Tag Context Breakout on News Portal (kzlabs.in subdomain)

## Vulnerability Type
Reflected XSS (Filter Bypass / Tag Context Breakout)

## Summary
The news page on `https://kzlabs.in/subdomains/news/` reflects the `title` query parameter inside a `<title>` HTML element without sanitization or output encoding. Unlike the other findings in this batch, the payload here first closes the `</title>` tag before injecting the `<img onerror>` vector, confirming the reflection point sits inside a specific HTML element context and that the filtering does not account for tag-context breakout.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/news/?title=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/news/?title=%3C/title%3Etanishq%27%22%3E%3CImg%20SrC=x%20OnErrOr=confirm(1)%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the payload closed the surrounding `<title>` tag and the injected `<img>` element executed via its `onerror` event handler.

## Payload Used
```
</title>tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request

<img width="1917" height="1035" alt="Screenshot 2026-09-08 060359" src="https://github.com/user-attachments/assets/bea211ad-41cf-42ec-ae1e-f5a9c0d32a82" />
<img width="1906" height="1031" alt="Screenshot 2026-09-08 060417" src="https://github.com/user-attachments/assets/b327b695-1889-4ac8-88c9-39ffba847368" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link. The `</title>` breakout technique confirms this reflection point has a different HTML context than the body-level reflections seen elsewhere, so a generic body-context fix may not cover this endpoint.

## Recommendations for Fix
Apply context-aware output encoding on the `title` parameter (e.g. `htmlspecialchars($title, ENT_QUOTES, 'UTF-8')` in PHP) before placing it inside the `<title>` element, since even a "closed" tag-context still requires encoding of `<`, `>`, and quote characters to prevent breakout. Add a Content-Security-Policy header as defense-in-depth, and specifically review any input reflected inside `<title>`, `<textarea>`, or similar elements, as they need the same encoding as body content despite appearing structurally different.
