# Codebase Audit Prompt

Run this prompt with an AI agent from the root of any repository.

---

## The Prompt

```
Determine the repository name from the current working directory.

You are auditing this repository to produce two deliverables:

1. **Detailed documentation** under `docs/agents/` — one file per applicable area (see output table below). **Skip files entirely** for areas that do not apply.
2. **`AGENTS.md`** at the repository root — a concise agent brief that links to the docs above.

If `docs/agents/` already contains files from a previous audit, this is an **incremental update**. Read every existing file first, preserve what is still accurate, update what has changed, and remove what no longer applies. Note meaningful changes at the top of each updated file in a `## Changelog` section (date + one-liner).

**Disclosure principles (apply to all output):** no secrets, environment **names** only, no private/internal URLs. For sensitive configuration, name the variable and where it is configured (e.g. `.env.example`, CI), not the value.

**Output files** (create only those that apply):

| File | Area | Typical topics |
|------|------|----------------|
| `docs/agents/overview.md` | Overview | Project purpose, architecture, tech stack, key files & entry points, known gaps & risks |
| `docs/agents/infrastructure.md` | Infrastructure | Docker, K8s, cloud, CI/CD, env layout, observability |
| `docs/agents/frontend.md` | UI / frontend | Components, design system, styling, accessibility, i18n |
| `docs/agents/backend.md` | Backend / API | Services, boundaries, auth, data layer |
| `docs/agents/database.md` | Database / data | Migrations, schema conventions |
| `docs/agents/testing.md` | Testing | Unit vs integration, e2e, how to run |
| `docs/agents/authentication.md` | Security / auth | OIDC, secrets handling (policy only, no secrets) |
| `docs/agents/contributing.md` | Contributing / releases | PR process, versioning, team conventions |

Work through the following phases. Be thorough — read actual files, don't guess.

---

### Phase 1: Discovery

Explore the repository and gather facts. For each item below, read the relevant files (don't just list directory names).

**Project identity**
- What is this project? (service, library, infrastructure, etc.)
- What problem does it solve / what domain does it serve?
- What are the primary languages, frameworks, and runtime versions?

**Architecture**
- What is the high-level architecture? (monolith, microservice, monorepo, IaC, library)
- What are the main modules / projects / packages and their responsibilities?
- How do they depend on each other?
- Are there any external service dependencies (databases, queues, APIs, auth providers)?

**Cross-repo dependencies**
- Does this repo consume packages, APIs, or shared libraries from sibling repositories in the same workspace?
- Does it publish anything (packages, contracts, events) consumed by other repos?
- Note these relationships — they will go into `docs/agents/overview.md`.

**Build & run**
- How is the project built? (build tool, scripts, CI pipeline)
- How is it run locally? (dev server, Docker, etc.)
- How is it deployed? (Helm, Bicep, Terraform, Azure Pipelines, GitHub Actions, etc.)
- What environment variables or config files are required?

**Testing**
- What test frameworks are used?
- Where are tests located?
- What are the commands to run tests (unit, integration, e2e)?
- Are there any test fixtures, snapshots, or seed data?

**Code conventions**
- What coding style is enforced? (linter, formatter, .editorconfig)
- What naming conventions are used? (files, classes, variables, CSS)
- What patterns are followed? (project structure, error handling, logging, DI)
- Are there pre-commit hooks?

**Documentation**
- What existing documentation exists? (README, wiki, docs/ folder, inline)
- Are there ADRs (Architecture Decision Records)?
- Is there API documentation (Swagger/OpenAPI, GraphQL schema)?

**Security & sensitive areas**
- Are there secrets management patterns? (Key Vault, env vars, config transforms)
- What authentication/authorization mechanisms are used?
- Are there areas that require extra caution when modifying?

**Repo health** (collect for `docs/agents/overview.md`)
- Are there tests? Roughly what coverage level? (read CI config or coverage reports if available)
- Are dependencies up to date or are there known outdated/vulnerable packages?
- Is CI green? What does the pipeline check?
- Are there TODOs, FIXMEs, or HACKs worth flagging?

**Deep dive per documentation area** (gather evidence for the output files; skip questions that do not apply)

- **Infrastructure** — Container images, compose/K8s manifests, cloud/IaC locations, pipeline definitions (e.g. `.github/workflows`, Azure Pipelines), environment variable layout and config templates, logging/metrics/tracing setup.
- **UI / frontend** — App entry points, component libraries/design system, CSS approach, a11y tooling, i18n/l10n libraries and message locations, Storybook or similar.
- **Backend / API** — Service/host boundaries, HTTP/gRPC layers, OpenAPI/Swagger paths, auth middleware, domain vs infrastructure layering.
- **Database / data** — ORM/migration tooling, migration folders, schema ownership, seed/data scripts.
- **Testing** — Test types and folders, runner commands, mocks/fixtures, e2e drivers, coverage expectations.
- **Security / auth** — OIDC/OAuth flows, API keys policy (where stored, never values), CSRF/CORS notes if relevant.
- **Contributing / releases** — `CONTRIBUTING.md`, branch/version policy, changelog/release docs, linters/formatters and how CI enforces them.

---

### Phase 2: Write Documentation

Write each applicable file under `docs/agents/`. Create the directory if it doesn't exist. Each file should be **self-contained** and **substantive** — not a single sentence. Cite real paths, tools, and workflows.

**`docs/agents/overview.md`** — always created:

```markdown
# <repo name>

## Overview
<!-- What this repo is, primary purpose, main technologies — 2–4 sentences -->

## Architecture
<!-- High-level shape: modules/services, dependencies, main data/control flow. Optional small diagram (mermaid OK). -->

## Cross-repo dependencies
<!-- What this repo consumes from or provides to sibling repos. Omit if standalone. -->

## Tech stack
<!-- Languages, frameworks, notable dependency versions -->

## Key files & entry points
<!-- Table of bulleted paths with one-line purpose -->

## Repo health
<!-- Test coverage (if known), CI status, dependency freshness, notable TODOs/FIXMEs -->

## Known gaps & risks
<!-- Tech debt, missing tests, unclear ownership, dep risks -->
```

**Per-area files** — use a heading matching the area name, then cover the topics listed in the output table above.

**Quality bar — example of a good `docs/agents/testing.md`:**

```markdown
# Testing

## Frameworks
- **Unit / integration:** xUnit 2.7 with FluentAssertions
- **E2E:** Playwright (configured in `e2e/playwright.config.ts`)

## Where tests live
- `src/Application.Tests/` — unit tests for domain logic
- `src/Api.Tests/` — integration tests (uses `WebApplicationFactory`, hits real DB via Testcontainers)
- `e2e/` — Playwright browser tests against a running instance

## How to run
| What | Command |
|------|---------|
| All unit tests | `dotnet test src/Application.Tests` |
| Integration tests | `dotnet test src/Api.Tests` (requires Docker for Testcontainers) |
| E2E | `cd e2e && npx playwright test` |
| Full CI suite | `dotnet test && cd e2e && npx playwright test` |

## Fixtures & data
- `src/Api.Tests/Fixtures/` — shared `WebApplicationFactory` setup
- `e2e/fixtures/` — seed data loaded before each Playwright suite

## Coverage
CI enforces 70% line coverage via Coverlet. Reports are generated to `coverage/` (gitignored).

## Conventions
- One test class per production class, mirroring the `src/` folder structure.
- Integration tests that need a database use the `[Collection("Database")]` attribute to avoid parallel conflicts.
- Prefer `Arrange / Act / Assert` comments only when the test is long enough to need them.
```

This is the level of detail expected. Adapt the content to whatever frameworks and patterns the repo actually uses.

---

### Phase 3: Generate AGENTS.md

Create or update **`AGENTS.md` at the repository root** using findings from Phases 1–2.

You are documenting this repository for AI coding agents (Cursor, Copilot, Claude Code, etc.).

**Goal:** Give agents enough context to work productively, with clear pointers to the detailed docs. Do not duplicate long guides — summarize briefly and link.

**Required sections:**

- **Overview** — what the project is and main technologies (short).
- **Architecture** — layers or services; optional small diagram or bullet list.
- **Local development** — prerequisites, how to run, ports; placeholder env vars only.
- **Common commands** — dev, test, lint, build, migrations/deploy helpers if applicable.
- **Conventions** — naming, formatting, patterns agents must follow; forbidden patterns if any.
- **Quality gates** — what must pass before a PR (match CI where possible).
- **Key paths** — entry points, main packages/apps, test locations.

**Documentation (required):** Add a subsection or table that **links to the detailed docs** under `docs/agents/`. Link only files that were created. **Omit a row** if that area does not apply. Include a **"Start here"** column that suggests which docs to read first for common tasks.

| Area | Link | Start here for… |
|------|------|-----------------|
| Overview | `docs/agents/overview.md` | Orientation, architecture, cross-repo dependencies |
| Infrastructure | `docs/agents/infrastructure.md` | CI/CD, deployment, environment config |
| UI / frontend | `docs/agents/frontend.md` | Component changes, styling, accessibility |
| Backend / API | `docs/agents/backend.md` | API changes, service boundaries, auth |
| Database / data | `docs/agents/database.md` | Migrations, schema changes |
| Testing | `docs/agents/testing.md` | Writing or running tests |
| Security / auth | `docs/agents/authentication.md` | Auth flows, secrets policy |
| Contributing / releases | `docs/agents/contributing.md` | PR process, versioning, code style |

Also add rows for any **other existing** repo docs (wiki, `docs/`, OpenAPI, `CONTRIBUTING.md`) as supplementary links.

Use **relative Markdown links** from the repository root. One short line per link describing when an agent should read it.

**Tone:** Concise, scannable, imperative. No long pasted content from other files — prefer links.

---

### Phase 4: Validate

Before finishing, verify everything you wrote:

1. **Path check** — every file path, directory, and link referenced in `docs/agents/` and `AGENTS.md` must exist in the repo. Remove or flag any that don't.
2. **Command check** — every shell command (build, test, lint, run) must be runnable. Verify by reading `package.json` scripts, `Makefile` targets, `*.csproj` files, CI configs, etc. Don't invent commands.
3. **Staleness check** — if updating existing docs, flag any sections that reference files, packages, or patterns that no longer exist in the repo.
4. **Secret scan** — re-read all output files and confirm no real secrets, tokens, connection strings, or internal URLs leaked in.

If validation finds issues, fix them before finishing. List any unresolvable issues at the bottom of `docs/agents/overview.md` under **Known gaps & risks**.

---

### Guidelines

- **Read before writing.** Don't assume — open files and verify.
- **Be specific.** Use exact commands, exact paths, exact versions.
- **Depth:** The `docs/agents/` files should be **detailed** (facts, paths, how things connect). `AGENTS.md` stays a **summary** with links.
- **Don't pad.** Skip files/rows that truly do not apply; skip generic filler that isn't repo-specific.
- **Flag unknowns.** If you can't determine something, say so.
- **Preserve existing docs.** If files already exist under `docs/agents/` or `AGENTS.md` exists, read first and improve rather than replacing blindly.
```
