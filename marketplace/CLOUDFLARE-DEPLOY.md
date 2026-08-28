# Deploying to Cloudflare Pages — marketplace.milesdash.com

> The 5-minute path to the live prototype on your own subdomain. The site is plain static HTML
> (no build step), so Cloudflare Pages serves it straight from this repo and redeploys on every
> merge to `main` — same rhythm as GitHub Pages today.

## A. Create the Pages project (~3 min, once)

1. Go to **dash.cloudflare.com → Workers & Pages → Create application → Pages →
   Connect to Git**.
2. Authorize GitHub if asked, and pick **`avieisner-boop/Focus-deck`**.
3. Build settings:
   - Project name: **quietlot** (gives you `quietlot.pages.dev`)
   - Production branch: **main**
   - Framework preset: **None**
   - Build command: *(leave empty)*
   - Build output directory: **`marketplace`**
4. **Save and Deploy.** In ~1 minute the full prototype is live at
   `https://quietlot.pages.dev` — landing, demo app, terms, privacy, design archive.

From then on every merge to `main` auto-deploys production, and every PR gets its own
preview URL — nothing else to maintain.

## B. Attach marketplace.milesdash.com (~2 min)

In the Pages project: **Custom domains → Set up a custom domain →** enter
`marketplace.milesdash.com`.

- **If milesdash.com's DNS is on Cloudflare** (nameservers point at cloudflare.com):
  Cloudflare creates the CNAME automatically and the domain is active in minutes, with
  TLS issued for you. Done.
- **If DNS lives elsewhere** (GoDaddy, registrar default, etc.): add a CNAME record at
  your DNS host — name `marketplace`, target `quietlot.pages.dev` — then finish the
  validation step in the Cloudflare dashboard. (Optional but recommended: move
  milesdash.com's nameservers to Cloudflare's free plan so future records are one click.)

## Notes

- `marketplace/_headers` in this repo sets security headers (frame-deny, nosniff,
  referrer policy) — Cloudflare Pages applies it automatically; GitHub Pages ignores it
  harmlessly. A strict Content-Security-Policy is deliberately NOT set yet: the prototype
  uses inline scripts by design; the CSP tightens when the real app ships (SECURITY.md §2).
- All in-site links are relative, so the same files serve correctly from GitHub Pages,
  Cloudflare Pages, and the custom domain simultaneously.
- When quietlot.com (or whichever brand domain wins) is purchased, it attaches to this
  same Pages project the same way — marketplace.milesdash.com can stay as an alias or be
  removed later.
