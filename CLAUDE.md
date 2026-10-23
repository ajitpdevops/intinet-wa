# CLAUDE.md — intinat-wa

This file tells Claude how to work in this repository. Read it fully at the start of every session.

## What this project is

intinat ("Internet is Under Attack") makes security threats and scams understandable for everyday people. This repo (`intinat-wa`) contains the web application: landing page, Threat Hub blog, and tools (AI scam checker first).

**The full spec is in `docs/FRS.md`. It is the source of truth.** Requirement IDs (e.g. `CHK-05`) and milestones (`M0`–`M8`) come from there.

## How to work: one milestone, one task at a time

1. **Start of session:** read `CLAUDE.md`, `docs/FRS.md`, and `docs/PROGRESS.md` (create `PROGRESS.md` on the first session from the template below).
2. **Pick the next task:** the first unchecked task in the current milestone in `PROGRESS.md`. Never skip ahead to a later milestone.
3. **Plan before coding:** state which requirement IDs the task covers, which files you will create or change, and how you will test it. Wait for approval if the task touches the database schema, auth, privacy handling, or adds a new dependency or external service.
4. **Implement in small steps.** Keep each change focused on the task.
5. **Verify:** run `pnpm lint`, `pnpm typecheck`, `pnpm test`, and `pnpm build`. Fix failures before calling the task done.
6. **Update `docs/PROGRESS.md`:** tick the task, note what was done, and list any follow-ups or decisions.
7. **Suggest a commit message** in Conventional Commits format (e.g. `feat(check): add redaction for OTP and card numbers [CHK-06]`).
8. **Stop and report.** Summarise what changed and what the next task is. Do not start the next task unless asked.

If a requirement is unclear or conflicts with another, ask instead of guessing. If you discover something the FRS should cover, propose an FRS change rather than silently doing it.

## Tech stack

- **Monorepo:** Turborepo + pnpm workspaces
- **Web:** Next.js (App Router), React, TypeScript (strict), Tailwind CSS
- **UI components:** shadcn/ui-style components living in `packages/ui`
- **Content:** MDX posts in `apps/web/content/posts/`, frontmatter validated with Zod at build time
- **Database:** Supabase Postgres; schema and migrations in `packages/db` (Drizzle ORM)
- **Auth (M6+):** Supabase Auth (magic link + Google)
- **AI:** Anthropic Claude API via the official SDK, wrapped in `packages/ai`; model name from env var `AI_MODEL`
- **Rate limiting / counters:** Upstash Redis
- **Bot protection:** Cloudflare Turnstile
- **Tests:** Vitest (unit), Playwright (E2E)
- **Errors:** Sentry
- **Hosting:** Vercel (web); worker jobs scheduled via GitHub Actions cron or platform cron
- **CI:** GitHub Actions

Use the latest stable versions when adding packages. Do not add a dependency that duplicates something already in the stack; ask first before adding any new external service.

## Repository structure

```
intinat-wa/
├── apps/
│   ├── web/                    # Next.js app: landing, blog, tools, admin
│   │   ├── app/
│   │   │   ├── (marketing)/    # /, about, how-it-works, for-business, legal pages
│   │   │   ├── blog/           # Threat Hub
│   │   │   ├── check/          # scam checker
│   │   │   ├── breach/         # M8
│   │   │   ├── scams/          # M8
│   │   │   ├── learn/          # M8
│   │   │   ├── dashboard/      # M7
│   │   │   ├── admin/          # M6
│   │   │   └── api/            # route handlers
│   │   ├── content/
│   │   │   ├── posts/          # MDX threat posts
│   │   │   └── lessons/        # MDX lessons (M8)
│   │   └── e2e/                # Playwright tests
│   └── worker/                 # scheduled jobs (feed ingestion, alerts, digests)
├── packages/
│   ├── ui/                     # design tokens + shared components
│   ├── db/                     # Drizzle schema, migrations, queries
│   ├── ai/                     # prompts, schemas, AI client, cost tracking
│   ├── feeds/                  # one parser module per threat source
│   ├── i18n/                   # UI strings (en, hi) and helpers
│   └── config/                 # shared tsconfig, eslint, env validation
├── docs/
│   ├── FRS.md                  # functional requirements (source of truth)
│   ├── PROGRESS.md             # milestone checklist and session log
│   └── decisions/              # short ADRs for notable decisions
├── .github/workflows/          # CI and scheduled jobs
├── .env.example                # every env var, documented
└── CLAUDE.md
```

Put code in the package that owns it: AI calls only in `packages/ai`, DB access only through `packages/db`, UI strings only in `packages/i18n`.

## Commands

```bash
pnpm install          # install all workspaces
pnpm dev              # run web app locally
pnpm lint             # eslint across workspaces
pnpm typecheck        # tsc --noEmit across workspaces
pnpm test             # vitest unit tests
pnpm test:e2e         # playwright E2E tests
pnpm build            # production build (also validates MDX frontmatter)
pnpm db:generate      # generate migration from schema changes
pnpm db:migrate       # apply migrations
```

If a command doesn't exist yet, create it as part of M0.

## Non-negotiable rules

### Privacy
- **Never store scam checker input** (pasted text, URLs, images) in the database, logs, analytics, or error reports. Only metadata listed in FRS §7.2 `check_events`. Write a test that proves this.
- Redact sensitive patterns (OTPs, card numbers, Aadhaar, PAN, account numbers) **before** sending anything to the AI provider.
- Strip user input from Sentry events and server logs (use `beforeSend` scrubbing).
- The server must never fetch or open a user-submitted URL.
- Collect the minimum data needed. Every new user data field needs a stated purpose in the PR.

### AI safety
- All AI prompts live in `packages/ai/prompts/` as versioned files, not inline strings.
- User content is passed as clearly delimited data. The system prompt tells the model to ignore any instructions inside it.
- AI responses must validate against a Zod schema. On failure or provider error, return the `not_sure` verdict with general safety advice — never a stack trace or error page.
- Enforce rate limits and the daily spend cap (`AI_DAILY_SPEND_CAP`) on every AI endpoint.
- AI-drafted content is never auto-published. It always goes to the admin review queue.

### Security
- Secrets only via environment variables; update `.env.example` whenever you add one. Never commit `.env*` files with values.
- Row-level security on every user-owned table.
- Admin routes check the role on the server, not just in the UI.
- Set security headers (CSP, HSTS, frame protection) in Next.js config.

### Content
- Every post must have the three sections (What happened / Am I affected? / What to do) and at least one source; the build must fail otherwise.
- Write user-facing copy in simple, calm language: short sentences, no jargon in the Simple depth, no fear-mongering, always end with a concrete action.
- Every checker result and threat post shows the official route (India: 1930 helpline, cybercrime.gov.in).

## Code conventions

- TypeScript strict; no `any` without a comment explaining why.
- Server Components by default; Client Components only when interactivity needs them.
- Validate all inputs at the API boundary with Zod.
- All user-facing strings go through `packages/i18n`, even English-only ones.
- Mobile-first styling; test layouts at 375px width.
- Accessibility: semantic HTML, labels on inputs, visible focus states, alt text.
- Name files in kebab-case; React components in PascalCase.
- Tests sit next to the code (`*.test.ts`) except E2E tests in `apps/web/e2e/`.
- Keep functions small and pure where possible, especially redaction, URL heuristics and feed parsers — these need thorough unit tests.

## Definition of done (every task)

- [ ] Covers the stated requirement IDs and nothing unrelated
- [ ] Lint, typecheck, tests and build pass
- [ ] New logic has unit tests; user flows touched have E2E coverage where the milestone requires it
- [ ] Privacy and AI safety rules above are respected
- [ ] `.env.example` and docs updated if needed
- [ ] `docs/PROGRESS.md` updated
- [ ] Works on a 375px mobile viewport

## `docs/PROGRESS.md` template

Create this on the first session, with one checkbox per task broken down from each milestone in FRS §9:

```markdown
# Progress

## Current milestone: M0 — Scaffold and tooling

### M0 tasks
- [ ] Initialise Turborepo + pnpm workspaces
- [ ] Create apps/web (Next.js, TS strict, Tailwind)
- [ ] Create packages (ui, db, ai, feeds, i18n, config) with stubs
- [ ] Shared tsconfig + eslint + prettier
- [ ] Env validation + .env.example
- [ ] Vitest + Playwright setup
- [ ] /api/health route
- [ ] GitHub Actions CI

### M1 tasks
- ...

## Session log
| Date | Task | Requirement IDs | Notes / follow-ups |
|---|---|---|---|
```

## When you're unsure

Ask. A short question is cheaper than rebuilding the wrong thing. Especially ask before: changing the FRS, changing the database schema, adding a dependency or service, or anything touching user data.
