# waf2p_redirect

A tiny static site that sends every request for `waf2p.dev` and `www.waf2p.dev`
to the same path on `https://www.wombat-gov.dev`.

- `index.html` — root redirect
- `404.html` — catches every other path on GitHub Pages (the requested URL is
  preserved, so the script can carry it over)
- `CNAME` — `waf2p.dev` for GitHub Pages
- `robots.txt` — keeps the old domain out of search indexes

The redirect preserves path, query string and hash, and normalises
`/index.html` to `/`.

## Deploy on GitHub Pages

1. Create a new repository (for example `waf2p-redirect`) and push the contents
   of this folder to its default branch.
2. **Settings → Pages → Build and deployment**: source **Deploy from a branch**,
   branch `main`, folder `/ (root)`.
3. **Settings → Pages → Custom domain**: enter `waf2p.dev` and save. The
   `CNAME` file is already included.
4. Tick **Enforce HTTPS** once the certificate is issued.
5. DNS at the registrar:
   - apex `waf2p.dev` → A records
     `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - `www` → CNAME to `<your-github-user>.github.io`

GitHub Pages will redirect `www.waf2p.dev` to `waf2p.dev`, then this site sends
both to Wombat.

### Note on status codes

GitHub Pages serves `404.html` with a **404** status for unknown paths, then the
script redirects. Users land correctly, but it is not a 301. For a real
permanent redirect, use an edge rule:

- **Cloudflare Redirect Rule**: when host is `waf2p.dev` or `www.waf2p.dev`,
  redirect to `https://www.wombat-gov.dev` with *Preserve query string*,
  status **301**. Static redirects also preserve the path.
- **Netlify** (`_redirects`): `/* https://www.wombat-gov.dev/:splat 301!`
- **Vercel** (`vercel.json`): a `redirects` entry with
  `"source": "/(.*)"`, `"destination": "https://www.wombat-gov.dev/$1"`,
  `"permanent": true`.

## Important

`.dev` is on the HSTS preload list, so the old domain **must** be served over
HTTPS. An HTTP-only redirector will be upgraded by the browser and fail.
