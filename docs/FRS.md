# intinat — Functional Requirements Specification (FRS)

**Repo:** `intinat-wa` (web application)
**Version:** 0.1 (draft)
**Owner:** Ajit
**Last updated:** 2026-10-23

---

## 1. Overview

intinat ("Internet is Under Attack") brings security threats and scams to everyday people in plain language. It combines:

1. **Landing page** — explains the promise and drives signups.
2. **Threat Hub (blog)** — weekly threat stories written at three depths (simple, more detail, technical), with embedded short videos.
3. **Web app tools** — a free AI scam checker first, then personalised alerts, breach lookup, family mode and learning modules.

**Core promise:** *What's happening on the internet this week, whether it affects you, and the one thing to do about it.*

Every piece of content answers three questions:
1. **What happened?**
2. **Am I affected?**
3. **What do I do in 2 minutes?**

## 2. Goals and non-goals

### Goals
- G1. Make current threats understandable for non-technical users in India first, then globally.
- G2. Give people a fast, free way to check whether a message or link is a scam.
- G3. Build an audience (newsletter + WhatsApp channel subscribers) that later supports alerts, sponsorships and B2B training.
- G4. Keep running costs near zero until traction is proven.

### Non-goals (for now)
- Not an antivirus, VPN or device-scanning app.
- Not a replacement for police or bank reporting; the app always points to official channels.
- No native mobile apps (responsive web only; PWA later).
- No paid plans until demand is proven (post-v2).

## 3. Personas

| ID | Persona | Description | Main need |
|---|---|---|---|
| P1 | Everyday user | Parent, shopkeeper, student, elderly person; uses WhatsApp, UPI, SMS | "Is this message a scam? What do I do?" |
| P2 | Entry-level tech user | Comfortable with apps, not security | Simple, practical protection steps |
| P3 | Tech-curious user | Developer, IT staff, student in tech | The technical "why" behind headlines |
| P4 | Family protector | Looks after parents' or children's devices | Alerts and nudges on others' behalf |
| P5 | Admin (owner) | Ajit / future editors | Publish fast, review AI drafts, moderate |

## 4. Information architecture (routes)

| Route | Area | Access |
|---|---|---|
| `/` | Landing | Public |
| `/about`, `/how-it-works`, `/for-business` | Marketing | Public |
| `/privacy`, `/terms`, `/disclaimer`, `/ai-transparency` | Legal / trust | Public |
| `/blog` | Threat Hub index | Public |
| `/blog/[slug]` | Threat post | Public |
| `/blog/category/[category]`, `/blog/tag/[tag]` | Listings | Public |
| `/check` | Scam checker | Public (rate-limited) |
| `/breach` | Breach lookup | Public (rate-limited) — v2 |
| `/scams` | Scam library | Public — v2 |
| `/learn`, `/learn/[lesson]` | Lessons & quizzes | Public — v2 |
| `/login`, `/signup` | Auth | Public — v2 |
| `/dashboard/*` | My stack, family, settings | Logged in — v2 |
| `/admin/*` | Back office | Admin role only |
| `/api/*` | API routes | Varies |
| `/rss.xml`, `/sitemap.xml` | Feeds / SEO | Public |

Language prefix (`/hi/...`) added in milestone M5.

---

## 5. Functional requirements

Priority: **P0** = MVP launch, **P1** = v1 (month 2), **P2** = v2 (month 3+).

### 5.1 Landing page (LND)

| ID | Requirement | Priority |
|---|---|---|
| LND-01 | Hero section stating the core promise with a primary CTA ("Check a suspicious message") and secondary CTA ("Get the weekly brief"). | P0 |
| LND-02 | Embedded mini scam checker on the landing page that submits to the same API as `/check`. | P0 |
| LND-03 | "This week's top threats" strip showing the 3 most recent posts flagged `featured`. | P0 |
| LND-04 | Newsletter signup form (email) and WhatsApp channel join button. | P0 |
| LND-05 | "How it works" section: What happened / Am I affected / What to do. | P0 |
| LND-06 | Trust section: no-storage promise for the checker, sources cited, not affiliated with any bank/government. | P0 |
| LND-07 | Footer with legal links, social links, and the India cyber fraud helpline (1930) and cybercrime.gov.in. | P0 |
| LND-08 | "For schools & companies" page with a contact/interest form (stores leads). | P1 |
| LND-09 | Fully responsive (mobile-first; most users arrive from WhatsApp/Instagram on phones). | P0 |

### 5.2 Threat Hub / blog (BLG)

| ID | Requirement | Priority |
|---|---|---|
| BLG-01 | Posts authored as MDX files in `apps/web/content/posts/` with typed frontmatter (see §7.1). | P0 |
| BLG-02 | Every post renders three fixed sections: **What happened**, **Am I affected?**, **What to do**. Build fails if a section is missing. | P0 |
| BLG-03 | Depth toggle on each post: **Simple / More detail / Technical**. Content blocks are tagged by depth; Simple is the default. Choice is remembered per browser. | P0 |
| BLG-04 | Severity badge: `act-now`, `be-aware`, `fyi`, with consistent colour and icon. | P0 |
| BLG-05 | Categories: Scams, Data breaches, Vulnerabilities, Privacy, AI threats, Kids & family. | P0 |
| BLG-06 | Product tags (e.g. `whatsapp`, `upi`, `android`, `iphone`, `chrome`, bank names) from a controlled list in code, used later for alert matching. | P0 |
| BLG-07 | Sources list at the end of every post (required frontmatter field, min 1). | P0 |
| BLG-08 | Embed YouTube / Instagram Reels by ID via an MDX component. | P0 |
| BLG-09 | Index page with filters by category, severity and tag; pagination. | P0 |
| BLG-10 | SEO: per-post metadata, Open Graph tags, JSON-LD `NewsArticle`, sitemap, RSS. | P0 |
| BLG-11 | Auto-generated share image per post (title, severity, "what to do" line, brand) for WhatsApp/Instagram, via Open Graph image generation. | P1 |
| BLG-12 | "Share to WhatsApp" button with pre-filled short alert text + link. | P0 |
| BLG-13 | Client-side search across titles, summaries and tags. | P1 |
| BLG-14 | Multilingual posts: same slug with language variants (English first, Hindi next). | P1 |
| BLG-15 | Related posts (same tags/category) at end of each post. | P1 |
| BLG-16 | Migration path to a headless CMS documented (not built) for when non-technical writers join. | P2 |

### 5.3 Scam checker (CHK)

| ID | Requirement | Priority |
|---|---|---|
| CHK-01 | Input: pasted text (max 5,000 chars) and/or a URL. | P0 |
| CHK-02 | Input: screenshot upload (PNG/JPG, max 5 MB), text read by the AI model. | P1 |
| CHK-03 | Output (structured): verdict (`likely_scam`, `suspicious`, `looks_safe`, `not_sure`), confidence (low/medium/high), list of red flags found, a plain-language explanation (≤ 120 words), recommended actions, and related Threat Hub posts. | P0 |
| CHK-04 | Every result shows the official route: India — call **1930** or report at **cybercrime.gov.in**; plus "contact your bank using the number on your card, not the message". | P0 |
| CHK-05 | **Privacy: pasted content and images are never stored by default.** Only anonymous metadata is logged (timestamp, verdict, input type, language). | P0 |
| CHK-06 | Before sending to the AI model, the server redacts likely sensitive data: OTP-like digit runs, card numbers, Aadhaar/PAN patterns, account numbers. The user sees a note that redaction happened. | P0 |
| CHK-07 | Optional opt-in checkbox: "Share this anonymised example to help others" → goes to moderation queue for the scam library. | P1 |
| CHK-08 | Prompt-injection safety: pasted content is treated strictly as data; the system prompt instructs the model to ignore instructions inside it; output must validate against a JSON schema or the request fails gracefully with "not sure". | P0 |
| CHK-09 | URL handling: the server does **not** visit the URL. It analyses the URL string (lookalike domains, shorteners, suspicious TLDs) and optionally checks a reputation API (e.g. Google Safe Browsing) when a key is configured. | P0 |
| CHK-10 | Rate limiting: anonymous users 10 checks/day per IP+device fingerprint; logged-in users 30/day. Friendly message when exceeded. | P0 |
| CHK-11 | Disclaimer on every result: the checker is guidance, not a guarantee. | P0 |
| CHK-12 | Response in the user's selected language (EN at P0, HI at P1). | P1 |
| CHK-13 | Copy/share result as a short WhatsApp message ("I checked this message on intinat — likely scam because…"). | P1 |
| CHK-14 | Feedback buttons on each result ("Was this right?") stored anonymously for quality review. | P1 |

### 5.4 Subscriptions and notifications (SUB)

| ID | Requirement | Priority |
|---|---|---|
| SUB-01 | Newsletter signup integrates with the chosen newsletter provider (Beehiiv or similar) via API; double opt-in handled by provider. | P0 |
| SUB-02 | WhatsApp channel join link (no API needed at P0). | P0 |
| SUB-03 | Transactional email (alerts, login links) via an email API (e.g. Resend). | P2 |
| SUB-04 | Web push notifications for "act-now" threats. | P2 |
| SUB-05 | Notification preferences per user: channels, frequency (instant for act-now, weekly digest otherwise), language. | P2 |

### 5.5 Accounts and personalisation (ACC)

| ID | Requirement | Priority |
|---|---|---|
| ACC-01 | Sign up / login via email magic link and Google. | P2 |
| ACC-02 | **My stack:** user selects their phone OS, apps, bank(s), browser, router brand from the controlled tag list. | P2 |
| ACC-03 | Personalised feed on dashboard: posts matching the user's stack first. | P2 |
| ACC-04 | Alert matching: when a post with severity `act-now` is published, users whose stack matches its tags get notified per their preferences. | P2 |
| ACC-05 | **Family mode:** add up to 5 family members (name + their stack); alerts say "Mom's phone may be affected". No accounts needed for family members. | P2 |
| ACC-06 | Account deletion with full data removal (self-service). | P2 |
| ACC-07 | Data export (JSON) of the user's own data. | P2 |

### 5.6 Breach lookup (BRC)

| ID | Requirement | Priority |
|---|---|---|
| BRC-01 | User enters an email; system checks a breach-data API (e.g. Have I Been Pwned) and lists breaches found with date and data types exposed. | P2 |
| BRC-02 | For each breach: plain-language "what this means" and recovery steps. | P2 |
| BRC-03 | Email is not stored; rate-limited; for logged-in users, optional monitoring of their own verified email only. | P2 |

### 5.7 Scam library (LIB)

| ID | Requirement | Priority |
|---|---|---|
| LIB-01 | Searchable library of anonymised scam examples (text or redacted image) with type, channel (SMS/WhatsApp/call/email), red flags and a link to the explainer post. | P2 |
| LIB-02 | Entries come from admin curation and approved community submissions (CHK-07). | P2 |
| LIB-03 | "Report a scam" form for logged-in users; goes to moderation. | P2 |

### 5.8 Learn (LRN)

| ID | Requirement | Priority |
|---|---|---|
| LRN-01 | Short lessons (3–5 minutes) authored in MDX, e.g. "Spot a phishing link", "Set up 2FA on WhatsApp". | P2 |
| LRN-02 | Interactive quizzes ("Is this real or a scam?") with explanations. | P2 |
| LRN-03 | Progress tracking for logged-in users; completion badge. | P2 |
| LRN-04 | Lesson bundles exportable as a group training pack (foundation for B2B). | P2 |

### 5.9 Admin / back office (ADM)

| ID | Requirement | Priority |
|---|---|---|
| ADM-01 | Admin area protected by role; only allow-listed emails get the admin role. | P1 |
| ADM-02 | **Threat intake queue:** items collected by the worker from feeds (§5.10), showing source, title, summary, detected products, suggested severity; actions: dismiss, mark as candidate. | P1 |
| ADM-03 | **AI drafting:** from a candidate item, generate a draft post (MDX with frontmatter and all three sections at three depths), a 45–60s Short script, and a share-card line. Drafts are saved for review, never auto-published. | P1 |
| ADM-04 | Export a draft as an MDX file (download or commit via GitHub API — commit is P2). | P1 |
| ADM-05 | Moderation queue for community scam submissions: approve, edit (further redaction), reject. | P2 |
| ADM-06 | Simple analytics dashboard: checks per day and verdict mix, signups, top posts, share clicks. | P1 |
| ADM-07 | Lead list from the "For business" form. | P1 |

### 5.10 Background jobs (JOB)

| ID | Requirement | Priority |
|---|---|---|
| JOB-01 | Feed ingestion every 6 hours from configured sources: CERT-In advisories, CISA Known Exploited Vulnerabilities, selected security news RSS feeds. Each source is a parser module. | P1 |
| JOB-02 | De-duplicate items across sources (by CVE ID, URL, and title similarity). | P1 |
| JOB-03 | Product detection: map item text to the controlled product tag list (rules first, AI fallback). | P1 |
| JOB-04 | Alert matching and dispatch for `act-now` posts (ACC-04). | P2 |
| JOB-05 | Weekly digest assembly (Friday): top posts of the week → newsletter draft. | P2 |
| JOB-06 | Data retention clean-up: delete anonymous check logs older than 90 days. | P1 |

---

## 6. Non-functional requirements (NFR)

| ID | Area | Requirement |
|---|---|---|
| NFR-01 | Performance | Landing and blog pages statically generated; Lighthouse ≥ 90 for performance, accessibility, SEO on mobile. |
| NFR-02 | Performance | Scam checker responds in < 8 s at p95; shows progress state immediately. |
| NFR-03 | Privacy | No storage of checker inputs by default (CHK-05). Privacy policy written in plain language. Analytics are cookieless/privacy-friendly. Comply with India's DPDP Act principles: purpose limitation, minimal data, deletion on request. |
| NFR-04 | Security | Secrets only in environment variables; never committed. Row-level security on all user tables. Admin routes server-side role-checked. Security headers (CSP, HSTS, X-Frame-Options). Dependency scanning in CI. |
| NFR-05 | Abuse | Rate limiting on all AI and lookup endpoints; bot protection (e.g. Turnstile) on forms and checker after first use. |
| NFR-06 | Cost | Per-request AI cost logged; daily global spend cap configurable via env; when hit, checker shows a friendly "busy" message. |
| NFR-07 | Accessibility | WCAG 2.1 AA: keyboard navigable, sufficient contrast, alt text, readable font sizes (min 16px body), simple language. |
| NFR-08 | i18n | All UI strings externalised from P0 (even if only English ships), so Hindi can be added without refactoring. |
| NFR-09 | Reliability | If the AI provider fails, checker returns "not sure" with general safety advice, never an error page. |
| NFR-10 | Content accuracy | Every post requires sources; AI drafts are always human-reviewed before publishing. |
| NFR-11 | Observability | Structured server logs; error tracking (e.g. Sentry); uptime check on `/` and `/api/health`. |
| NFR-12 | Testing | Unit tests for redaction, URL heuristics, feed parsers, frontmatter validation; E2E tests for landing → check flow and blog post rendering. |

## 7. Data model (high level)

### 7.1 Post frontmatter (MDX)

```yaml
title: string              # required
slug: string               # required, unique
summary: string            # required, ≤ 160 chars
date: YYYY-MM-DD           # required
updated: YYYY-MM-DD        # optional
severity: act-now | be-aware | fyi
category: scams | breaches | vulnerabilities | privacy | ai-threats | kids-family
tags: [product-tag, ...]   # from controlled list
featured: boolean
lang: en | hi
video: { platform: youtube | instagram, id: string }   # optional
shareLine: string          # one-line "what to do", used on share cards
sources: [{ title: string, url: string }]   # min 1
```

### 7.2 Database tables (Postgres)

| Table | Purpose | Phase |
|---|---|---|
| `check_events` | Anonymous checker metadata: id, created_at, input_type, verdict, confidence, lang, latency_ms, cost_estimate. **No content.** | P0 |
| `check_feedback` | check_event_id, was_correct, created_at | P1 |
| `business_leads` | name, org, email, type (school/company/society), message, created_at | P1 |
| `feed_items` | source, external_id, url, title, summary, published_at, detected_tags, suggested_severity, status (new/dismissed/candidate/drafted) | P1 |
| `drafts` | feed_item_id, mdx, short_script, share_line, created_at, status | P1 |
| `scam_submissions` | redacted_text, image_path, channel, status, created_at | P1 |
| `profiles` | user_id, display_name, lang, role (user/admin) | P2 |
| `user_stack` | user_id, tag | P2 |
| `family_members` | id, user_id, name, tags[] | P2 |
| `notification_prefs` | user_id, channels, frequency | P2 |
| `alerts_sent` | user_id, post_slug, channel, sent_at | P2 |
| `lesson_progress` | user_id, lesson_slug, completed_at, score | P2 |

## 8. External integrations

| Integration | Use | Phase |
|---|---|---|
| AI model API (Anthropic Claude) | Scam checker, drafting, translation | P0 / P1 |
| Supabase (Postgres + Auth + Storage) | Database, auth, image storage | P0 (DB) / P2 (auth) |
| Upstash Redis (or equivalent) | Rate limiting, spend counter | P0 |
| Newsletter provider (Beehiiv or similar) | Subscriber management | P0 |
| Cloudflare Turnstile | Bot protection | P0 |
| Google Safe Browsing | URL reputation (optional key) | P0 optional |
| Privacy-friendly analytics (Plausible / Vercel Analytics) | Traffic | P0 |
| Sentry | Error tracking | P0 |
| CERT-In, CISA KEV, RSS feeds | Threat intake | P1 |
| Have I Been Pwned API | Breach lookup | P2 |
| Resend | Transactional email | P2 |

## 9. Milestones and acceptance criteria

Build strictly in this order. Each milestone ends with a working, deployable app.

### M0 — Scaffold and tooling
- Turborepo + pnpm monorepo with `apps/web`, `apps/worker`, `packages/{ui,db,ai,feeds,i18n,config}`.
- Next.js (App Router, TypeScript strict), Tailwind, lint/format, Vitest, Playwright.
- `.env.example` with every variable documented; env validation fails fast on missing required vars.
- GitHub Actions CI: install, lint, typecheck, test, build.
- `/api/health` returns 200.
- **Accept when:** `pnpm dev` runs, CI passes on a PR, a preview deployment works.

### M1 — Design system and landing page
- Brand tokens (colours, type, spacing) in `packages/ui`; light/dark mode.
- Components: Button, Input, Card, Badge (severity), Alert, Section, Header, Footer.
- Landing page per LND-01, 03 (with placeholder data), 04–07, 09; legal page shells.
- All strings via `packages/i18n`.
- **Accept when:** landing page scores Lighthouse ≥ 90 on mobile and matches LND P0 items.

### M2 — Threat Hub MVP
- MDX pipeline with frontmatter schema validation (build fails on invalid posts).
- Post page with three required sections, depth toggle, severity badge, sources, video embed, WhatsApp share.
- Index with filters and pagination; category and tag pages; RSS, sitemap, JSON-LD.
- 3 sample posts covering different severities.
- Landing "top threats" strip wired to real posts.
- **Accept when:** BLG P0 items pass; an invalid post breaks the build with a clear message.

### M3 — Scam checker v1
- `/check` page and landing mini-checker sharing one component.
- `/api/check`: validation → redaction → URL heuristics → AI call with strict JSON schema → response.
- Rate limiting, Turnstile, spend cap, graceful "not sure" fallback.
- `check_events` logging (metadata only).
- Unit tests: redaction patterns, URL heuristics, schema validation, prompt-injection test cases.
- **Accept when:** CHK P0 items pass; a test confirms no input content reaches the database or logs.

### M4 — MVP launch readiness
- Newsletter signup wired to provider; WhatsApp join link.
- Analytics, Sentry, security headers, final legal pages (privacy, terms, disclaimer, AI transparency).
- E2E tests: landing → check → result; blog index → post → share.
- **Accept when:** all P0 requirements pass and production deploy is live on the domain.

### M5 — v1 content features
- Screenshot input (CHK-02), shareable result (CHK-13), feedback (CHK-14).
- OG share images (BLG-11), search (BLG-13), related posts (BLG-15).
- Hindi: i18n routing, Hindi UI strings, Hindi post variants, checker replies in Hindi.
- **Accept when:** a Hindi user can read a Hindi post and get a Hindi checker result end to end.

### M6 — Admin and intake
- Admin auth (allow-listed emails), ADM-01, 02, 03, 04, 06, 07.
- Worker with JOB-01, 02, 03, 06; scheduled via GitHub Actions cron or platform cron.
- "For business" form (LND-08) and community submission opt-in (CHK-07).
- **Accept when:** a new CERT-In or KEV item appears in the intake queue and can be turned into a reviewed MDX draft.

### M7 — Accounts and alerts
- ACC-01 to ACC-04, ACC-06, ACC-07; SUB-03, SUB-05; JOB-04.
- **Accept when:** a user with "WhatsApp + Android" in their stack receives an email for a new act-now post tagged with either.

### M8 — Breach lookup, family mode, library, learn
- BRC-*, ACC-05, LIB-*, LRN-*, SUB-04, JOB-05.
- **Accept when:** each feature's requirements pass with tests.

## 10. Out of scope (until revisited)
- Native iOS/Android apps.
- Paid subscriptions and payments.
- Device scanning or browser extensions.
- Community forums or comments.
- Phone-number breach lookup (data source and legality to be evaluated).

## 11. Open questions
1. Final consumer brand name vs. "intinat" as umbrella domain.
2. Newsletter provider choice (Beehiiv vs. Substack vs. self-hosted).
3. Whether the blog moves to a headless CMS before Hindi launch.
4. Which regional languages follow Hindi (Marathi? Tamil? Bengali?).
5. Legal review of disclaimer and privacy policy before launch.
