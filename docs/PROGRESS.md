# Progress

## Current milestone: M0 — Scaffold and tooling

### M0 tasks
- [x] Initialise Turborepo + pnpm workspaces
- [ ] Create apps/web (Next.js, TS strict, Tailwind)
- [ ] Create packages (ui, db, ai, feeds, i18n, config) with stubs; apps/worker stub
- [ ] Shared tsconfig + eslint + prettier
- [ ] Env validation + .env.example + .env.docker
- [ ] Vitest + Playwright setup (unit + integration projects)
- [ ] /api/health route (test first)
- [ ] Dockerfiles (web, worker) + compose.yaml (db, migrate, redis, redis-http, ai-mock, web, worker, test, e2e)
- [ ] pnpm docker:* scripts
- [ ] GitHub Actions CI running lint/typecheck/build + docker:test + docker:e2e + dependency scan
- [ ] Vercel preview deployment

### M1 tasks — Design system and landing page
- [ ] Brand tokens (colour, type, spacing) + light/dark mode in packages/ui
- [ ] Components: Button, Input, Card, Badge (severity), Alert, Section, Header, Footer
- [ ] packages/i18n: string catalogue + helper; all landing strings through it (NFR-08)
- [ ] Landing: hero + CTAs (LND-01)
- [ ] Landing: top threats strip with placeholder data (LND-03)
- [ ] Landing: newsletter form + WhatsApp join (UI only) (LND-04)
- [ ] Landing: how it works (LND-05), trust section (LND-06), footer with 1930 / cybercrime.gov.in (LND-07)
- [ ] Legal page shells (privacy, terms, disclaimer, ai-transparency); about, how-it-works
- [ ] Responsive at 375px + Lighthouse ≥ 90 mobile (LND-09, NFR-01, NFR-07)

### M2 tasks — Threat Hub MVP
- [ ] Frontmatter Zod schema + controlled tag/category lists (BLG-01, 05, 06; §7.1)
- [ ] MDX pipeline; build fails on invalid post or missing section/source (BLG-02, BLG-07)
- [ ] Post page: three sections, depth toggle remembered per browser (BLG-02, BLG-03)
- [ ] Severity badge, sources list, video embed component (BLG-04, 07, 08)
- [ ] WhatsApp share button (BLG-12)
- [ ] Index with filters + pagination; category and tag pages (BLG-09)
- [ ] SEO: metadata, OG, JSON-LD NewsArticle, sitemap, RSS (BLG-10)
- [ ] 3 sample posts with different severities
- [ ] Landing top-threats strip wired to featured posts (LND-03)

### M3 tasks — Scam checker v1
- [ ] Input schema + validation (CHK-01)
- [ ] Redaction module (CHK-06)
- [ ] URL heuristics, no fetching; optional Safe Browsing client (CHK-09)
- [ ] packages/ai: versioned prompt, client, output Zod schema, not_sure fallback (CHK-03, CHK-08, NFR-09)
- [ ] ai-mock fixtures incl. prompt-injection cases
- [ ] Rate limiter with salted daily IP hash (CHK-10) + Turnstile (NFR-05)
- [ ] Daily spend cap + per-request cost logging (NFR-06)
- [ ] check_events table + metadata-only logging; restricted DB role (CHK-05, NFR-04)
- [ ] /api/check route wiring
- [ ] Checker UI shared by /check and landing mini-checker; official route + disclaimer (CHK-04, CHK-11, LND-02)
- [ ] Privacy test: input never reaches DB, logs or Sentry

### M4 tasks — MVP launch readiness
- [ ] Newsletter provider integration (SUB-01) — needs provider decision
- [ ] WhatsApp channel link (SUB-02)
- [ ] Analytics (cookieless) — needs tool decision
- [ ] Sentry with input scrubbing (NFR-11)
- [ ] Security headers + CSP allow-list (NFR-04)
- [ ] Final legal pages
- [ ] E2E: landing → check → result; blog index → post → share
- [ ] Production deploy on domain + uptime checks

### M5 tasks — v1 content features
- [ ] Screenshot input (CHK-02)
- [ ] Share result (CHK-13), feedback (CHK-14)
- [ ] OG share images (BLG-11), search (BLG-13), related posts (BLG-15)
- [ ] Hindi routing, UI strings, post variants, checker replies (BLG-14, CHK-12)

### M6 tasks — Admin and intake
- [ ] Admin auth with allow-list (ADM-01)
- [ ] Feed parsers: CERT-In, CISA KEV, RSS (JOB-01)
- [ ] De-dup + product detection (JOB-02, JOB-03)
- [ ] Intake queue (ADM-02), AI drafting (ADM-03), MDX export (ADM-04)
- [ ] Analytics dashboard (ADM-06), leads (ADM-07, LND-08)
- [ ] Community submission opt-in (CHK-07)
- [ ] Retention clean-up job (JOB-06) + scheduling

### M7 tasks — Accounts and alerts
- [ ] Supabase Auth: magic link + Google (ACC-01) in Compose
- [ ] My stack (ACC-02), personalised feed (ACC-03)
- [ ] Notification prefs (SUB-05), transactional email via Resend + mailpit locally (SUB-03)
- [ ] Alert matching + dispatch (ACC-04, JOB-04)
- [ ] Account deletion (ACC-06), data export (ACC-07)
- [ ] Logged-in rate limit 30/day (CHK-10)

### M8 tasks — Breach, family, library, learn
- [ ] Breach lookup (BRC-*)
- [ ] Family mode (ACC-05)
- [ ] Scam library + report form (LIB-*)
- [ ] Lessons, quizzes, progress (LRN-*)
- [ ] Web push (SUB-04), weekly digest (JOB-05)

## Decisions
- 2026-10-23: Docker Compose parity for dev/test/CI; TDD for all production code (FRS NFR-12, NFR-13).
- 2026-10-23: Approved images: postgres:17, redis:7, hiett/serverless-redis-http, official Playwright image, node:22-alpine.
- 2026-10-23: AI provider is mocked in tests/CI via an `ai-mock` container (real SDK, `ANTHROPIC_BASE_URL`).

## Session log
| Date | Task | Requirement IDs | Notes / follow-ups |
|---|---|---|---|
| 2026-10-23 | FRS review; CLAUDE.md + FRS v0.2; PROGRESS.md | NFR-04, NFR-12, NFR-13, CHK-10 | Open before M4: newsletter provider, analytics tool. |
| 2026-10-23 | Initialise Turborepo + pnpm workspaces | M0 | Root package.json (pnpm 10.28, Node ≥22), pnpm-workspace.yaml (apps/*, packages/*), turbo.json pipeline, turbo 2.11.10, .gitignore, .nvmrc. Red: `pnpm turbo run build --dry` failed with no manifest; green: workspace resolves. Follow-up: set TURBO_TELEMETRY_DISABLED=1 in CI/containers. |
