# Spec Kit — Constitution & supporting files

Versioned source of truth for this project's **Spec-Driven Development** setup
([GitHub Spec Kit](https://github.com/github/spec-kit)).

The repository keeps `.specify/` git-ignored (agent config is local-only), so these
artifacts live here under `docs/spec-kit/` to be **shared, reviewed and versioned** by the
team. To actually run Spec Kit, copy them into a local `.specify/`:

```bash
mkdir -p .specify
cp -R docs/spec-kit/memory   .specify/
cp -R docs/spec-kit/adr      .specify/
cp    docs/spec-kit/tech-stack.md .specify/
```

## What's here

```
docs/spec-kit/
├── memory/
│   ├── constitution.md          # Non-negotiable project laws (18 principles)
│   ├── frontend-guidelines.md   # How the laws apply in the UI layer
│   ├── backend-guidelines.md    # How the laws apply in the service layer
│   ├── api-contracts.md         # Endpoint contracts (verifiable agreements)
│   ├── testing-guidelines.md    # Testability & determinism strategy
│   └── security-guidelines.md   # Security & privacy controls
├── tech-stack.md                # Approved technologies (React, Node, PostgreSQL…)
└── adr/
    ├── adr-001-react.md
    ├── adr-002-nodejs.md
    └── adr-003-postgresql.md
```

## How they relate

- **`constitution.md`** — the *laws*. Abstract, non-negotiable, never name a technology.
- **`*-guidelines.md`** — *how* we apply each law per layer (UI, services, testing, security).
- **`api-contracts.md`** — the laws made *verifiable* (request/response, errors, side effects).
- **`tech-stack.md` / `adr/`** — the concrete *technology*. The constitution only references
  "technologies approved by the Technology Stack" — it never names React directly, so the
  stack can change without touching the constitution.
