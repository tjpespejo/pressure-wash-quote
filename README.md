# Finch Harbor Pressure Washing — Instant Quote Page

A single-page web app that lets a pressure washing company give a customer a price
on the spot. The customer enters their details and the areas to be cleaned, sees the
price calculated live, and on submit the details and quote are emailed to
`careers@finchharbor.com`.

## Live page

- **URL:** https://tjpespejo.github.io/pressure-wash-quote/
- **Repository:** https://github.com/tjpespejo/pressure-wash-quote

A note at the top of the form tells the customer that the request is sent to
`careers@finchharbor.com`.

## Pricing rules

| Service  | Rate              |
|----------|-------------------|
| House    | $0.15 per sq ft   |
| Driveway | $0.12 per sq ft   |
| Deck     | $0.30 per sq ft   |
| Fence    | $1.00 per lin ft  |
| **Minimum job** | **$150**   |

The customer can select any combination of services and enter a size for each. The
total is summed from all selected services. If the summed total is greater than $0 but
below $150, the price is raised to the $150 minimum. The minimum is shown as a note on
the quote when it applies.

## How it works

The whole app is one static file, `index.html` — HTML, CSS, and vanilla JavaScript.
There is no backend server and no build step.

1. **Input** — The customer fills in name, email, phone, and address, then checks the
   services they need and types the size (square feet, or linear feet for fencing).
2. **Live quote** — As the customer types, JavaScript recalculates the price
   immediately using the rates above and applies the $150 minimum.
3. **Submit** — On submit, the page validates the input (name, valid email, at least one
   service with a size) and sends the quote by email.

## How the email submission works

Email is sent from the browser using **[EmailJS](https://www.emailjs.com/)**, a service
that delivers email directly from client-side JavaScript — no server required.

### The flow

1. The page loads the EmailJS browser SDK and initializes it with a **Public Key**.
2. When the customer submits, the page builds a set of template parameters:
   - `to_email` — the destination address, `careers@finchharbor.com`
   - `name`, `email`, `phone`, `address` — the customer's contact details
   - `quote_total` — the final price (e.g. `$300.00`)
   - `quote_breakdown` — a line-by-line summary of each service and cost
   - `time` — submission timestamp
3. The page calls `emailjs.send(SERVICE_ID, TEMPLATE_ID, templateParams)`.
4. EmailJS fills those parameters into a pre-defined **email template** and sends the
   message through a connected Gmail account (the "from" account).
5. The template's **To Email** field is set to the placeholder `{{to_email}}`, which
   receives `careers@finchharbor.com` from the page. So every quote is delivered to
   `careers@finchharbor.com`. A green confirmation message is shown to the customer on
   success.

The customer's own email (`{{email}}`) is captured as contact info and used as the
template's **Reply To**, so the company can reply straight to the customer. It is **not**
the destination of the quote.

### EmailJS configuration

Three identifiers are set near the top of the `<script>` block in `index.html`:

```js
var EMAILJS_PUBLIC_KEY  = "...";   // Account → General → Public Key
var EMAILJS_SERVICE_ID  = "...";   // Email Services → connected Gmail service
var EMAILJS_TEMPLATE_ID = "...";   // Email Templates → the quote template
var QUOTE_RECIPIENT     = "careers@finchharbor.com";  // passed to the template as to_email
```

The subject line and layout live in the EmailJS **template** on the EmailJS dashboard.
The destination is driven by the template's **To Email** field, which is set to the
placeholder `{{to_email}}` and receives its value from the page. The template uses these
variables: `{{to_email}}`, `{{name}}`, `{{email}}`, `{{phone}}`, `{{address}}`,
`{{quote_total}}`, `{{quote_breakdown}}`, `{{time}}`.

### Required EmailJS template settings

- **To Email:** `{{to_email}}`
- **Reply To:** `{{email}}`
- **Subject:** `New pressure washing quote: {{name}} — {{quote_total}}`

### Why EmailJS

A static page cannot send email by itself. EmailJS lets the page send through a Gmail
account that the company owns, delivering to any address — the recipient
(`careers@finchharbor.com`) does not need to verify or configure anything.

## Running locally

Open `index.html` directly in a browser, or serve the folder:

```bash
# any static server works, e.g.
npx serve .
```

## Deploying

The page is static, so any static host works. This project is deployed with
**GitHub Pages**: in the repository, go to Settings → Pages → Source: "Deploy from a
branch" → branch `main`, folder `/ (root)`. The page is then served at
https://tjpespejo.github.io/pressure-wash-quote/.

Other static hosts (Netlify Drop, Vercel, Surge) work equally well.

If EmailJS **Allowed Origins** is enabled (Account → Security), add the deployed domain
(e.g. `https://tjpespejo.github.io`) so submissions from the live page are not blocked.

## Files

- `index.html` — the entire application (markup, styles, pricing logic, email submission)
- `README.md` — this file
