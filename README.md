# Atelier Dawat — Luxury Wedding Invitation Creator + Chargily Pay

## What's in this folder
- `index.html` — the whole app (builder, live preview, invitation pages). Open it directly for the UI, but the **payment button needs the two server functions below to actually work.**
- `api/create-checkout.js` — creates a 500 DZD Chargily checkout. Holds the **secret key**.
- `api/verify-checkout.js` — confirms with Chargily that a checkout was really paid before the invitation unlocks. Also holds the secret key.

## Why the secret key isn't inside index.html
Anything in a browser-served HTML/JS file can be read by anyone via "view source" — there is no way to hide it with obfuscation or minification. If the Chargily **secret** key were embedded there, any visitor could copy it and create or cancel checkouts on your account. The **public** key is safe client-side; the **secret** key is not. That's why the two files above exist: they run only on the server, and the browser only ever talks to them, never to Chargily directly with the secret key.

The `?paid=1` redirect from Chargily is also not proof of payment by itself — anyone could type that into the address bar. `api/verify-checkout.js` re-checks the real status of the checkout with Chargily's API before the invitation is shown, which is what makes the gate real.

## Deploy (Vercel, free, ~2 minutes)
1. Install the CLI once: `npm i -g vercel`
2. From this folder, run: `vercel`
3. Accept the defaults — Vercel auto-detects the `api/` folder as serverless functions and serves `index.html` as the static site.
4. (Recommended) In the Vercel project settings → Environment Variables, add `CHARGILY_SECRET_KEY` with your key, then remove the fallback literal from both files in `api/`. This keeps the key out of your source control entirely.

Any other Node-friendly host (Netlify Functions, Cloudflare Pages Functions, a small Express server, etc.) works the same way — just keep the two functions server-side and point the two `fetch('/api/...')` calls in `index.html` at wherever you host them.

## Test mode notes
- These are **test** Chargily keys — no real money moves. Chargily's sandbox checkout page lets you simulate a successful or failed EDAHABIA/CIB payment.
- Base URL used: `https://pay.chargily.net/test/api/v2/`. Switch to `https://pay.chargily.net/api/v2/` and your **live** keys when you're ready for real payments.
- Amount is fixed at 500 DZD per invitation link, set in `index.html` (`PAY_AMOUNT_DZD`).

## If you open index.html without deploying the API
The "إنشاء الدعوة" button will show an inline error explaining the endpoint isn't reachable — this is expected for a static-only preview. The builder and live preview work fully offline either way; only the paid checkout step needs the two server functions.
