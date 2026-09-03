## Title
Reflected Cross-Site Scripting (XSS)— PubMed National Library of Medicine Search Builder

## Vulnerability Type
Reflected XSS

## Summary
The search functionality on `https://kzlabs.in/subdomains/pubmed/index.php` reflects the `term` query parameter directly into the page's HTML response without proper sanitization or output encoding. This allows an attacker to inject arbitrary HTML/JavaScript that executes in the victim's browser when they visit a crafted URL. This instance uses an `<img onerror>` based payload, confirming that the application is not filtering event-handler attributes either.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/pubmed/index.php?term=`

## Steps to Reproduce
1. Navigate to the following URL:
   `https://kzlabs.in/subdomains/pubmed/index.php?term=tanishq%27%22%3E%3CImg+SrC%3Dx+OnErrOr%3Dconfirm%281%29%3E`
2. Observe that a JavaScript confirm box pops up displaying `1` — confirming the injected payload executed in the page context via the image `onerror` event handler.

## Payload Used
```
tanishq'"><Img SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept Request
<img width="1915" height="1041" alt="Screenshot 2026-09-03 071208" src="https://github.com/user-attachments/assets/4a3c8043-9af1-4cd1-b37d-da55f087d755" />
<img width="1917" height="1041" alt="Screenshot 2026-09-03 071221" src="https://github.com/user-attachments/assets/af9a6dae-bb8c-465b-bc3a-490cb8e105c5" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data — all by getting a victim to click a single crafted link.

## Recommendations for Fix
Apply context-aware output encoding on the `term` parameter (e.g. `htmlspecialchars($term, ENT_QUOTES, 'UTF-8')` in PHP) before reflecting it into the page, and add a Content-Security-Policy header as defense-in-depth. Note that this payload used an `<img onerror>` vector rather than `<script>` tags, so any fix relying solely on stripping `<script>` tags will remain bypassable — output encoding of all HTML metacharacters is required regardless of tag/attribute used.
