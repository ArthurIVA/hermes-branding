# Proving site ownership to Google

Two different things get confused here. Know which one you need.

## 1. What the OAuth consent screen actually requires
Google's **Branding → App domain** check does **not** require Search Console verification. It only
needs the three URLs (home page, privacy policy, terms of service) to be publicly reachable over HTTPS
and to describe the app and its scopes. Our site already satisfies that:

- https://arthuriva.github.io/hermes-branding/
- https://arthuriva.github.io/hermes-branding/privacy.html
- https://arthuriva.github.io/hermes-branding/terms.html

If verification of the OAuth app is what you're submitting for, Search Console ownership proof is not
a listed prerequisite. Reviewers do sometimes look for signs the domain is genuinely the developer's,
so the tag below is cheap insurance — but it is not a gate.

## 2. Search Console ownership verification (if you want/need it)
Per https://support.google.com/webmasters/answer/9008080 — methods and whether they work on GitHub Pages:

| Method | Works here? | Notes |
|---|---|---|
| HTML tag (`<meta name="google-site-verification">` in `<head>` of the home page) | **Yes** | Simplest. Must be inside `<head>`, within the first 2 MB, on the page that matches the property URL exactly. |
| HTML file upload (`google1234.html` / `google1234.txt` at property root) | **Yes** | Drop the file at repo root; Pages serves it at `/hermes-branding/google1234.html`. Do not rename or edit its contents. Search Console does NOT follow cross-domain redirects. |
| Google Analytics / Tag Manager snippet | Yes but pointless | Adds a tracking script to a page whose privacy policy promises no telemetry. Don't. |
| Domain name provider (DNS TXT) | **No** | Requires a custom domain we control. `github.io` is not ours, so a **Domain property** (`arthuriva.github.io`) is impossible. Only URL-prefix properties work here. |

### Property type matters
- Use a **URL-prefix property**: `https://arthuriva.github.io/hermes-branding/` — exact string, including
  the trailing path. HTML tag/file methods only work for URL-prefix properties.
- Do not try to add `arthuriva.github.io` as a Domain property; it will fail without DNS control.

### The placeholder is already in the repo
`index.html` carries:

```html
<meta name="google-site-verification" content="PLACEHOLDER">
```

with an explanatory comment above it. Steps:
1. Search Console → Add property → **URL prefix** → paste `https://arthuriva.github.io/hermes-branding/`
2. Choose **HTML tag**, copy the `content="..."` value Google shows.
3. Replace `PLACEHOLDER` in `index.html` with that value (keep it inside `<head>`), then push.
   Ask Hermes to do it — one command: give Hermes the token value, not the whole page.
4. Wait for the Pages rebuild (~1–3 min), confirm the tag is live:
   `curl -s https://arthuriva.github.io/hermes-branding/ | grep google-site-verification`
5. Click **Verify** in Search Console.

Prefer the file method instead? Save Google's file into the repo root as-is, push, confirm it returns
200 at `/hermes-branding/<filename>`, then Verify.

## Keeping verification from lapsing
Google re-checks tokens periodically with the **Google Site Verifier** user agent and notifies you if the
token vanishes; unfix it and owner permissions expire after a grace period. So:
- Never remove the meta tag or the uploaded file when editing the site.
- Add a second method (tag + file) so one failing doesn't cost verification.
- Verification is tied to the Google account that generated the token — keep using the same Gmail.

## GitHub Pages gotchas that cause verification failures
- The page must be reachable **logged out**. Test in an incognito window.
- A 200 can be a **stale build** — grep the served HTML for the token, don't trust status alone.
- `GET /hermes-branding/googleXXX.html` on a non-existent name returns **404**; a **503** means Pages
  was mid-deploy. Re-check after a minute rather than assuming failure.
