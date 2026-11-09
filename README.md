# Ship Checklist

**The list your project passes before you call it done.** A Claude Code plugin (two Agent Skills) that audits any website, app, bot or AI agent for SEO, security and production readiness, and makes it prove each line.

AI makes it easy to ship fast. It does not make the thing safe, findable or able to survive real traffic. This checklist covers the gaps that vibe-coded projects usually miss: rate limits, backups, secrets in the bundle, tracking pixels that fire before consent, missing tests, no rollback plan.

Every line ends one of three ways. Blank is not allowed.

- **Pass**, with evidence (command output, a file, a CI run)
- **N/A**, with a reason (and the trigger that would make it apply)
- **Owner**, an action only a person can take (DNS, Search Console, legal sign-off)

## Install

In Claude Code:

```
/plugin marketplace add AbrasiveUnit/ship-checklist
/plugin install ship-checklist@ship-checklist
```

Then, inside any project:

```
Run the ship checklist on this project.
```

Claude copies the template to `docs/SHIP-CHECKLIST.md`, works every line that applies, fixes what it can, and hands you a table of passes, N/As and owner actions.

**Without the plugin system:** copy `skills/ship-checklist` and `skills/site-seo` into `~/.claude/skills/` (or your project's `.claude/skills/`). Any agent that reads `SKILL.md` skills can use them the same way.

## What it checks

| Part | Lines | Covers |
|---|---|---|
| SEO | SEO-1 to SEO-19 | sitemap, robots, canonicals, one host, titles, descriptions, headings, alt text, schema, internal links, OG images, image weight, Lighthouse 90+, mobile, Search Console |
| Security | S1 to S14 | security headers, TLS, secrets, row-level security and access control, SQL injection / XSS / SSRF, CORS and CSRF, rate limits and WAF, dependency audit, encryption at rest, money and keys, AI agents, privacy and email law, security.txt, **tracking-pixel consent** |
| Production | R1 to R13 | queries and indexes, migrations, backups and restore, race conditions and locks, timeouts / retries / idempotency, cron and queues, webhooks, real-time, caching, performance budget, scaling plan, cost caps, DNS |
| Shipping | D1 to D5 | CI/CD, feature flags and rollback, config as code, health checks, **tests by layer (unit, integration, end to end, performance)** |
| Seeing problems | O1 to O4 | logs and tracing, metrics and alerts, SLOs and error budgets, on call, runbook, postmortems, chaos drill |

Not every line applies to every project. Tag the project with its profiles and only those lines run:

| Tag | Profile |
|---|---|
| W | Website |
| A | App or API with logins, user data or a database |
| M | Moves money, holds keys or trades |
| J | Background jobs, bots, scheduled pipelines |
| G | AI agent facing customers |
| X | Scale-only items (sharding, Kubernetes, multi-region): N/A with a trigger |

## Two checks most vibe-coded sites fail

**S14, tracking before consent.** If your Meta Pixel, Google Analytics or any third-party script loads before the visitor clicks Accept, your cookie banner is decoration. Sites like that are getting wiretap demand letters under California's Invasion of Privacy Act. The check: open the site in a private window, filter the network log to other domains, and confirm zero tracker requests before Accept.

**D5, tests by layer.** Push → unit → integration → end to end → performance → ship, with a red stage blocking the deploy. Agents make tests cheap to write, so there is no excuse to skip them.

## Files

```
.claude-plugin/plugin.json        plugin manifest
.claude-plugin/marketplace.json   lets /plugin marketplace add this repo
skills/ship-checklist/SKILL.md    the rules: every line, how to run it, coverage index
skills/ship-checklist/template.md the per-project docs/SHIP-CHECKLIST.md
skills/site-seo/SKILL.md          the 19 SEO checks (Part 1)
```

## Credits

The production list maps all 111 topics from [@nataindata](https://www.instagram.com/reel/DeeR2JyNrx-/)'s "Who's gonna tell vibe coders about..." video. S14 comes from [@dr.erinjacques](https://www.instagram.com/reel/De-pHkzxiuX/) on tracking-pixel lawsuits. D5 comes from [@rollins.io](https://www.instagram.com/reel/DfdtXLfAim5/), "Vibe coding is not software engineering." The SKILL.md has a coverage index mapping every topic to the line that handles it.

Built and used by Zenith on our own and our clients' projects.

## Not legal advice

The privacy and email lines are engineering checks. If your site handles health, finance or legal data, get your own lawyer to sign off.

## License

MIT. Use it, fork it, add your own lines. Pull requests that add a check with a clear "how to verify" are welcome.
