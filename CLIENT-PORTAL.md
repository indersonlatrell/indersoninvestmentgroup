# Client Portal — build & deploy notes

A per-client, read-only view of a managed funded account. First client: **Wesley Blanca**
(MyFundedFutures Builder 25K, login `MFFUSFBLDR111140017`).

## Files added
```
login.html                 # real credential check → routes to client.html
client.html                # the funded-account dashboard (data-driven)
data/wesley-blanca.json     # Wesley's account snapshot (the page reads this)
```
No build step. Render static-site publish dir stays `.`. Push to `main` and it deploys.

## How a client signs in
- Go to `/login.html`, enter **Access ID** + **Passcode**.
- On success the browser stores a session flag and opens `/client.html`, which loads
  `data/<slug>.json` for that client.

Wesley's credentials:
- Access ID: `WESLEY-BLANCA`
- Passcode: *(sent separately — change it any time by replacing the hash, see below)*

## Adding another client (3 steps)
1. Add `data/<slug>.json` (copy Wesley's and edit the numbers).
2. In `login.html`, add a row to `CLIENTS`:
   `"THEIR-ID": { slug: "<slug>", name: "Their Name", hash: "<sha256 of passcode>" }`
3. Generate the hash: in a terminal
   `printf '%s' 'their-passcode' | shasum -a 256`
   (or in the browser console: `crypto.subtle.digest('SHA-256', new TextEncoder().encode('their-passcode')).then(b=>console.log([...new Uint8Array(b)].map(x=>x.toString(16).padStart(2,'0')).join('')))`).

## Keeping the data fresh
`data/wesley-blanca.json` is a snapshot. It maps 1:1 to the **Notion Account Status** row
(`MFFU 25K — MFFUSFBLDR111140017`) plus the **Trade Journal** rows tagged `MFFU 25K` —
the same numbers the morning/evening MFF jobs already recompute. Two ways to refresh:
- **Manual:** ask Claude "refresh Wesley's portal" — it re-reads Notion and regenerates the JSON for you to commit.
- **Automatic (recommended):** give Claude write access to this repo (a GitHub token, the Zapier GitHub
  connection, or connect the repo folder on the Mac) and it will commit the updated JSON on the same
  schedule as the MFF status jobs.

## ⚠️ Security — read before relying on this for privacy
The login is a **client-side gate**. It's fine for one client, but the JSON is served at a public
URL, so anyone who guesses `data/wesley-blanca.json` could read it without signing in. Before you add
more clients (or show real money), move the data behind a server check. The clean upgrade, reusing the
Cloudflare Worker the myjournal app already uses:
1. Store each client's passcode hash in the Worker (KV or env).
2. Add a Worker route `GET /api/client?id=…` that returns the JSON **only** after verifying the passcode.
3. Point `client.html`'s `fetch()` at that route instead of the static file, and keep the JSON out of the public repo.
This makes each client's data genuinely private and removes the "guess the URL" hole.

## Two things to fix / confirm (not code)
1. **Remove the "SEC Registered Investment Advisor" line** from `index.html`, `dashboard.html`,
   `analytics.html`, `trades.html` footers. Unless IIG actually holds that registration, it's a false
   regulatory claim and a real legal exposure. (The new `login.html`/`client.html` already omit it.)
2. **Confirm MyFundedFutures allows this arrangement.** Many prop firms prohibit trading a funded
   account on behalf of a third party / taking outside investor capital against the account. Worth
   verifying against MFF's terms so the account isn't at risk before you formalize the client relationship.
