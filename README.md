# Workspace Template

A reusable template for managing multiple repositories under a single workspace. Follows the [ASDLC](https://asdlc.io) approach to agent context — minimal constitutions, toolchain-first constraints, judgment-layer only.

## Quick Start

1. Clone this template into a new directory for your customer/project:

   ```bash
   git clone <this-repo-url> my-workspace
   cd my-workspace
   ```

2. Remove the template's git history and start fresh:

   ```bash
   rm -rf .git
   git init
   ```

   > **Alternative:** To preserve the ability to pull template updates later, keep the template remote and rename it instead: `git remote rename origin template`. Then add your own remote with `git remote add origin <your-repo-url>`.

3. Clone the repositories into the `repos/` directory:

   ```bash
   git clone <repo-url> repos/<directory-name>
   ```

   All cloned repositories live under `repos/` — the directory itself is tracked (via `repos/.gitkeep`), but its contents are gitignored.

4. Run the initialization prompt (`prompts/init.md`) with an AI agent (Claude Code, Cursor, etc.) from the workspace root. It will:

   - Discover all cloned repos under `repos/`
   - Populate `AGENTS.md` with a repository table
   - Verify `.gitignore` excludes `repos/*`
   - Audit each repo

## What Gets Generated

### Per repository

| File | Purpose |
|---|---|
| `AGENTS.md` | **Agent constitution** — minimal, judgment-layer content only (toolchain commands, behavioral boundaries, context map). Symlinked as `CLAUDE.md`. |
| `docs/agents/overview.md` | Architectural orientation — project purpose, tech stack, key files, repo health, known gaps |
| `docs/agents/infrastructure.md` | Architectural orientation — CI/CD, Docker, cloud, env layout, observability |
| `docs/agents/frontend.md` | Architectural orientation — components, design system, styling, a11y, i18n |
| `docs/agents/backend.md` | Architectural orientation — services, API surface, auth, layering |
| `docs/agents/database.md` | Architectural orientation — migrations, schema conventions |
| `docs/agents/testing.md` | Architectural orientation — test frameworks, commands, fixtures, coverage |
| `docs/agents/authentication.md` | Architectural orientation — auth flows, secrets policy |
| `docs/agents/contributing.md` | Architectural orientation — PR process, versioning, code style |
| `docs/specs/INVENTORY.md` | **Behavioral specs (Spec Reversing)** — feature inventory + prioritized backlog |
| `docs/specs/<area>/<feature>.md` | **Behavioral specs (Spec Reversing)** — Blueprint+Contract per feature |

Only applicable files are created — areas that don't apply are skipped.

`AGENTS.md` is loaded into agent context automatically. `docs/agents/` and `docs/specs/` files are **not** — they exist for humans and for agents to read on demand when they need detail on a specific area or are about to modify a specific feature. This follows the ASDLC principle that unnecessary context actively harms agent performance.

`docs/agents/` answers *"how is this codebase structured?"* (architectural orientation). `docs/specs/` answers *"what does this feature do, and what contract must I preserve?"* (behavioral specs derived via the ASDLC **Spec Reversing** pattern).

### At workspace root

| File | Purpose |
|---|---|
| `AGENTS.md` | Workspace-level agent constitution — mission, toolchain, judgment boundaries, repo table, context map |
| `CLAUDE.md` | Symlink to `AGENTS.md` (for Claude Code compatibility) |
| `.gitignore` | Excludes the contents of `repos/` from the workspace git history |
| `repos/` | Container directory for all cloned repositories; contents gitignored, folder preserved via `.gitkeep` |

## Design Principles

This template follows the [ASDLC AGENTS.md Specification](https://asdlc.io/practices/agents-md-spec) and the [Spec Reversing pattern](https://asdlc.io/patterns/spec-reversing):

- **Minimal by design** — `AGENTS.md` contains only what agents cannot discover from the repo itself
- **Toolchain first** — if a linter, formatter, or type checker enforces a rule, it is not restated in `AGENTS.md`
- **Judgment boundaries** — uses the NEVER/ASK/ALWAYS tier system for behavioral rules
- **Context map** — only lists directories/files that would surprise someone who knows the framework
- **Three-layer separation** — agent constitution (`AGENTS.md`), architectural orientation (`docs/agents/`), and behavioral specs (`docs/specs/`) are kept distinct. Only the constitution is loaded into context automatically.
- **Spec Reversing for brownfield** — behavioral specs are derived from existing code via a two-phase workflow (inventory → drafts), with a human-review gate between phases to prevent canonizing bugs as features.

## Prompts

| Prompt | When to use |
|---|---|
| [prompts/init.md](prompts/init.md) | First-time workspace setup, or when repos are added/removed |
| [prompts/add-repo.md](prompts/add-repo.md) | Add a new repo — clone, register in AGENTS.md and .gitignore, audit |
| [prompts/audit.md](prompts/audit.md) | Audit a single repo — generates `AGENTS.md`, `docs/agents/`, and orchestrates spec reversing |
| [prompts/spec-inventory.md](prompts/spec-inventory.md) | Spec Reversing Phase 1 — produce `docs/specs/INVENTORY.md` for a single repo |
| [prompts/spec-drafts.md](prompts/spec-drafts.md) | Spec Reversing Phase 2 — produce per-feature Blueprint+Contract specs from the inventory |
| [prompts/migrate-v2.md](prompts/migrate-v2.md) | One-time migration from original template to ASDLC-aligned v2 |

## Adding a New Repository

Run `prompts/add-repo.md` with the clone URL. It handles everything: cloning, registering in `AGENTS.md` and `.gitignore`, and running the full audit.

## Removing a Repository

Delete the repo directory, then re-run `prompts/init.md` from the workspace root. It will re-discover repos and clean up stale entries from `AGENTS.md` and `.gitignore`.

## Migrating from v1

If you have an existing workspace initialized with the original (pre-ASDLC) template:

1. Copy the updated template files into your workspace, replacing the old versions:

   ```
   prompts/audit.md
   prompts/init.md
   README.md
   ```

2. Copy the new migration prompt into your workspace:

   ```
   prompts/migrate-v2.md
   ```

3. Run `prompts/migrate-v2.md` with an AI agent from the workspace root. It will:

   - Extract your existing repository table and custom content
   - Rewrite the workspace `AGENTS.md` to the ASDLC anatomy
   - Move cloned repositories from the workspace root into `repos/`
   - Replace per-repo `.gitignore` entries with the `repos/*` block
   - Replace `CLAUDE.md` with a symlink to `AGENTS.md`
   - Re-audit each repo to generate minimal, judgment-layer `AGENTS.md` files
   - Update cross-repo dependency summary

For large workspaces (4+ repos), the migration prompt will complete the workspace-level changes in one session, then you can re-audit repos individually by running `prompts/audit.md` from each repo's root in separate sessions.

## Re-auditing

Run `prompts/audit.md` from any repo's root to update its documentation. The audit supports incremental updates — it reads existing files, preserves what's accurate, and tracks changes.

To re-audit all repos, run `prompts/init.md` from the workspace root again.

## Structure

```
my-workspace/
├── AGENTS.md              # Workspace agent constitution (auto-populated)
├── CLAUDE.md              # Symlink → AGENTS.md
├── .gitignore             # Excludes repos/* (contents gitignored)
├── prompts/
│   ├── init.md            # Workspace initialization prompt
│   ├── add-repo.md        # Add a new repo prompt
│   ├── audit.md           # Single-repo audit prompt (orchestrates spec reversing)
│   ├── spec-inventory.md  # Spec Reversing Phase 1 — feature inventory
│   ├── spec-drafts.md     # Spec Reversing Phase 2 — per-feature specs
│   └── migrate-v2.md      # One-time migration prompt
└── repos/                 # Container for cloned repos (contents gitignored)
    ├── .gitkeep           # Keeps the directory in git
    ├── repo-a/            # Cloned repo
    │   ├── AGENTS.md      # Agent constitution (minimal, judgment-layer)
    │   ├── docs/agents/   # Architectural orientation (on-demand)
    │   └── docs/specs/    # Behavioral specs (Spec Reversing — on-demand)
    ├── repo-b/            # Cloned repo
    │   ├── AGENTS.md
    │   ├── docs/agents/
    │   └── docs/specs/
    └── ...
```
