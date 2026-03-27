# Workspace Template

A reusable template for managing multiple repositories under a single workspace. Provides AI agents with structured documentation and context to work productively across repos.

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

3. Clone the repositories into the workspace:

   ```bash
   git clone <repo-url> <directory-name>
   ```

4. Run the initialization prompt (`prompts/init.md`) with an AI agent (Claude Code, Cursor, etc.) from the workspace root. It will:

   - Discover all cloned repos
   - Populate `AGENTS.md` with a repository table
   - Add repos to `.gitignore`
   - Audit each repo and generate documentation

## What Gets Generated

### Per repository

| File | Purpose |
|---|---|
| `AGENTS.md` | Concise agent brief at each repo root — architecture, commands, conventions, links to detailed docs |
| `docs/agents/overview.md` | Project purpose, tech stack, key files, repo health, known gaps |
| `docs/agents/infrastructure.md` | CI/CD, Docker, cloud, env layout, observability |
| `docs/agents/frontend.md` | Components, design system, styling, a11y, i18n |
| `docs/agents/backend.md` | Services, API surface, auth, layering |
| `docs/agents/database.md` | Migrations, schema conventions |
| `docs/agents/testing.md` | Test frameworks, commands, fixtures, coverage |
| `docs/agents/authentication.md` | Auth flows, secrets policy |
| `docs/agents/contributing.md` | PR process, versioning, code style |

Only applicable files are created — areas that don't apply are skipped.

### At workspace root

| File | Purpose |
|---|---|
| `AGENTS.md` | Workspace-level agent instructions — lists all repos, links to prompts |
| `.gitignore` | Excludes cloned repos from the workspace git history |

## Prompts

| Prompt | When to use |
|---|---|
| [prompts/init.md](prompts/init.md) | First-time workspace setup, or when repos are added/removed |
| [prompts/add-repo.md](prompts/add-repo.md) | Add a new repo — clone, register in AGENTS.md and .gitignore, audit |
| [prompts/audit.md](prompts/audit.md) | Audit a single repo — run from that repo's root |

## Adding a New Repository

Run `prompts/add-repo.md` with the clone URL. It handles everything: cloning, registering in `AGENTS.md` and `.gitignore`, and running the full audit.

## Re-auditing

Run `prompts/audit.md` from any repo's root to update its documentation. The audit supports incremental updates — it reads existing files, preserves what's accurate, and tracks changes.

To re-audit all repos, run `prompts/init.md` from the workspace root again.

## Structure

```
my-workspace/
├── AGENTS.md              # Workspace agent instructions (auto-populated)
├── .gitignore             # Excludes cloned repos (auto-populated)
├── prompts/
│   ├── init.md            # Workspace initialization prompt
│   └── audit.md           # Single-repo audit prompt
├── repo-a/                # Cloned repo (gitignored)
│   ├── AGENTS.md          # Generated agent brief
│   └── docs/agents/       # Generated detailed docs
├── repo-b/                # Cloned repo (gitignored)
│   ├── AGENTS.md
│   └── docs/agents/
└── ...
```
