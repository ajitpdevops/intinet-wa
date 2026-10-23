# intinet-wa

Web application for **intinat** ("Internet is Under Attack"): plain-language security threats and scams for everyday people. It contains the landing page, the Threat Hub blog, and tools, starting with a free AI scam checker.

- Spec (source of truth): [`docs/FRS.md`](docs/FRS.md)
- Progress and task checklist: [`docs/PROGRESS.md`](docs/PROGRESS.md)
- How Claude works in this repo: [`CLAUDE.md`](CLAUDE.md)

## Status

Milestone **M0 — Scaffold and tooling** is in progress. The monorepo (Turborepo + pnpm workspaces) is set up; the web app, packages, tests and Docker Compose stack are being added task by task. See `docs/PROGRESS.md` for what is done.

## Tech stack

Turborepo + pnpm · Next.js (App Router) + TypeScript + Tailwind · Drizzle + Postgres (Supabase in production) · Upstash Redis · Anthropic Claude API · Vitest + Playwright · Docker Compose for local dev, tests and CI · Vercel for hosting.

## Repository layout

```
apps/web        Next.js app (landing, blog, checker, admin)
apps/worker     scheduled jobs (feed ingestion, alerts, clean-up)
packages/*      ui, db, ai, feeds, i18n, config
docker/         Dockerfiles and local stand-ins (e.g. the fake AI API)
compose.yaml    every service, runnable locally and in CI
docs/           FRS, progress log, decision records
```

## Testing locally on Ubuntu

Everything is designed to run with Docker Compose, so you don't need any cloud accounts or API keys to test. The steps below are for Ubuntu 22.04 / 24.04.

### 1. Install prerequisites (once)

**Git**

```bash
sudo apt update
sudo apt install -y git curl ca-certificates
```

**Docker Engine + Compose plugin** (official Docker apt repository):

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# run docker without sudo (log out and back in afterwards)
sudo usermod -aG docker $USER
```

Check it works:

```bash
docker run --rm hello-world
docker compose version
```

**Node.js 22 + pnpm** (only needed to run things outside Docker, e.g. fast unit tests or `pnpm dev`):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install        # reads .nvmrc (Node 22)
corepack enable    # provides the pnpm version pinned in package.json
pnpm --version
```

### 2. Get the code

```bash
git clone https://github.com/ajitpdevops/intinet-wa.git
cd intinet-wa
git checkout FRS-001   # current working branch until it is merged to main
```

### 3. What you can test today

Only the workspace skeleton exists so far:

```bash
pnpm install                       # installs workspace tooling (turbo)
pnpm turbo run build --dry=json    # shows Turborepo resolves the workspace
pnpm lint                          # runs (no packages to lint yet)
```

### 4. Full local test flow (available once M0 is complete)

These commands are being added during M0; this is how you'll use them.

**Run the whole app in containers**

```bash
pnpm docker:up       # or: docker compose up --build
```

Then open:

- http://localhost:3000 — the web app
- http://localhost:3000/api/health — should return `200` with `{"status":"ok"}`

The stack starts Postgres, runs migrations, starts Redis (with an Upstash-compatible proxy), a fake AI API (`ai-mock`), the web app and the worker. It uses only the committed defaults in `.env.docker`, so no secrets are needed. External services (Turnstile, Sentry, analytics, newsletter) use test keys or are switched off.

**Run the tests in containers** (the same commands CI runs):

```bash
pnpm docker:test     # unit + integration tests against real Postgres and Redis
pnpm docker:e2e      # Playwright browser tests against the web container
```

**Stop and clean up**

```bash
pnpm docker:down             # stop containers
docker compose down -v       # also delete the local database volume
```

**Faster loop without Docker** (for unit tests and UI work):

```bash
pnpm install
pnpm dev          # Next.js dev server on http://localhost:3000
pnpm test         # Vitest unit tests
pnpm lint
pnpm typecheck
pnpm build
```

Integration tests need Postgres and Redis, so start just those with `docker compose up -d db redis redis-http ai-mock` and run them from your machine, or use `pnpm docker:test`.

### 5. Troubleshooting

| Problem | Fix |
|---|---|
| `permission denied ... /var/run/docker.sock` | You aren't in the `docker` group yet. Run `sudo usermod -aG docker $USER`, then log out and back in. |
| `port is already allocated` (3000, 5432, 6379) | Another service is using the port. Stop it (e.g. `sudo systemctl stop postgresql`) or change the host port in `compose.yaml`. |
| Old data or a broken database | `docker compose down -v` then `pnpm docker:up`. |
| `pnpm: command not found` | Run `corepack enable` (Node 22 must be active: `nvm use`). |
| Build is slow the first time | Docker is downloading images and dependencies; later runs use the cache. |

## How we work

- One milestone and one task at a time, in the order in `docs/PROGRESS.md`.
- Test-driven: tests are written first and must fail before the code is written.
- Every change must pass lint, typecheck, tests and build, both locally and in Docker Compose.
- Scam checker input is never stored. See the privacy rules in `CLAUDE.md`.
