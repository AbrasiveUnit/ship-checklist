# Ship checklist: {{project}}

Profiles: {{W A M J G}}   ·   Last run: {{YYYY-MM-DD}} by {{who}}   ·   Rules: [ship-checklist skill](https://github.com/AbrasiveUnit/ship-checklist)

Status is one of: **Pass** (with evidence) · **N/A: reason** (X lines add the trigger) · **Owner: action**. No blank lines.

## SEO (W), per `site-seo`

| ID | Check | Status | Evidence |
|---|---|---|---|
| SEO-1 | sitemap.xml, every public and dynamic page | | |
| SEO-2 | robots.txt allows public, blocks API/admin/auth, names sitemap | | |
| SEO-3 | no wrong noindex | | |
| SEO-4 | absolute canonical on every indexable page | | |
| SEO-5 | one host, 301/308 to HTTPS origin, HSTS | | |
| SEO-6 | unique descriptive titles | | |
| SEO-7 | unique meta descriptions, 120-160 chars | | |
| SEO-8 | exactly one h1 | | |
| SEO-9 | headings in order | | |
| SEO-10 | alt text on every meaningful image | | |
| SEO-11 | clean slugs | | |
| SEO-12 | true JSON-LD schema | | |
| SEO-13 | internal links, key pages two clicks from home | | |
| SEO-14 | no broken internal links | | |
| SEO-15 | OG image and Twitter card, 1200x630 | | |
| SEO-16 | images compressed, sized, modern formats | | |
| SEO-17 | Lighthouse performance and SEO 90+ on top 3 pages | | |
| SEO-18 | mobile: no sideways scroll at 390 px, 44 px taps, 16 px text | | |
| SEO-19 | Search Console verified, sitemap submitted, backlink plan | | |

## Security

| ID | Check | Applies | Status | Evidence |
|---|---|---|---|---|
| S1 | Security headers (HSTS, CSP no inline, XFO, nosniff, Referrer, Permissions) | W A | | |
| S2 | TLS everywhere, one HTTPS origin, https-only user links | W A M | | |
| S3 | Secrets only in server env, bundle grep clean, rotation steps written | all | | |
| S4 | RLS on every table, server-side permission checks, signed + logged admin writes, 2FA on dashboards | A M | | |
| S5 | No SQLi, XSS or SSRF; schema validation on every route; uploads checked | W A | | |
| S6 | CORS exact origins, CSRF protection on cookie routes | A | | |
| S7 | Rate limits on public writes, form bot protection, WAF and DDoS on | W A M | | |
| S8 | Dependency audit in CI, Dependabot, runtime pins | all | | |
| S9 | Encrypted at rest, private buckets with expiring URLs, no PII in logs | A M | | |
| S10 | Transaction allowlist before signing, caps, Hero review before live | M | | |
| S11 | Agent Operating Instructions followed | G | | |
| S12 | Privacy, terms, consent, unsubscribe, SPF/DKIM/DMARC | W A | | |
| S13 | security.txt | W A | | |
| S14 | No pixel, analytics or third-party script loads before Accept; Reject works; network log saved | W A | | |

## Production readiness

| ID | Check | Applies | Status | Evidence |
|---|---|---|---|---|
| R1 | Indexes, no N+1, slow queries reviewed, pooler | A | | |
| R2 | Numbered migrations with rollback, version checkable | A | | |
| R3 | Daily backups, one restore tested, recovery page written | A M J | | |
| R4 | Locks/versions on races, DB uniqueness, job locks, saga steps, lag labeled | A M J | | |
| R5 | Timeouts, safe retries with backoff, graceful provider failure, idempotency keys | A M J G | | |
| R6 | Jobs listed + in health check, dead-letter, queues with batch caps, UTC + signed windows | J A | | |
| R7 | Webhook signatures + event ids, API versioned, semver releases | A | | |
| R8 | Simplest real-time transport, reconnect with backoff | A | | |
| R9 | Asset caching, edge caching rules, invalidation written | W A | | |
| R10 | p50/p99 recorded, load test at 10x, cold starts, platform limits, memory over a day | A | | |
| R11 | Scaling plan with triggers (replicas, partitions, sharding, multi-region = X) | A | | |
| R12 | Monthly cost line, spend caps and alerts | all | | |
| R13 | DNS written down, registrar lock, auto-renew, client owns domain | W | | |
| D1 | CI runs D5 stages, build, audit, red blocks deploy; auto deploy; previews; build cache | all | | |
| D2 | Feature flags, rollback tried once, staged app rollout | W A M | | |
| D3 | Config in repo, .env.example, Docker for own servers (K8s/Terraform/Helm = X) | all | | |
| D4 | /api/health, uptime check, liveness vs readiness | A J | | |
| D5 | Tests by layer: unit, integration, e2e on every main path, perf budget on money paths | all | | |
| O1 | Structured logs with request id, no secrets, security refusals logged | A M J G | | |
| O2 | Named metrics, alerts reach a named person | A M J | | |
| O3 | SLI, SLO, error budget written | A M | | |
| O4 | On call named, RUNBOOK first moves, postmortem rule, chaos drill | all | | |

## Owner actions

- {{list every Owner line here so the owner or the client sees them in one place}}
