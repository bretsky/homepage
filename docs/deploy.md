# Deploying brettselby.xyz

The site is static: everything in `public/` is uploaded as-is to Cloudflare Pages. There is no
build step.

| | |
|---|---|
| Host | Cloudflare Pages, **classic Pages project** `homepage` (not a Worker) |
| Config | `wrangler.toml` (`name`, `pages_build_output_dir = "public"`) |
| Production branch | `main` |
| Default URL | https://homepage-8yg.pages.dev |
| Custom domain | https://brettselby.xyz (set in the dashboard, see below) |
| Git integration | none: deploys are uploaded from this machine with Wrangler |

## Everyday deploy

```bash
npm run deploy
```

This runs `npx wrangler@4 pages deploy --branch main`, which uploads `public/` to production.
Only changed files are uploaded, and the new version is live within a few seconds.

- Wrangler warns when the working tree has uncommitted changes. Commit first, or pass
  `npm run deploy -- --commit-dirty=true` to deploy anyway.
- Each deploy also gets its own permanent URL (`https://<hash>.homepage-8yg.pages.dev`), printed
  at the end. Use it to check the exact version that went out.

### Preview without touching production

```bash
npx wrangler@4 pages deploy --branch preview   # -> https://preview.homepage-8yg.pages.dev
npm run dev                                    # local server on http://localhost:8788
```

Opening `public/index.html` directly in a browser also works.

### Check a deploy

```bash
curl -s https://brettselby.xyz/ | grep -E 'canonical|og:title'
npx wrangler@4 pages deployment list --project-name homepage
```

### Roll back

In the Cloudflare dashboard: Workers & Pages → `homepage` → Deployments → pick an earlier
production deployment → **Rollback to this deployment**. Or check out the old commit and run
`npm run deploy` again.

## One-time setup (already done)

1. `npx wrangler@4 login` (OAuth in the browser; the account that owns the brettselby.xyz zone).
2. Create the project:

   ```bash
   npx wrangler@4 pages project create homepage --production-branch main --force
   ```

   Recent Wrangler versions try to create new Pages projects as Workers instead. `--force`
   creates a classic Pages project, matching pickems and lps. It is only needed for this one
   command; `pages deploy` never needs it.
3. Custom domain: Workers & Pages → `homepage` → **Custom domains** → *Set up a custom domain* →
   `brettselby.xyz`. Because the zone is on Cloudflare, it creates the DNS record (a flattened
   CNAME at the apex) and the TLS certificate itself. Allow a few minutes before it resolves.

## SEO notes

- The page declares `https://brettselby.xyz/` as canonical, so the `*.pages.dev` copies aren't
  indexed as duplicates.
- Open Graph / Twitter tags and a JSON-LD `Person` block live in the `<head>` of
  `public/index.html`. Keep their descriptions in step with the bio when it changes.
- No sitemap or `robots.txt`: a single page doesn't need them.
- Once the custom domain is live, add `brettselby.xyz` to Google Search Console (verify via
  Cloudflare DNS) and request indexing.

## Related sites

| Site | Pages project | Repo |
|---|---|---|
| pickems.brettselby.xyz | `pickems` | `bretsky/pickems` (`scripts/deploy.sh`) |
| clr.brettselby.xyz | `lps` | `bretsky/lps` (`frontend/scripts/deploy.mjs`) |
