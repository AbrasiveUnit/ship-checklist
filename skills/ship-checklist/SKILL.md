---
name: ship-checklist
description: The checklist a project passes before it is called done, launched or handed to a client. It covers SEO (the 19 checks in the bundled site-seo skill), security S1-S14 (headers, TLS, secrets, access control, injection, CORS/CSRF, rate limits, dependencies, data at rest, money, AI agents, legal and email, security.txt, tracking-pixel consent) and production readiness (R1-R13, D1-D5, O1-O4, covering 111 backend topics from rate limiting to postmortems, plus layered tests in CI). Use whenever building, launching, handing off or reviewing any project (websites, SaaS apps, APIs, mobile apps, bots, scheduled jobs, AI agents), when starting a new project, or when someone asks whether something is "production ready", "secure", "client ready" or "safe to launch", or mentions tracking pixels, cookie banners, analytics, tests, rate limits, backups, monitoring, secrets, CORS, CSRF, XSS, migrations, rollbacks or incidents.
license: MIT
metadata:
  version: 1.0.0
---

# Ship checklist: SEO, security, production readiness

Every project gets `docs/SHIP-CHECKLIST.md`, copied from `template.md` beside this file. Every line ends in one of three states, never blank:

- **Pass**, with evidence (a command output, a file path, a link to a CI run or a decision).
- **N/A: reason**. For scale-only items, add the trigger that would make it apply.
- **Owner: action**, for steps only the owner or the client can take (DNS, Search Console, billing caps, legal sign-off).

A project is done when no line is blank. Re-run the checklist on every launch, redesign and client handoff, and every quarter for live client work.

## Profiles: which lines apply

Tag the project with every profile that fits, then work the lines tagged with those letters.

| Tag | Profile | Examples |
|---|---|---|
| W | Any website | marketing site, landing page, client site, blog |
| A | App or API with logins, user data or a database | SaaS app, mobile app backend, dashboard |
| M | Moves money, holds keys or trades | checkout, wallet, trading bot, payouts |
| J | Background jobs, bots, scheduled pipelines | cron jobs, scrapers, social posting bots, ETL |
| G | AI agent facing customers | support chatbot, in-app assistant |
| X | Scale only: record N/A with the trigger, revisit when the trigger hits | sharding, Kubernetes, multi-region |

## Part 1. SEO (W)

The 19 checks in the `site-seo` skill (bundled in this plugin), run as that skill says, on the production build. The template lists them as SEO-1 to SEO-19.

## Part 2. Security

- **S1 Security headers** (W A). Every response: `Strict-Transport-Security: max-age=63072000; includeSubDomains` (add `preload` only after every subdomain is confirmed on HTTPS), a `Content-Security-Policy` with no inline scripts, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=(), browsing-topics=()`. Verify with `curl -sI https://<domain>`. On Vercel these go in the `headers` block of `vercel.json`; on Next.js, `headers()` in `next.config`.
- **S2 TLS everywhere** (W A M). HTTPS only, `http://` and `www.` redirect to one HTTPS origin, no mixed content, user-supplied links open only `https://`. Covers TLS and encryption in transit.
- **S3 Secrets management** (all). Secrets live only in server env (hosting env vars, your database's secret store, a password manager or vault). Never in a `NEXT_PUBLIC_`, `VITE_` or `EXPO_PUBLIC_` variable, the repo, a prompt or the app bundle. `.env*` in `.gitignore`, a `.env.example` lists every variable without values. Grep the production bundle for key prefixes (`sk-`, `sk_live`, `service_role`, `eyJ`). Each secret has a written rotation step (where it lives, who rotates it); session and JWT signing secrets can rotate without logging everyone out.
- **S4 Access control** (A M). Row-level security (for example Supabase RLS) on every table, checked with the provider's security advisor; admin or service keys are server only; permission checks live in the server handler, never only in the UI or a prompt. Admin writes need a fresh, signed or re-authenticated action and land in an append-only audit log first. Sign-in uses a provider (Supabase Auth, Auth0, Clerk, OAuth) with the smallest scopes, never hand-rolled password storage; tokens are short-lived with refresh. Dashboard accounts (hosting, database, registrar, app stores, Google) use 2FA and least privilege (IAM).
- **S5 Input handling** (W A). Parameterized queries or the database client only, no SQL built from strings (SQL injection). No `dangerouslySetInnerHTML` or `innerHTML` on user content (XSS). Server fetches of user-supplied URLs go through a host allowlist and refuse private IPs (SSRF; image and link-preview endpoints are the usual culprit). Every API route validates its body with a schema (zod or similar). Uploads are checked for type and size and tied to a hash of the bytes.
- **S6 Cross-origin** (A). CORS allows exact origins, never `*` with credentials. Cookie-authenticated routes that change state use `SameSite=Lax` or `Strict` plus an Origin check, or bearer tokens (CSRF).
- **S7 Abuse limits** (W A M). Rate limit every public write and every expensive route per IP and per user (for example 10 per minute per IP on payment routes). Forms have a honeypot or Turnstile. WAF rules and DDoS protection are on at the host or CDN, and you know how to switch on attack mode.
- **S8 Dependencies** (all). Lockfile committed; CI runs `npm audit --omit=dev --audit-level=critical` (or `pip-audit`); Dependabot or Renovate weekly for app folders and CI actions. Upgrades that move a runtime (major versions, mobile OTA runtime) are pinned and done on purpose. This is the answer to dependency hell and semantic versioning drift.
- **S9 Data at rest** (A M). Confirm the database and storage are encrypted at rest (most managed providers do this by default; cite the provider's doc). Anything not public sits in a private bucket behind signed URLs that expire (1 hour). No secrets or personal data in logs.
- **S10 Money and keys** (M). Every transaction is checked against a pinned allowlist (contracts, recipients, amounts) before anything signs or charges; full-address review on sends; spend and size caps; a second reviewer signs off before the live switch.
- **S11 AI agents** (G). API key server side only; every tool checks the user's permission in its handler, not in the prompt; no secrets or other users' data in the system prompt; tool and web output is treated as untrusted data (prompt injection); tool calls are logged; per-user and total spend caps.
- **S12 Email and legal** (W A). Privacy policy and terms pages; consent before any tracker loads (S14); anything that emails people has a working opt-out and a `List-Unsubscribe` header; SPF, DKIM and DMARC on any domain that sends.
- **S13 Disclosure** (W A). `/.well-known/security.txt` with a contact address and an expiry date.
- **S14 Tracking and consent** (W A). No tracking pixel, analytics tag or third-party script (Meta Pixel, Google Analytics, Ads or Tag Manager, TikTok pixel, Hotjar, chat widgets, embeds that set cookies) loads, sets a cookie or sends a request until the visitor clicks Accept. Reject is as easy as Accept, and nothing loads after Reject. Server-side conversion APIs (Meta CAPI, GA measurement protocol) follow the same consent flag. The privacy policy names each tracker. Evidence: a fresh private-window load of the top 3 pages, with the network log filtered to other domains, shows zero tracker requests before Accept (Playwright or browser DevTools; save the request list). Re-check whenever a tag or widget is added. Why: a cookie banner is not permission. Law firms send wiretap demand letters under the California Invasion of Privacy Act (Penal Code 631(a), with statutory damages of $5,000 per violation under 637.2) to sites whose pixels fire on page load, before Accept. A one-line fix to give your coding agent: block every pixel, analytics tag and third-party script until the visitor clicks Accept. A site with no trackers passes with the empty request list as evidence. Sites in health, finance or legal get their own counsel's sign-off as an Owner line. This is an engineering check, not legal advice.

## Part 3. Production readiness

Each line names the topics it covers. The coverage index at the end maps all of them.

### Data
- **R1 Queries** (A). Index every column used in a `where`, `join` or `order by` on a hot path (`EXPLAIN`, the provider's performance advisor); no N+1 queries (fetch lists with one joined or batched query); the slowest queries in `pg_stat_statements` reviewed once before launch; serverless code goes through a connection pooler.
- **R2 Migrations** (A). Every schema change is a numbered migration file in the repo, applied in order, with a written rollback; the applied version is checkable; schema changes reach production through the migration pipeline, never typed straight into the production database from a laptop. Covers database migrations and schema versioning.
- **R3 Backups and recovery** (A M J). Daily backups on (point-in-time recovery for M projects); one restore actually tested into a scratch project; a written recovery page: what happens if the host, the database or the domain fails, and who holds which login. Failover, replication and multi-region are X: single region is a stated choice until an SLO needs more.
- **R4 Concurrency** (A M J). Anything two requests can change at once uses a transaction with a row lock (pessimistic) or a version column (optimistic); uniqueness is a database constraint, not an app check; a scheduled job takes a lock (lock file or Postgres advisory lock) so two copies never run; locks are always taken in the same order (deadlocks). A multi-step flow across services (charge, then deliver) has a compensating step for each step (saga, distributed transactions). Data that can lag (cached balances, replicas, external feeds) is labeled as such in the UI (eventual consistency, CAP). One runner per scheduled job is the leader election.

### Calls to other services
- **R5 Failure handling** (A M J G). Every outbound call has a timeout; retries only on safe or idempotent calls, with exponential backoff, jitter and a max attempt count; when a provider is down the page shows cached data or a clear message, not a blank screen (circuit breaker); payments, orders, posts and webhook handlers use idempotency keys so a retry never doubles anything. Network partitions are handled by these same rules.
- **R6 Background work** (J A). Every cron job is listed in the repo with schedule, lock and owner, and is watched by a health check or uptime monitor. Failed items land in a dead-letter table or folder that the health check reports. Use a queue, pub/sub or events when work is slow or fans out, with a max batch size so producers cannot bury consumers (backpressure). Times are stored in UTC (`timestamptz`); signed messages carry a short validity window (about 5 minutes) so clock skew cannot replay them.
- **R7 Webhooks and versions** (A). Incoming webhooks verify the signature, store the event id (idempotent), answer fast and queue the work. A public API is versioned (`/v1`) and a breaking change gets a new version; releases follow semantic versioning.
- **R8 Real-time** (A). Pick the simplest that works: polling for minute-level freshness, Server-Sent Events for a one-way stream, WebSockets for two-way; reconnect with backoff; long polling only as a fallback. gRPC, HTTP/2 and HTTP/3, TCP vs UDP: N/A on a managed host that already serves HTTP/2 and 3; they apply only to service-to-service traffic you build yourself.

### Speed and scale
- **R9 Caching** (W A). Fingerprinted assets cached a year as `immutable`; HTML short-cached or ISR; safe API responses use `s-maxage` with `stale-while-revalidate` at the CDN edge; per-user data is never cached at the edge; each cache has a written invalidation rule (what busts it).
- **R10 Performance budget** (A). p50 and p99 latency recorded for the top routes, target p99 under 1 s for a user action, and tail latency outliers explained; one load test at 10x expected traffic (`npx autocannon` or k6) for throughput; cold starts kept small (lean imports, edge runtime where it fits); the platform limits written down (function timeout and body size, database connection cap, each provider's rate limit). Long-running jobs checked for memory growth over a day (memory leaks, garbage collection).
- **R11 Scaling plan** (A, X lines). Write today's setup (serverless functions scale out, the database scales up by tier) and the next step with its trigger, for example database CPU over 70% for a week, bump the tier; read-heavy, add a read replica; a table past about 100 M rows, partition it. Load balancing, reverse proxies, API gateways and service discovery come from the host; a service on your own server sits behind Caddy or nginx with TLS. Sharding and multi-region are X.
- **R12 Cost** (all). A monthly cost line per service in the project docs; spend caps and alerts on at every provider that has them; paid API usage logged.
- **R13 DNS and domain** (W). Records written down, registrar lock and auto-renew on, TTL lowered before a move. For client work, the client owns the domain account and the builder holds delegated access.

### Shipping
- **D1 CI/CD** (all with code). Every push runs the D5 test stages, build and the dependency audit, and a red stage blocks the deploy; `main` deploys automatically; each branch gets a preview; build caching on.
- **D2 Release safety** (W A M). Risky features ship behind a flag (feature flags); the rollback is known and tried once (instant rollback on the host, `git revert`, the previous mobile update); app updates roll out to a share of users first when the change is risky (canary). Blue-green and rolling deployments: atomic deploys on most managed hosts already give this; say so.
- **D3 Config as code** (all). Host config, database migrations, build config, scheduler definitions and `.env.example` live in the repo (infrastructure as code). Docker for any service you run on your own server; Kubernetes, Terraform and Helm charts are X: N/A on managed hosting, revisit if you run your own fleet.
- **D4 Health checks** (A J). `/api/health` checks the database and returns 200 or 503; an uptime monitor hits it; a long-running service separates liveness (process up) from readiness (dependencies up).
- **D5 Tests by layer** (all with code). The pipeline runs push → unit → integration → end to end → performance → ship, every feature gets more than one kind of test, and AI-speed coding is the reason to write them, not to skip them. Unit: every function with logic (prices, totals, tax, fees, permissions, parsing) has tests. Integration: each route or module runs against a real test database or provider sandbox, not only mocks. End to end: every main user path (sign in, search, add to cart, checkout, pay, edit profile) is clicked through by Playwright on the web or Maestro on mobile, each with one failure case; let the agent write and run them. Performance: the money and sign-up paths have a time budget (for example checkout answers in under 1 s at p95) checked in CI or before each launch, and the numbers feed R10. Evidence: the test count per layer and the latest green CI run. A static site with no logic: unit and integration N/A, an end-to-end smoke test covers forms and links.

### Seeing problems
- **O1 Logging and tracing** (A M J G). Structured server logs with a request id carried through every service (distributed tracing); no secrets or personal data; refused security events are logged server side, not only in the browser console.
- **O2 Metrics and alerts** (A M J). Named metrics: errors per minute, p99, sign-ups, job success. Alerts reach a named person for site down, error spike, job failure and spend spike. This plus O1 is your observability.
- **O3 SLOs** (A M). One line each: the SLI (share of requests answered 2xx under 1 s), the SLO (99.5% a month), the error budget (about 3.6 hours a month). A blown budget means reliability work before new features.
- **O4 Incidents** (all). On call is named (plus the client contact on client work). `docs/RUNBOOK.md` has the first moves (roll back, flag off, post a status note). Any user-visible outage gets a postmortem in the decision log within 48 hours. Before an A or M launch, one chaos drill: break a provider in a preview (bad key, blocked host) and confirm the app fails gracefully.

## How to run it

1. Copy `template.md` to `<repo>/docs/SHIP-CHECKLIST.md` and tag the profiles at the top.
2. Work the lines in order: SEO with the `site-seo` skill, then S, R, D, O. Use command output as evidence (`curl -sI`, the database advisors, `npm audit`, Lighthouse JSON, the load test summary, the CI run).
3. Fix what fails, or record N/A with a reason or an Owner line. Record any real decision in the repo's decision log.
4. Report a short table: pass, N/A, owner actions, fixed this run.

## Running it with an agent

A full pass on an existing project is a big job: give it to one agent at high effort, on its own branch or worktree, with this skill (and `site-seo` for websites) loaded first. Keep the security lines that touch logins, money or stored data at high effort. M projects get a second reviewer before the live switch. A new project gets the checklist file on day one and the lines inside its build brief, not as a pass at the end.

## Coverage index: 111 backend topics

From the "Who's gonna tell vibe coders about..." list by @nataindata (instagram.com/reel/DeeR2JyNrx-):

Rate Limiting S7 · Caching R9 · Load Balancing R11 · Reverse Proxies R11 · API Gateways R11 · CI/CD D1 · Docker D3 · Kubernetes D3 · Service Discovery R11 · Circuit Breakers R5 · Timeouts R5 · Retries R5 · Exponential Backoff R5 · Idempotency R5 · Message Queues R6 · Pub/Sub R6 · Event-Driven Architecture R6 · Distributed Transactions R4 · Saga Pattern R4 · Dead Letter Queues R6 · Cron Jobs R6 · WebSockets R8 · Long Polling R8 · Server-Sent Events R8 · Database Indexing R1 · Query Optimization R1 · N+1 Queries R1 · Connection Pooling R1 · Read Replicas R11 · Sharding R11 · Partitioning R11 · Replication R3 · Leader Election R4 · CAP Theorem R4 · Eventual Consistency R4 · Optimistic Locking R4 · Pessimistic Locking R4 · Distributed Locks R4 · Race Conditions R4 · Deadlocks R4 · Memory Leaks R10 · Garbage Collection R10 · Thread Safety R4 · Backpressure R6 · Autoscaling R11 · Horizontal Scaling R11 · Vertical Scaling R11 · CDN R9 · Edge Caching R9 · Cache Invalidation R9 · Feature Flags D2 · Blue-Green Deployment D2 · Canary Releases D2 · Rolling Deployments D2 · Rollbacks D2 · Health Checks D4 · Liveness & Readiness Probes D4 · Monitoring O2 · Logging O1 · Distributed Tracing O1 · Metrics O2 · Alerts O2 · SLOs O3 · SLIs O3 · Error Budgets O3 · Observability O2 · Secrets Management S3 · IAM S4 · OAuth S4 · JWT Rotation S3 · TLS S2 · Encryption at Rest S9 · Encryption in Transit S2 · WAF S7 · DDoS Protection S7 · CORS S6 · CSRF S6 · SQL Injection S5 · XSS S5 · SSRF S5 · Database Migrations R2 · Schema Versioning R2 · Disaster Recovery R3 · Backups R3 · Failover R3 · Multi-Region Deployment R3 · Chaos Engineering O4 · Cost Optimization R12 · Cold Starts R10 · Serverless Limits R10 · Latency R10 · Throughput R10 · P99 Latency R10 · Tail Latency R10 · Network Partitions R5 · Clock Skew R6 · DNS R13 · TCP vs UDP R8 · HTTP/2 & HTTP/3 R8 · gRPC R8 · Webhooks R7 · API Versioning R7 · Semantic Versioning R7 · Infrastructure as Code D3 · Terraform D3 · Helm Charts D3 · Build Caching D1 · Dependency Hell S8 · Production Incidents O4 · On-Call O4 · Postmortems O4

## Coverage index: tracking pixels

From the tracking-pixel lawsuit video by @dr.erinjacques (instagram.com/reel/De-pHkzxiuX):

Tracking pixels S14 · Meta and Google pixels S14 · Scripts firing on page load before Accept S14 · A cookie banner is not permission S14 · Accept and Reject S14 · Analytics S14 · Third-party scripts S14 · California wiretap demand letters (Penal Code 631(a), 637.2) S14 · Privacy policy S12

## Coverage index: tests and CI/CD

From "Vibe coding is not software engineering" by @rollins.io (instagram.com/reel/DfdtXLfAim5):

CI/CD pipeline D1 · Unit tests D5 · Every feature, multiple tests D5 · Integration tests D5 · End-to-end tests D5 · Let the agent click every path D5 · Performance tests D5 R10 · Push → unit → integration → e2e → perf → ship D1 D5
