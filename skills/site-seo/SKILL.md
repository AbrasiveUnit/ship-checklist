---
name: site-seo
description: The search-ready checklist every website passes before launch, and the audit for an existing site. 19 checks across crawl and index, on-page, structure and sharing, speed and mobile, and off-site. Use whenever building, launching, redesigning or reviewing any website or landing page, or when someone says a site "isn't showing on Google", or asks about SEO, sitemaps, robots.txt, meta tags, canonicals, Search Console, Lighthouse, Core Web Vitals or backlinks. Part 1 of the ship-checklist skill.
license: MIT
metadata:
  version: 1.0.0
---

# Site SEO: the launch checklist

A site is not done until it passes this list. Every item is either done, already fine, or an owner action written down (an `OPEN:` line in the project's decision log). Apply it while the site is built, not as a pass afterwards.

## The 19 checks

**Crawl and index**
1. `sitemap.xml` with every public page, plus dynamic pages (products, listings, posts) read from the data source, bounded and cached. If the data source is down, the sitemap still builds with the static pages.
2. `robots.txt` allows public pages, disallows API, admin, auth, messages and other per-user routes, and names the sitemap.
3. No wrong `noindex`. Keep deliberate ones (private pages, empty records) with a comment saying why.
4. A canonical URL on every indexable page, absolute against one origin.
5. One host: `www` and `http` 301/308 to the bare HTTPS origin (or the reverse, but one). HSTS on.

**On each page**
6. A unique, descriptive title (a root template like `%s · Brand`).
7. A unique meta description, 120 to 160 characters, written for a person.
8. Exactly one `<h1>`.
9. Headings in order (h1 → h2 → h3), no skipped levels in main content.
10. Alt text on every meaningful image; decorative images get `alt=""`.
11. Clean slugs: lowercase, hyphens, no IDs or query strings where a word works.

**Structure and sharing**
12. JSON-LD schema that is true: Organization and WebSite on the root; Product, SoftwareApplication, LocalBusiness or Article as the page warrants. Never invent ratings, reviews or prices.
13. Internal links: every key page reachable in two clicks from home, key pages link each other, and the footer carries the main set.
14. No broken links: crawl the production build; every internal href returns 200.
15. OG image and Twitter card on the root and on every page people share, 1200×630.

**Speed and mobile**
16. Images compressed (no file over about 200 KB without reason), modern formats, explicit width and height.
17. Core Web Vitals: Lighthouse on the production build for the top 3 pages, performance and SEO 90+ as the target; record before and after.
18. Mobile: no sideways scroll at 390 px, tap targets 44 px, text 16 px base.

**Off-site (owner actions)**
19. Search Console: the verification tag is read from an env var (`GOOGLE_SITE_VERIFICATION`), then the owner adds the property, verifies, and submits `/sitemap.xml`. Plus a one-page backlink plan in the repo (directories, listings, launch posts, partner pages that fit the niche; no invented contacts).

## How to run it

- **Next.js App Router:** `app/sitemap.ts`, `app/robots.ts`, `metadataBase` and `title.template` in the root layout, `alternates.canonical` in each page's metadata or `generateMetadata`, `verification.google` from env, JSON-LD as a `<script type="application/ld+json">` in a server component, the host redirect in `vercel.json` (`has: [{type: "host", value: "www.<domain>"}]`) or `next.config`.
- **Static or generated sites:** the generator writes `sitemap.xml` and `robots.txt` at build, each template gets the title, description, canonical and OG block plus JSON-LD, and a build-time check fails on a missing h1, a missing alt or a broken internal link.
- **Verify on the production build**, not the dev server: `curl` `/sitemap.xml` and `/robots.txt`, a link crawl over the sitemap, Lighthouse (`npx -y lighthouse@12 <url> --only-categories=performance,seo,accessibility --chrome-flags=--headless=new --output=json`), Playwright at 390 px for sideways scroll.
- **After deploy:** `curl -sI` the live root for HSTS, `curl -s -o /dev/null -w "%{http_code} %{redirect_url}"` on `http://` and `www.`, and fetch the live sitemap.
- **Report** a table of the 19 checks: done / already fine / owner action, plus Lighthouse before → after.

## Running it with an agent

A full pass on an existing site is one agent at high effort on its own branch with this skill loaded first. A new site gets the checks inside its build brief.
