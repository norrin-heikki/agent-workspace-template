# Spec Reversing — Feature Inventory Prompt

Run this prompt with an AI agent from the root of any repository in the workspace. It produces a feature inventory + prioritized spec backlog following the ASDLC **Spec Reversing** pattern. This is **Phase 1** of two — it does NOT draft per-feature specs (see `prompts/spec-drafts.md` for that).

---

## The Prompt

```
You are auditing this repository to produce a feature inventory and a prioritized spec backlog, following the ASDLC **Spec Reversing** pattern. Do NOT draft per-feature specs in this run — only the inventory.

Determine the repository name from the current working directory.

## Phase 0: Sync with remote

Before any reading or writing:

1. Run `git status`. If the working tree is dirty, **stop and ask the user** — do not pull on top of uncommitted work, and do not stash without explicit permission.
2. Run `git fetch`, then `git pull --ff-only` if the current branch is behind its upstream. If the pull is not a fast-forward, **stop and ask the user**.
3. Record `git rev-parse HEAD` and include it at the top of the generated `INVENTORY.md` so reviewers know which revision the inventory reflects.

If the repo has no remote, skip steps 2 and note "local-only repo" in the inventory header.

## Reference material

If the **ASDLC skill** is available in your environment, consult it — specifically the **Spec Reversing** pattern and **The Spec** pattern. If not, the key directives are summarized inline below; that is enough to complete this task.

Read up front (whichever apply):

- `AGENTS.md` at the repo root and any `docs/agents/` files (existing conventions and reference material)
- `README.md`, the solution/project files (`*.sln`, `package.json`, `pyproject.toml`, `Cargo.toml`, etc.), and any deployment surface (`helm/`, `infra/`, `bicep/`, `terraform/`, `.github/workflows`, `azure-pipelines.yml`)
- The workspace-level `AGENTS.md` at `../../AGENTS.md` — specifically the **Repositories** and **Cross-repo Dependencies** sections — to discover whether sibling repos consume or produce this one's contracts

### Spec Reversing — key directives (inline summary)

- **Don't trust the code blindly.** Reversed specs will document bugs as features unless flagged. Call out anything that looks like a bug-being-documented-as-a-feature so a human can decide.
- **Keep it high-level.** Describe *behavior* ("retry limit is 5"), not code narration ("variable retryCount is assigned 5").
- **One file at a time.** This run produces ONLY the inventory.

## Scope

Determine the scope by reading the repository itself. A "feature" is a unit of behavior with its own contract — not a single class or component. Typical sources of features:

- **Backend** — controllers, route handlers, message consumers, background jobs, public services
- **Frontend** — top-level routes/pages, user-facing flows, embedded widgets
- **Library / SDK** — public modules, exported APIs
- **Infrastructure / IaC** — deployable units, modules, pipelines
- **CLI / tools** — top-level commands and subcommands

Read the code to confirm — do not infer from filenames alone.

## Cross-repo reconciliation

Check the workspace `AGENTS.md` (`../../AGENTS.md`) **Cross-repo Dependencies** section for relationships involving this repo. For each relationship in either direction:

- **This repo consumes** sibling X → read X's public surface (its `AGENTS.md`, its `docs/specs/INVENTORY.md` if present, or its actual published API/package contract) and note which features in this repo depend on it.
- **Sibling Y consumes** this repo → read Y's configuration or code that points at this repo (e.g. registry/config files, package references, API client code) and use it as a **source of truth for which features are externally visible**. Sibling-side keys, paths, or contract entries become candidate features to confirm here.

For each cross-repo touchpoint, cite both sides with `file_path:line` so a reviewer can verify.

If a declared sibling repo is **not checked out locally** under `repos/`, do not guess — note it as a limitation in the inventory's **Drift** section. The auditor (or a teammate) can rerun the cross-check later.

## What "audit" means here

1. **Enumerate features** by reading the code, routes, controllers, top-level frontend routes/pages, public APIs, deployable units, and any existing `docs/`. Don't infer from filenames alone — confirm by reading.

2. **For each feature, capture** (briefly, 1–3 lines each):
   - **Name** and one-sentence purpose (the behavior, not the implementation)
   - **Primary entry points** (files / routes / endpoints / commands) with `file_path:line`
   - **Internal touch points** (e.g. backend ↔ frontend, service ↔ database, module ↔ module)
   - **External exposure** — if the feature is reachable from a sibling repo per the cross-repo reconciliation above, list the sibling-side key(s) and contract entries with `file_path:line`. If not externally exposed, mark as **Internal-only**.
   - **Apparent dependencies** on other features (in this repo or siblings)
   - **Risk signals** — complex logic, missing tests, recent churn, TODOs, comments hinting at known issues

3. **Flag drift** — places where `docs/`, `AGENTS.md`, or comments contradict the code. Per the Spec Reversing directive, do not trust the code blindly — call out anything that looks like a bug-being-documented-as-a-feature so a human can decide during review.

4. **Reconcile against siblings** (where applicable):
   - Sibling-side keys/contracts whose target has no corresponding feature in this repo (dead sibling config or removed feature here).
   - Features here with no sibling reference where one is expected (orphan flows, or intentionally internal — confirm which).
   - Tile/widget/event/contract endpoints that other repos depend on — confirm they exist and document their response/payload shape at the field+type level.

## Output — write exactly one file

Path: `docs/specs/INVENTORY.md` (create `docs/specs/` if it does not exist).

Sections:

1. **Summary** — feature count, coverage gaps, top 3 risk areas (≤10 lines).
2. **Feature Inventory** — table or list per feature with the fields above.
3. **Spec Backlog (Prioritized)** — ordered list of which features should get a spec first. For each, give a one-line justification grounded in risk, churn, or imminent work. Highest priority first.
4. **Drift / Questions for Human Review** — things that look wrong or ambiguous and need a human decision before specs are written. Include any sibling repos that were not checked out and could not be reconciled.
5. **Proposed spec file naming convention** — e.g. `docs/specs/<area>/<feature>.md`, with a worked example using a real feature from this repo.

## Constraints

- Stay high-level. Describe behavior, not implementation details.
- One file at a time discipline: this run produces ONLY `INVENTORY.md`. No spec drafts, no edits to other files.
- Don't run tests or build the project — read-only audit.
- Cite `file_path:line` for every claim about code so a reviewer can verify.
- Match the existing `docs/` tone; no emoji.
- Disclosure: no secrets, environment **names** only, no private/internal URLs.

## Stop conditions

- Stop and ask before writing if `docs/specs/INVENTORY.md` already exists. Offer to incrementally update it instead of overwriting.
- If the repo is much bigger than expected (>30 features), produce a partial inventory covering the highest-risk surface and flag what was deferred — don't try to enumerate everything in one pass.
```
