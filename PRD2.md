# PRD2 — Coderoad World Cup 2026 (Post-Reengineering)

**Status:** Draft · supersedes [`PRD.md`](./PRD.md) for the as-built product
**Owner:** Engineering / People & Culture
**Last updated:** 2026-05-08

---

## 1. Why this document exists

[`PRD.md`](./PRD.md) was the initial three-hour-MVP brief. Since then we've:

1. Run the full Spec Kit pipeline (`/speckit-constitution` →
   `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` →
   `/speckit-tasks` → `/speckit-implement`) and built the application
   skeleton.
2. Replaced the placeholder visual language with a complete dark-mode
   design system (the **Coderoad World Cup** identity).
3. Wired a real fixture provider against [wc2026api.com](https://api.wc2026api.com/docs).
4. Decided on a four-tab UX (Dashboard / Predictions / Ranking / Rules)
   that reorganizes the original five dashboard surfaces.
5. Moved several UX details — scoring constants, badges, leaderboard
   columns, tier system, prize structure — that **conflict with the
   2026-05-08 clarifications**. §10 lists every drift item explicitly.

This PRD captures the product as currently envisioned in the design
system (images 10–16 in chat — see §6). Implementation status as of
this writing is in §11.

---

## 2. Goals & success criteria

Same as PRD.md §6 — unchanged:

- ≥70% of Coderoad employees registered before the tournament begins
- ≥60% participation rate among registered employees
- ≥90% of matches with at least one prediction recorded
- Leaderboard refreshed within 60 minutes of any official result
- **Zero** critical scoring errors over the course of the tournament

---

## 3. Stack (as built)

| Concern | Choice | Notes |
|---|---|---|
| Framework | Next.js 15 (App Router, Server Actions) | Single deployment unit |
| Runtime | Node 20+ | Vercel-compatible |
| Language | TypeScript 5 (`strict`, `noUncheckedIndexedAccess`) | Constitution II |
| UI | React 19 + Tailwind CSS 3 | Dark theme tokens in `tailwind.config.ts` |
| Auth | Supabase Auth — Google OAuth | Domain-restricted via callback (FR-001) |
| Database | Supabase Postgres | Four tables: `profiles`, `matches`, `predictions`, `scores` |
| Security | Supabase Row Level Security | Lock-time enforcement at the DB boundary |
| Match data | [`api.wc2026api.com`](https://api.wc2026api.com/docs) | Bearer-token-authenticated REST |
| Tests | Vitest | Mandatory: scoring engine + 15-min lock predicate |
| Lint | ESLint + `eslint-config-next` | Default Next preset |

### 3.1 Repository layout

```
src/
  app/                    Next.js App Router
    auth/{sign-in,callback}
    (dashboard)/{matches/[matchId], leaderboard, profile}
  features/
    matches/              data, lock predicate, MatchCard/MatchList, Locked indicator
    predictions/          Zod schemas, Server Action, PredictionForm
    scoring/              rules, pure engine, apply orchestrator
    rankings/             read-time leaderboard queries + Leaderboard UI
    badges/               end-of-tournament computation + BadgeList UI
    providers/            MatchResultsProvider interface, MockProvider, ApiProvider
    shell/                TopNav (Client Component, pill tabs)
  lib/
    env.ts                Zod-parsed env (split public / server)
    supabase/             server, client, service-role variants + Database types
supabase/
  migrations/0001_init.sql
  seed.fixtures.json      mock data fallback
scripts/
  seed-matches.ts         load fixtures via service role
  record-result.ts        manual fallback CLI (Constitution VII)
  compute-badges.ts       end-of-tournament badge computation
specs/001-prediction-game/   spec, plan, contracts, tasks
```

### 3.2 Routing model

| Path | Type | Auth |
|---|---|---|
| `/` | Server redirect to `/auth/sign-in` or `/dashboard` | n/a |
| `/auth/sign-in` | Public | — |
| `/auth/callback` | Public OAuth handler | issues session, runs domain check |
| `/dashboard` | Authenticated | "Matches" tab in nav |
| `/leaderboard` | Authenticated | "Ranking" tab |
| `/profile` | Authenticated | accessed via avatar click (no nav tab in mockups) |
| `/matches/[matchId]` | Authenticated | match detail + prediction form |

The mockups (image 12+) use a 4-tab nav: **Dashboard / Predictions /
Ranking / Rules**. Current routes are 3 — **see §10 for reconciliation**.

---

## 4. Visual design system

### 4.1 Palette (`tailwind.config.ts`)

| Token | Value | Use |
|---|---|---|
| `accent.DEFAULT` | `#930BF5` | Primary CTAs, brand mark, "you" highlight on tables |
| `accent.secondary` | `#00E676` | Success states, "Spot On!" callouts, "+pts" deltas |
| `accent.tertiary` | `#B367FD` | Lighter purple for emphasis / progress fills |
| `accent.warn` | `#F59E0B` | Locked match indicator |
| `accent.danger` | `#EF4444` | Validation errors |
| `surface.DEFAULT` | `#0A0B14` | Page background |
| `surface.card` | `#14161F` | Cards, inputs, raised panels |
| `surface.dark` | `#06070C` | Hero panels, score panels |
| `ink.DEFAULT` | `#F4F5F7` | Primary text on dark surface |
| `ink.muted` | `#9CA0AB` | Secondary text |
| `ink.inverse` | `#0A0B14` | Text on flipped (light) elements |

### 4.2 Typography (loaded via `next/font/google`)

| Family | Stack token | Use |
|---|---|---|
| **Lexend** | `font-headline` | Page titles, big numerals (1,450 / 42 / score wheel) |
| **Inter** | `font-sans` (default) | Body copy |
| **Space Grotesk** | `font-label` | Uppercase tracked labels (`UPCOMING`, `GROUP A · OCT 15`, `+150 PTS`) |

### 4.3 Component primitives

The MVP intentionally avoids a Card/Button/Stat primitive layer
(Constitution I, IV) — components inline Tailwind utility classes.
The **single** extracted UI primitive is `<Locked />`
(`src/features/matches/ui/Locked.tsx`), reused across the prediction
form and the post-lock reveal panel.

---

## 5. Information architecture

### 5.1 Navigation

Top bar (always visible when authenticated):

- **Brand**: 🏆 + "Coderoad World Cup" wordmark
- **Tabs** (pill underline / active accent): Dashboard · Predictions · Ranking · Rules
- **Points chip**: `1,450 PTS` in `accent` pill — current user's total
- **Avatar**: links to `/profile`

### 5.2 Page surfaces

Each page packs the spec FRs onto one screen — see §6 for visuals.

| Page | FRs covered |
|---|---|
| Sign-in | FR-001, FR-002 |
| Dashboard | FR-004 (upcoming), FR-018 (top-3 global), FR-022 (recent result), FR-023 |
| Predictions | FR-004 (full match list), FR-006, FR-007, FR-008, FR-010, FR-011 |
| Ranking | FR-018, FR-019, FR-020, FR-020a, FR-021 |
| Profile | FR-022, FR-024, FR-025 |
| Rules | (new — explanatory; see §7) |

---

## 6. Screen catalogue

> **To embed the screenshots**: drop the seven images into
> `docs/screenshots/` with the filenames below. The markdown is already
> wired up.

### 6.1 Design system (`docs/screenshots/01-design-system.png`)

![Design system](./docs/screenshots/01-design-system.png)

The reference card we built the dark theme from. Shows:

- The four-color palette swatches with their hex codes
- The three Aa specimens (Lexend headline / Inter body / Space Grotesk label)
- Button states: Primary (purple fill), Secondary (dark fill), Inverted
  (light fill), Outlined
- A search input variant
- A bottom-nav variant (icons in pills) — **decoration only**, not used today
- Progress bars in primary/secondary/tertiary
- Action chips (icon-only and label) and tag dots

### 6.2 Sign-in (`docs/screenshots/02-sign-in.png`)

![Sign-in](./docs/screenshots/02-sign-in.png)

- Brand: 🏆 + "Coderoad World Cup"
- Headline: **Predict. Compete. Dominate.**
- Subtitle: "The ultimate enterprise prediction tournament. Log in to
  lock in your bracket, track live fixtures, and climb the global
  leaderboard."
- **Kickoff countdown**: 14 DAYS / 08 HOURS / 42 MINS (live tickdown)
- Google sign-in button with "Corporate credentials required" footnote
- Right column: three feature cards — `Analyze & Predict`,
  `Track Live`, `Climb the Ranks`
- Background: stadium photograph with purple gradient overlay

### 6.3 Dashboard (`docs/screenshots/03-dashboard.png`)

![Dashboard](./docs/screenshots/03-dashboard.png)

Four panels on a 2×2 grid:

- **My Standing** (left, top): rank `42 / 1,204`, total points `1,450`,
  percentile `Top 3%`
- **Upcoming** (right, top): featured upcoming match — `ARG vs FRA`,
  `Today, 20:00 UTC`, big purple **Make Prediction →** button
- **Global Top 3** (left, bottom): podium-less list with rank, avatar,
  display name, department, and points
- **Recent match result** (right, bottom): green check + **"Spot On!"**
  + `+120 Points Earned` chip

Footer: `© 2024 Coderoad Tech Stadium · Support · Official Rules · Privacy Policy`

### 6.4 Ranking (`docs/screenshots/04-ranking.png`)

![Ranking](./docs/screenshots/04-ranking.png)

- **GLOBAL** / **CURRENT PHASE** segmented toggle (purple fill on active)
- Multi-column leaderboard table:

| Column | Meaning |
|---|---|
| Rank | numeric or medal icon for top 3 |
| User | avatar + display name + (optional) tier badge ("ELITE PREDICTOR") |
| ME | match exact predictions (count) |
| GA | goals away predictions (?) — **needs spec** |
| GoA | goal accuracy (?) — **needs spec** |
| Pu | unique predictions (count) |
| Bonuses | accumulated bonus points (in green) |
| Total | grand total (Lexend big numeral) |

- Visual treatment: top 3 with gold/silver/bronze rank pills, "..."
  separator before the requesting user's row, and a **purple-tinted
  highlight row** for "You" (rank 42) followed by the row immediately
  after them

### 6.5 Predictions (`docs/screenshots/05-predictions.png`)

![Predictions](./docs/screenshots/05-predictions.png)

- Title: "Match Predictions"
- Subtitle: "Lock in your scores before kickoff. Earn points for exact
  matches and correct outcomes."
- Right rail: **ALL / PENDING / FINISHED** segmented filter
- Group tabs: `GROUP A` (active, accent underline) / `GROUP B` /
  `GROUP C` / `D` / `E` / `F` / `G` / `H` — **eight groups**
- Match cards in a grid. Two states shown:
  - **Editable**: `QAT 1 - 1 ECU`, `Nov 20 · 17:00`, `✓ SAVED` badge,
    "Update Prediction" button
  - **Locked**: `SEN 0 - 2 NED`, `STARTS IN 12M`, `🔒 LOCKED` badge,
    "Match Locked" disabled button (red/orange ring border)

### 6.6 Profile (`docs/screenshots/06-profile.png`)

![Profile](./docs/screenshots/06-profile.png)

- Header: large avatar with **`#4`** rank badge, name "Alex Mercer",
  subtitle `Engineering Dept. · Gold Tier`, total points (purple `1,450`),
  accuracy (`68%` in green)
- **Earned badges row**: pill chips — `🏆 Perfect Streak x5`,
  `🎯 Underdog Picker`, `⏱ Early Bird`
- Search input: "Search other participants..." (purple placeholder)
- Tabs: `All Matches` (active, purple pill) / `Group Stage` / `Knockouts`
- **Prediction history** stack — one row per finished match with:
  - Group label + date (Space Grotesk caps)
  - Big team codes + final score in Lexend (`BRA 2 - 0 SRB`)
  - `COMPLETED` chip
  - Your guess card (dark)
  - Actual & Points card with green outline if exact, neutral if not.
    Includes points awarded and reason ("Exact Score Bonus" /
    "Incorrect Result").

### 6.7 Rules (`docs/screenshots/07-rules.png`)

![Rules](./docs/screenshots/07-rules.png)

Three panels:

**Base Points** (icons + bilingual subtitles):

- 🚩 Exact Score (Marcador Exacto) — **5 PTS**
- ⊕ Correct Winner/Draw (Ganador o Empate) — **2 PTS**
- ⚽ Correct Goal Count (Gol Acertado) — **1 PT**
- 💜 Unique Prediction (Predicción Única) — **2 PTS**

**Phase Multipliers**: x1.5 Octavos · x2.0 Cuartos · x3.0 Semifinal ·
**x5.0 Final** (active in green)

**Grand Prizes** right rail:

1. **World Champion** — MacBook Pro M3 Max + Custom Trophy
2. **Elite Predictor** — iPad Pro 11" + Apple Pencil
3. **Sharp Eye** — $500 Premium Tech Voucher

---

## 7. Scoring model — proposed change

The Rules screen (image 16) shows a **different scoring model** than
the one currently implemented and locked in unit tests.

| Rule | Currently in code | Mockup proposes |
|---|---|---|
| Exact score | 10 pts | **5 pts** |
| Correct winner / draw | 5 pts | **2 pts** |
| Correct goal count (one team) | 3 pts | **1 pt** |
| Unique prediction | 5 pts | **2 pts** |
| Phase bonus | **flat +25** when winner correct on every phase match | **per-match multiplier** — x1.5 R16, x2.0 QF, x3.0 SF, **x5.0 Final** |

The two models are **structurally different**, not just numerically:

- The **flat all-or-nothing bonus** rewards consistency across a phase
- The **per-match multiplier** rewards getting any knockout match
  right, scaled by stage importance

Adopting the mockup's model requires:

1. Updating `RULE_VALUES` in `src/features/scoring/rules.ts`
2. Refactoring `engine.ts` — `scoreMatch` needs to take the match's
   round and apply the multiplier, replacing the
   `computePhaseBonusEarned` flat-bonus calculation
3. Rewriting `engine.test.ts` — all 20 cases need to be re-asserted;
   add cases per phase
4. Re-running `pnpm verify`
5. Amending the 2026-05-08 clarifications in
   [`specs/001-prediction-game/spec.md`](./specs/001-prediction-game/spec.md)

**Recommended next step:** confirm which model wins, then I can land
the change in one focused commit.

---

## 8. Tournament data integration

### 8.1 Provider adapter

```
src/features/providers/
  results-provider.ts    # interface + Zod schemas + factory
  mock-provider.ts       # reads supabase/seed.fixtures.json
  api-provider.ts        # real wc2026api.com client
```

Selection is via env: `RESULTS_PROVIDER=mock` (default) or `api`.

### 8.2 wc2026api.com integration (`api-provider.ts`)

| Concern | Detail |
|---|---|
| Base URL | `https://api.wc2026api.com` |
| Auth | `Authorization: Bearer <token>` |
| Endpoints used | `GET /matches` (listFixtures), `GET /matches/{id}` (getResult) |
| Output schema | Zod-validated against `Fixture` and `FinalResult` |
| Caching | `cache: "no-store"` (Next.js fetch) |

### 8.3 Round mapping

The API exposes 7 rounds. Our `match_phase` enum has 6 (no R32).

| API `round` | Our `MatchPhase` |
|---|---|
| `group` | `group_stage` |
| `R32` | **skipped** (16 fixtures lost) |
| `R16` | `round_of_16` |
| `QF` | `quarter_finals` |
| `SF` | `semi_finals` |
| `3rd` | `third_place` |
| `final` | `final` |

The mockups also imply a 32-team / 8-group format ("GROUP A through H"),
which doesn't match the actual 48-team / 12-group / 104-match format.
**See §10 for reconciliation.**

### 8.4 Operational pattern

```
1. Initial seed (pre-tournament): 72 group-stage fixtures load
2. After group stage finalises: re-run seed → R16 fixtures fill in (8 more)
3. After R16: re-run seed → QF (4 more)
4. After QF: re-run seed → SF (2 more)
5. After SF: re-run seed → 3rd-place + Final (2 more)
```

The seed UPSERTs on `external_ref`, so re-running is safe.

---

## 9. Secrets & operational config

All secrets live in `.env.local` at the repo root (gitignored). Never
commit them.

| Variable | Scope | Purpose |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | public | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | public | Supabase anon JWT |
| `SUPABASE_SERVICE_ROLE_KEY` | server-only | RLS-bypassing key for scripts + auth-callback cleanup |
| `ALLOWED_EMAIL_DOMAIN` | server-only | Domain whitelist for OAuth callback (FR-001) |
| `RESULTS_PROVIDER` | server-only | `mock` (default) or `api` |
| `WC2026_API_TOKEN` | server-only | Bearer token for wc2026api.com |

### 9.1 Current dev token (rotate before public launch)

> ⚠ **Treat this token like a database password.** It's currently
> committed in chat history; assume any reviewer with repo access can
> see it. Rotate from the wc2026api.com dashboard before the
> tournament goes live.

```
WC2026_API_TOKEN=wc26_EaCzw1LaHaWgUQLwY4a6ns
```

### 9.2 Token usage in scripts

```bash
pnpm tsx scripts/seed-matches.ts        # uses RESULTS_PROVIDER + WC2026_API_TOKEN
pnpm tsx scripts/record-result.ts ...   # CLI manual-fallback (Constitution VII)
pnpm tsx scripts/compute-badges.ts      # end-of-tournament badge computation
```

---

## 10. Spec reconciliation — open questions

The new mockups introduce **eight items** that conflict with the
2026-05-08-locked spec. Each needs a yes/no decision before the next
implementation pass.

| # | Item | Conflict | Recommended path |
|---|---|---|---|
| 1 | **Scoring values + phase multiplier** | FR-015 + 2026-05-08 clarification fix flat values + flat bonus | §7 — adopt mockup; update `RULE_VALUES`, engine, tests |
| 2 | **Group count** (mockups show 8 groups, A–H) | WC2026 has 12 groups (A–L) for 48 teams | Use 12 groups in product; update mockup labels (or accept it as design shorthand) |
| 3 | **Live match tracking** ("Track Live" feature card on sign-in) | FR-026 / Constitution VI excludes live tracking | Either remove the feature card, or amend FR-026 + add a live-feed provider integration (separate vendor) |
| 4 | **Tier system** ("Gold Tier", "Elite Predictor" badge inline on leaderboard rows) | Not in FR-024's 5-badge list | Either spec a tiering rule or treat tiers as visual-only labels for top contributors |
| 5 | **New badges on profile**: Perfect Streak, Underdog Picker, Early Bird | FR-024's closed list is 5 badges | Amend FR-024; document each badge's earning rule |
| 6 | **Leaderboard columns** ME, GA, GoA, Pu, Bonuses | Current schema has the per-match breakdown but no aggregate columns by name | Define each abbreviation; pre-aggregate or compute at read-time |
| 7 | **Physical prizes** (MacBook, iPad, $500 voucher) | FR-026 excludes payments; this is more like prizes than monetary stakes — but it's still real money | Confirm budget + procurement; document as informational on Rules page; don't add a "claim prize" feature |
| 8 | **Notifications implied** by "Track Live" / countdown / live updates | FR-026 explicitly excludes notifications | Decide if a kickoff-countdown is a notification (probably not — purely visual). Anything beyond that needs amendment |

---

## 11. Implementation status

| Component | Status |
|---|---|
| Auth flow (Google OAuth + domain check) | ✅ built |
| Supabase migration + RLS policies | ✅ built; needs `pnpm dlx supabase db push` against your project |
| Scoring engine + 20 unit tests | ✅ built (current 10/5/3/5 + flat bonus model) |
| Lock predicate + 6 unit tests | ✅ built |
| MatchResultsProvider (mock + api) | ✅ built; live-API smoke-tested (72 group fixtures) |
| Manual-fallback CLI | ✅ built |
| Badge computation CLI | ✅ built (current 5-badge model) |
| Dashboard panel layout (4 panels) | 🟡 current implementation has 3 panels; mockup proposes 4 (My Standing / Upcoming / Top 3 / Recent Result). Easy reorganization |
| Predictions page with group tabs | 🟡 not yet — current `/dashboard` lists upcoming inline. Needs a dedicated `/predictions` route with A–H (or A–L) group tabs and ALL/PENDING/FINISHED filter |
| Ranking page with multi-column table | 🟡 current Leaderboard has rank/name/total only. Needs ME/GA/GoA/Pu/Bonuses columns once those are spec'd (§10 #6) |
| Profile with tier + badges row + history | 🟡 current `/profile` has stat cards + badges + history but no tier; close to mockup |
| Rules page (new) | ❌ not built — purely informational; ship after §7 scoring decision |
| Sign-in countdown | ❌ not built — visual only; small Client Component, low cost |
| Live match panel (image 5 from old mocks) | ❌ excluded per Constitution VI |
| Reactions chat | ❌ excluded per Constitution VI |

---

## 12. Constitution & scope guards

The active constitution
([`.specify/memory/constitution.md`](./.specify/memory/constitution.md))
is unchanged. Its seven principles (I Simplicity, II Type Safety,
III Scoring Test Coverage, IV UX Consistency, V Real-time Performance,
VI Strict Scope, VII Operational Resilience) still apply.

Items that PRD2 proposes which would require constitutional or spec
amendments are listed in §10. Any new feature that is not listed
in §10 is **out of scope**, regardless of how compelling the mockup
makes it look. The same hard "no" list from `tasks.md` §"Scope guards"
applies:

- ❌ Admin panel
- ❌ Notifications (email/push/Slack/webhook)
- ❌ Payments / monetary stakes (note: physical prizes per §10 #7 are
  procurement, not payments-in-app)
- ❌ Multi-group / private leagues
- ❌ Configurable scoring rules (§7 is a one-time spec amendment, not
  a runtime config)
- ❌ Mobile-first / PWA
- ❌ Social sharing / external embedding
- ❌ "For future use" stubs

---

## 13. References

- Original brief: [`PRD.md`](./PRD.md)
- Constitution: [`.specify/memory/constitution.md`](./.specify/memory/constitution.md)
- Spec: [`specs/001-prediction-game/spec.md`](./specs/001-prediction-game/spec.md)
- Plan: [`specs/001-prediction-game/plan.md`](./specs/001-prediction-game/plan.md)
- Tasks: [`specs/001-prediction-game/tasks.md`](./specs/001-prediction-game/tasks.md)
- Quickstart / runbook: [`specs/001-prediction-game/quickstart.md`](./specs/001-prediction-game/quickstart.md)
- README (operational): [`README.md`](./README.md)
- WC2026 API docs: [https://api.wc2026api.com/docs](https://api.wc2026api.com/docs)

---

## 14. Decision log

| Date | Decision |
|---|---|
| 2026-05-08 | Initial spec ratified with five clarifications (scoring values, phase-bonus trigger, third-place phase, badge tie-break, leaderboard eligibility) |
| 2026-05-08 | Plan + simplification round: dropped denormalized totals, dropped `points_phase_bonus` column, dropped `match_state` enum, dropped HTTP fallback endpoint |
| later | Re-skin to GolaZo lime/cream theme (Option 3 of design review) |
| later | **This document** — re-skin to Coderoad World Cup dark/purple theme with the new screen IA (Dashboard / Predictions / Ranking / Rules) |
| pending | §7 scoring model decision |
| pending | §10 reconciliation items 2–8 |
