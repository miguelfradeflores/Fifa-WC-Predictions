# Coderoad World Cup 2026

Internal prediction game for the FIFA World Cup 2026. Submit score predictions before each match's 15-minute lock, earn points, and compete on a real-time leaderboard across 104 matches.

**Stack**: Next.js 15 · Supabase · Tailwind CSS · TypeScript 5

---

## Table of contents

1. [Prerequisites](#1-prerequisites)
2. [Clone and install](#2-clone-and-install)
3. [Spec Kit setup (optional)](#3-spec-kit-setup-optional)
4. [Supabase project setup](#4-supabase-project-setup)
5. [Environment variables](#5-environment-variables)
6. [Database setup](#6-database-setup)
7. [Seed match fixtures](#7-seed-match-fixtures)
8. [Run locally](#8-run-locally)
9. [Verify everything works](#9-verify-everything-works)
10. [Deployment (Vercel)](#10-deployment-vercel)
11. [Scripts reference](#11-scripts-reference)
12. [Scoring model](#12-scoring-model)
13. [CI](#13-ci)

---

## 1. Prerequisites

| Tool | Version | Check |
|---|---|---|
| Node.js | 20+ | `node -v` |
| pnpm | 9+ | `pnpm -v` |
| Supabase CLI | latest | `pnpm dlx supabase --version` |
| Git | any | `git --version` |

You also need:
- A **Supabase account** — [supabase.com](https://supabase.com) (free tier is enough)
- A **Google Cloud project** with OAuth credentials configured
- A **wc2026api.com bearer token** — only required when using the live provider; the mock provider works fully offline

---

## 2. Clone and install

```bash
git clone <repo-url> wc-2026
cd wc-2026
pnpm install
```

---

## 3. Spec Kit setup (optional)

This project was built with [Spec Kit](https://github.com/github/spec-kit) — an open-source toolkit for Spec-Driven Development. If you want to contribute new features or continue development using the same structured workflow, follow these steps to set it up locally.

> **Skip this section** if you only need to run or deploy the app.

### 3.1 Install prerequisites for Spec Kit

Spec Kit requires **Python 3.11+** and the `uv` package manager:

```bash
# Install uv (macOS / Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify
uv --version
```

For Windows or alternative install methods, see the [uv installation guide](https://docs.astral.sh/uv/getting-started/installation/).

### 3.2 Install the Specify CLI

Install the latest stable release (v0.10.1):

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.10.1
```

Verify the installation:

```bash
specify --version
```

To upgrade to a newer release in future:

```bash
specify self upgrade               # latest stable
specify self upgrade --tag v0.X.Y  # specific version
```

### 3.3 Initialize Spec Kit in this project

The `.specify/` directory is already committed to this repo, so initialization links Spec Kit to your local Claude Code agent — it does **not** overwrite any existing configuration:

```bash
# From the repo root
specify init . --integration claude --force
```

This writes the Spec Kit slash commands into `.claude/commands/` so they become available inside Claude Code.

Verify setup by opening Claude Code in the project directory — you should see these commands available:

| Command | Purpose |
|---|---|
| `/speckit-constitution` | Create or update project governing principles |
| `/speckit-specify` | Define a new feature (requirements + user stories) |
| `/speckit-clarify` | Clarify underspecified areas before planning |
| `/speckit-plan` | Create a technical implementation plan |
| `/speckit-tasks` | Break the plan into an ordered task list |
| `/speckit-implement` | Execute all tasks and build the feature |
| `/speckit-analyze` | Cross-artifact consistency check |
| `/speckit-checklist` | Generate a quality checklist for the feature |

### 3.4 Development workflow

When adding a new feature to this project, follow this sequence:

```
/speckit-specify   → describe what you want to build
/speckit-clarify   → resolve any ambiguities
/speckit-plan      → define the tech approach
/speckit-tasks     → generate the task breakdown
/speckit-implement → execute all tasks
```

All feature specs, plans, and tasks are saved to `specs/<feature-id>/` and committed alongside the code.

For full documentation, visit the [Spec Kit repo](https://github.com/github/spec-kit) or the [docs site](https://github.github.io/spec-kit/).

---

## 4. Supabase project setup

### 4.1 Create a project

1. Go to [supabase.com/dashboard](https://supabase.com/dashboard) → **New project**
2. Choose a region close to your users
3. Note the **Project ref** (short ID in the project URL)

### 4.2 Enable Google OAuth

1. In your Supabase project: **Authentication → Providers → Google**
2. Toggle **Enable Google provider**
3. Enter your **Client ID** and **Client Secret** from Google Cloud Console
4. Add the Supabase callback URL to your Google OAuth app's **Authorized redirect URIs**:
   ```
   https://<project-ref>.supabase.co/auth/v1/callback
   ```
5. For local development, also add:
   ```
   http://localhost:3000/auth/callback
   ```

### 4.3 Get your API keys

From **Project Settings → API**:

| Key | Where to find |
|---|---|
| Project URL | `https://<ref>.supabase.co` |
| Anon public key | "Project API keys" → `anon public` |
| Service role key | "Project API keys" → `service_role` (keep secret) |

---

## 5. Environment variables

Copy the example file and fill in the values:

```bash
cp .env.example .env.local
```

`.env.local` is gitignored — never commit it.

```bash
# ── Public (safe to expose to the browser) ──────────────────────────────────
NEXT_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon-key>

# ── Server-only (never bundled into client code) ─────────────────────────────
SUPABASE_SERVICE_ROLE_KEY=<service-role-key>

# '*' accepts any Google account.
# Set to 'yourcompany.com' to restrict sign-in to corporate email addresses.
ALLOWED_EMAIL_DOMAIN=*

# 'mock' loads fixtures offline (no API token needed).
# 'api'  fetches live data from wc2026api.com (requires WC2026_API_TOKEN).
RESULTS_PROVIDER=mock

# Required only when RESULTS_PROVIDER=api
WC2026_API_TOKEN=
```

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | ✅ | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | ✅ | Supabase anon JWT (safe for browser) |
| `SUPABASE_SERVICE_ROLE_KEY` | ✅ | Used by scripts and the auth callback |
| `ALLOWED_EMAIL_DOMAIN` | ✅ | `*` = any Google account; `company.com` = restrict to that domain |
| `RESULTS_PROVIDER` | ✅ | `mock` or `api` |
| `WC2026_API_TOKEN` | if `api` | Bearer token for the live fixture API |

---

## 6. Database setup

### 6.1 Link to your Supabase project

```bash
pnpm dlx supabase login
pnpm dlx supabase link --project-ref <ref>
```

### 6.2 Apply the migration

```bash
pnpm dlx supabase db push
```

Creates all tables (`profiles`, `matches`, `predictions`, `scores`), the `match_phase` enum, Row Level Security policies, and indexes.

### 6.3 Regenerate TypeScript types (optional)

```bash
pnpm gen:types
```

---

## 7. Seed match fixtures

Load the WC2026 fixtures into the `matches` table:

```bash
# Offline — uses the bundled seed file (no API token needed)
pnpm seed:matches

# Live API — fetches from wc2026api.com
RESULTS_PROVIDER=api pnpm seed:matches
```

Both commands are idempotent — safe to re-run. Re-run after each knockout round as newly-drawn teams replace `TBD` placeholders.

To validate that your DB matches the live API before seeding:

```bash
RESULTS_PROVIDER=api pnpm validate:api
```

---

## 8. Run locally

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) → sign in with Google → `/dashboard`.

> **Tip**: If sign-in redirects back with `?error=domain_not_allowed`, set `ALLOWED_EMAIL_DOMAIN=*` in `.env.local`.

---

## 9. Verify everything works

```bash
pnpm verify        # lint + typecheck + tests
pnpm test          # tests only
```

---

## 10. Deployment (Vercel)

1. Import the repository at [vercel.com/new](https://vercel.com/new)
2. Set the **Framework preset** to `Next.js`
3. Add all six environment variables from §4 under **Settings → Environment Variables**
4. Set `RESULTS_PROVIDER=api` and add `WC2026_API_TOKEN` for production
5. Add your Vercel URL as an authorized OAuth redirect URI in both Google Cloud and Supabase:
   ```
   https://<your-app>.vercel.app/auth/callback
   ```
6. Deploy — Vercel runs `pnpm build` automatically

---

## 11. Scripts reference

| Command | Description |
|---|---|
| `pnpm dev` | Start dev server at `localhost:3000` |
| `pnpm build` | Production build |
| `pnpm start` | Start production server (after `pnpm build`) |
| `pnpm test` | Run all tests once |
| `pnpm test:watch` | Tests in watch mode |
| `pnpm verify` | `lint + typecheck + test` — run before every PR |
| `pnpm seed:matches` | Load/refresh fixtures into the `matches` table |
| `pnpm snapshot:fixtures` | Pull live API → update the local seed file |
| `pnpm validate:api` | Dry-run diff between API fixtures and the DB |
| `pnpm score:match -- --match <uuid>` | Re-score a specific match |
| `pnpm record:result -- --match <ref> --home N --away N --actor "Name"` | Manually record a result and trigger scoring |
| `pnpm gen:types` | Regenerate Supabase TypeScript types |

---

## 12. Scoring model

Points are awarded per prediction after a match result is recorded:

| Rule | Points |
|---|---|
| Exact score (e.g. predicted 2-1, actual 2-1) | **5** |
| Correct winner or draw outcome | **2** |
| Correct goal count for one team | **1** |
| Unique prediction (only you predicted that exact score) | **2** |

The base total is multiplied by the match's tournament phase:

| Phase | Multiplier |
|---|---|
| Group stage | ×1.0 |
| Round of 16 | ×1.5 |
| Quarter-finals | ×2.0 |
| Semi-finals | ×3.0 |
| Third place | ×1.0 |
| Final | ×5.0 |

**Example**: predicting the Final result exactly with a unique score → `(5 + 2) × 5.0 = 35 points`.

---

## 13. CI

`.github/workflows/verify.yml` runs `pnpm verify` (lint + typecheck + test) on every PR targeting `main`. Merges are blocked if it fails.
