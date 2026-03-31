# Codebase Audit Prompt

Run this prompt with an AI agent from the root of any repository.

---

## The Prompt

```
Determine the repository name from the current working directory.

You are auditing this repository to produce two deliverables:

1. **`AGENTS.md`** at the repository root — the agent constitution. Minimal, human-authored-style, judgment-layer content only. This is the primary deliverable.
2. **Detailed documentation** under `docs/agents/` — human reference files for onboarding and deep dives. These are **not agent context** — they exist for humans and for agents to read on demand when they need detail on a specific area. **Skip files entirely** for areas that do not apply.

**Design principle:** Only put content in `AGENTS.md` that an agent cannot discover by reading the repo itself. Facts the agent can derive (tech stack from `package.json`, project structure from the filesystem, linter rules from config files) belong in `docs/agents/` as human reference — not in the agent constitution. Research shows that unnecessary instructions in context files actively harm agent performance by broadening exploration and increasing reasoning cost (Gloaguen et al., 2026).

If `docs/agents/` already contains files from a previous audit, this is an **incremental update**. Read every existing file first, preserve what is still accurate, update what has changed, and remove what no longer applies. Note meaningful changes at the top of each updated file in a `## Changelog` section (date + one-liner). Keep only the **last 5 entries** — remove older ones to prevent unbounded growth.

**Disclosure principles (apply to all output):** no secrets, environment **names** only, no private/internal URLs. For sensitive configuration, name the variable and where it is configured (e.g. `.env.example`, CI), not the value.

**Human reference files** (create only those that apply under `docs/agents/`):

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

**Judgment layer analysis** (critical — this feeds AGENTS.md)

Identify the things an agent **cannot** discover from toolchain config or file structure alone:
- Domain-specific constraints (e.g. "all writes must be ACID-compliant", "offline support is required")
- Architectural boundaries that are judgment calls, not enforced by tools (e.g. "no direct DB access from controllers")
- Dangerous areas where mistakes have outsized consequences (e.g. "billing logic requires extra review")
- Non-obvious conventions that differ from framework defaults
- Human-in-the-loop triggers specific to this project

---

### Phase 2: Write Human Reference Documentation

Write each applicable file under `docs/agents/`. Create the directory if it doesn't exist. These files are **detailed human reference** — they exist for onboarding developers and for agents to read on demand when working in a specific area. They are not loaded into agent context by default.

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

Create or update **`AGENTS.md` at the repository root**. This is the **agent constitution** — it follows the ASDLC anatomy and contains only what agents cannot discover from the repo itself.

**Design rule:** Before adding anything to `AGENTS.md`, ask: "Can the agent find this by reading a config file, `package.json`, directory listing, or running a command?" If yes, it does not belong here. If a linter, formatter, or type checker enforces a rule, do not restate it — the tool is the enforcement mechanism.

**Required structure:**

```markdown
# AGENTS.md

> **Project:** <one-line description of purpose and domain>
> **Core constraint:** <the single most important non-obvious constraint>

## Toolchain

| Action | Command | Notes |
|---|---|---|
| Build | `<command>` | <config file or output location> |
| Test | `<command>` | <flags or prerequisites> |
| Lint | `<command>` | <config file — do NOT describe what it enforces> |
| <other> | `<command>` | <notes> |

## Judgment Boundaries

**NEVER**
- <hard limits that require judgment, not tool enforcement>

**ASK**
- <human-in-the-loop triggers specific to this project>

**ALWAYS**
- <proactive judgment rules the agent should follow>

## Context Map

```yaml
<only list directories/files that would surprise someone who knows the framework>
<omit standard framework conventions the agent can infer>
```

## Human Reference

Detailed documentation lives in `docs/agents/` — read on demand, not preloaded.

| Area | File | Read when… |
|------|------|------------|
| Overview | `docs/agents/overview.md` | Orientation, architecture, cross-repo dependencies |
| <area> | `docs/agents/<file>.md` | <when to read> |
```

**What does NOT belong in AGENTS.md:**
- Tech stack lists (read `package.json`, `*.csproj`, `Cargo.toml`, etc.)
- Full architecture descriptions (read the code)
- Coding style rules enforced by linters/formatters (read tool configs)
- File paths the agent can discover by listing directories
- Content duplicated from README

**What DOES belong in AGENTS.md:**
- Domain constraints the code doesn't express (e.g. "offline-first", "ACID on all writes")
- Architectural boundaries that are judgment calls (e.g. "no business logic in controllers")
- Dangerous areas with outsized consequences (e.g. "billing module — changes require extra review")
- Non-obvious conventions that differ from framework defaults
- Cross-repo relationships not expressed in package dependencies

**Tone:** Concise, scannable, imperative. The entire file should fit in under 60 lines for most repos.

---

### Phase 4: Validate

Before finishing, verify everything you wrote:

1. **Path check** — every file path, directory, and link referenced in `docs/agents/` and `AGENTS.md` must exist in the repo. Remove or flag any that don't.
2. **Command check** — every shell command (build, test, lint, run) must be runnable. Verify by reading `package.json` scripts, `Makefile` targets, `*.csproj` files, CI configs, etc. Don't invent commands.
3. **Staleness check** — if updating existing docs, flag any sections that reference files, packages, or patterns that no longer exist in the repo.
4. **Secret scan** — re-read all output files and confirm no real secrets, tokens, connection strings, or internal URLs leaked in.
5. **AGENTS.md minimality check** — re-read `AGENTS.md` and remove anything the agent can discover from toolchain configs, `package.json`, directory structure, or README. If a linter enforces it, delete it from AGENTS.md.

If validation finds issues, fix them before finishing. List any unresolvable issues at the bottom of `docs/agents/overview.md` under **Known gaps & risks**.

---

### Guidelines

- **Read before writing.** Don't assume — open files and verify.
- **Be specific.** Use exact commands, exact paths, exact versions.
- **Minimal AGENTS.md.** The constitution carries only judgment-layer content. Everything else goes in `docs/agents/` as human reference.
- **Detailed human reference.** The `docs/agents/` files should be substantive (facts, paths, how things connect) — they serve humans and on-demand agent reads.
- **File size:** Aim for **100–300 lines** per `docs/agents/` file. Enough to be substantive, short enough to stay scannable. Split or trim if a file grows beyond this.
- **Don't pad.** Skip files/rows that truly do not apply; skip generic filler that isn't repo-specific.
- **Flag unknowns.** If you can't determine something, say so.
- **Preserve existing docs.** If files already exist under `docs/agents/` or `AGENTS.md` exists, read first and improve rather than replacing blindly.
- **Toolchain first.** If a constraint is enforced by a tool, the tool config is the authority — don't restate it in AGENTS.md.
```
