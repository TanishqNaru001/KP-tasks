## Title
Stored Cross-Site Scripting (XSS) - Members Portal / Flipkart Clone 

## Vulnerability Type
Stored XSS (Persistent XSS)

## Summary
The product review functionality on `https://kzlabs.in/subdomains/members/` accepts and stores user-supplied input in the Detailed Review Comments field without sanitization or output encoding. The injected payload is persisted to the database and executed in the browser of any user who views the affected product page - without any further interaction from the attacker. Any authenticated user who browses the product listing is a potential victim.

## Vulnerable Endpoints
**Registration:** `https://kzlabs.in/subdomains/members/index.php?view=register`

**Login:** `https://kzlabs.in/subdomains/members/index.php`

**Injection point (product review form):** `https://kzlabs.in/subdomains/members/index.php?product_id=1`

**Execution point (payload triggers on page load):** `https://kzlabs.in/subdomains/members/index.php?product_id=1`

## Vulnerable Field
| Field | Trigger Location |
|---|---|
| Detailed Review Comments | Product page (`?product_id=1`) - visible to all users |

## Steps to Reproduce

### Step 1 - Register and log in
1. Navigate to `https://kzlabs.in/subdomains/members/index.php?view=register` and create an account.
2. Log in at `https://kzlabs.in/subdomains/members/index.php`.

### Step 2 - Inject payload via product review
1. Navigate to the product page: `https://kzlabs.in/subdomains/members/index.php?product_id=1`
2. Open the product listing and locate the review/comment submission form.
3. In the **Detailed Review Comments** field, enter the XSS payload:
   ```
   tanishq'"><ImG SrC=x OnErrOr=confirm(1)>
   ```
4. Submit the review.

### Step 3 - Verify payload execution
1. Reload the product page: `https://kzlabs.in/subdomains/members/index.php?product_id=1`
2. Observe that a JavaScript confirm box pops up displaying `1` - confirming the stored payload in the Detailed Review Comments field executed on page load.
3. For strongest PoC: open the product page in a separate browser or incognito session (simulating a different victim user) and confirm the payload fires there too without any attacker interaction.

## Payload Used
```
tanishq'"><ImG SrC=x OnErrOr=confirm(1)>
```

## Proof of Concept

<img width="1917" height="1031" alt="image" src="https://github.com/user-attachments/assets/f5452cb3-92f2-4760-9e35-f0ed7b18db55" />
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/860b989d-66f4-4460-8506-5321dca40172" />


## Impact
Stored XSS in a product review section carries high real-world impact:

- **Any user who views the affected product page has JavaScript executed in their browser** - no crafted link required.
- An attacker can steal session cookies (`document.cookie`) from every victim who browses the product, leading to mass account takeover across all platform users.
- On an e-commerce platform specifically, an attacker could silently redirect victims to phishing checkout pages, capture payment-related data rendered on the page, or manipulate cart/order actions on their behalf.
- The payload persists until the review is deleted or the stored data is sanitized - standard WAF rules and link-based protections provide no defence since the payload is served from the database on every page load.
- Other product pages (`?product_id=2`, `?product_id=3`, etc.) likely share the same vulnerable review component and should be considered equally affected.

## Recommendations for Fix
1. Apply context-aware output encoding (e.g. `htmlspecialchars($comment, ENT_QUOTES, 'UTF-8')` in PHP) on the Detailed Review Comments field before rendering it anywhere in the application.
2. Implement server-side input validation at submission time to reject inputs containing unescaped HTML metacharacters (`<`, `>`, `"`, `'`) in plain-text review fields.
3. If rich text or basic formatting is intentionally supported in reviews, use a server-side HTML sanitization library (e.g. HTMLPurifier for PHP) rather than a custom filter or denylist.
4. Add a strict Content-Security-Policy header (avoiding `unsafe-inline`) across the entire members subdomain, which would block inline event-handler execution (`onerror`, `onclick`, etc.) even if stored payloads reach the page.
5. Audit all other user-generated content fields in the application (ratings, Q&A, seller responses) for the same issue, as the root cause is likely a shared rendering layer that does not encode output.
