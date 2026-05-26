# Spec Reversing — Draft Specs from Inventory

Run this prompt with an AI agent from the root of any repository in the workspace **after** `prompts/spec-inventory.md` has produced `docs/specs/INVENTORY.md`. It drafts full specs for every feature in the inventory, following the ASDLC **The Spec** pattern (Blueprint + Contract).

---

## The Prompt

```
You are drafting full specs for every feature listed in `docs/specs/INVENTORY.md`, following the ASDLC **The Spec** pattern (anatomy: Blueprint + Contract) and the **Spec Reversing** pattern (directives for reversing specs from existing code). One spec file per feature.

Determine the repository name from the current working directory.

## Phase 0: Sync with remote

Before any reading or writing:

1. Run `git status`. If the working tree is dirty, **stop and ask the user** — do not pull on top of uncommitted work, and do not stash without explicit permission.
2. Run `git fetch`, then `git pull --ff-only` if the current branch is behind its upstream. If the pull is not a fast-forward, **stop and ask the user**.
3. Confirm `INVENTORY.md`'s recorded HEAD SHA (if present) matches `git rev-parse HEAD`. If the inventory was produced against an older revision, **stop and ask the user** whether to re-run `prompts/spec-inventory.md` first or proceed with the existing inventory.

If the repo has no remote, skip step 2 and proceed.

## Reference material

If the **ASDLC skill** is available in your environment, consult it — specifically **The Spec** pattern and the **Spec Reversing** pattern. If not, the key directives are summarized inline below; that is enough to complete this task.

Read up front:

- `docs/specs/INVENTORY.md` — the source of truth for what to spec. **Stop and ask the user** if this file does not exist; run `prompts/spec-inventory.md` first.
- `AGENTS.md` at the repo root and any relevant `docs/agents/` subpages.
- The **Drift / Questions for Human Review** section of `INVENTORY.md` — if a feature has unresolved questions there, **STOP and ask the user** before writing that feature's spec. Do not paper over ambiguity.
- The workspace-level `AGENTS.md` at `../../AGENTS.md` — specifically the **Cross-repo Dependencies** section. For any sibling repo this repo interacts with, attempt to read it locally under `repos/<sibling>/` (typically `../../repos/<sibling>/` from the current repo, or `../<sibling>/`). The sibling repo is the source of truth for any external contract this repo must honor. If the sibling is not checked out locally, note that in each affected spec's **Open Questions** and proceed with the inventory's notes as best evidence.

### Spec Reversing — key directives (inline summary)

- **Don't trust the code blindly.** Flag bugs as Open Questions instead of canonizing them in the spec.
- **Keep it high-level.** Describe *behavior*, not code narration.
- **One file at a time.** Even within this batch run, draft and review each file before moving on (see Workflow below).

## Output

For each feature in `INVENTORY.md`, write exactly one file at the path proposed by the inventory's naming convention (typically `docs/specs/<area>/<feature>.md`).

Use this structure:

```markdown
---
name: <feature-slug>
status: Reversed (Draft)
source: spec-reversing
last-reviewed: <today's date, ISO>
related-pbis: []
---

# <Feature Name>

## Blueprint

### Context
Why this feature exists. The user-facing or system-level problem it solves. 3–6 sentences. No implementation detail.

### Architecture
The contracts and boundaries an agent must respect:
- API endpoints / routes (HTTP method, path, request/response shape at the level of fields and types — not code)
- Data model: entities and their key fields, ownership boundaries
- Backend ↔ frontend boundary (or module ↔ module): what crosses, in what shape
- External dependencies (other services, libraries with load-bearing behavior)
- Dependency direction: who calls whom; what must not call what
Every claim cites `file_path:line` so a reviewer can verify.

### Cross-repo Integration Surface
How sibling repos in this workspace consume this feature, or how this feature consumes sibling contracts. Omit this section only if the feature is **Internal-only** per the inventory; otherwise include:
- Sibling-side contract entry/key (e.g. registry config, package reference, API client) and the localized or environment-specific surfaces it exposes.
- The target route/endpoint/contract on this side that handles it, with `file_path:line` on both sides.
- Whether the feature contributes shared payloads (tiles, events, webhooks, etc.) that siblings depend on; if so, the payload shape this feature returns.
- Auth handoff notes if the feature has special SSO/logout/token behavior beyond the repo defaults.
This section makes the **external contract** explicit — it is the part that breaks siblings if changed without coordination.

### Anti-Patterns
Things an agent must NOT do when modifying this feature. Derive these from code that looks intentionally defensive, from comments warning against patterns, or from drift you noticed. Examples:
- "Do not bypass <validator> — it enforces <invariant>."
- "Do not call <X> directly from the frontend; route through <Y>."
Cite the `file_path:line` that motivates each rule.

## Contract

### Definition of Done
Observable success criteria for any change to this feature. Phrased as checkable statements, not implementation steps.

### Regression Guardrails
Invariants that must never break, even during refactors. These are the load-bearing rules of the feature. Cite existing tests that enforce them (`file_path:line`); flag invariants that have NO test coverage as **Gap:** entries — these are candidates for new tests.

### Scenarios
Gherkin-style behavioral specs. Aim for 3–8 scenarios per feature covering: happy path, primary alternative paths, and known edge cases.

  Scenario: <name>
    Given <state>
    When <action>
    Then <observable outcome>

Do not invent scenarios — every Given/When/Then must be traceable to real code paths. If a scenario describes desired-but-unverified behavior, mark it `[unverified]` and add to **Open Questions** below.

## Open Questions
Anything the code didn't make obvious. One bullet per question, with the `file_path:line` that triggered it. Empty section is fine.
```

## Workflow (strict)

For each feature in `INVENTORY.md`'s prioritized backlog:

1. Read every `file_path` cited for that feature in the inventory.
2. Grep/read adjacent code to confirm behavior — do not extrapolate from filenames or comments alone.
3. If the feature has cross-repo exposure per the inventory, read the matching sibling-side configuration/contract entries to confirm the external contract.
4. Check this repo's test folders for assertions that pin invariants. Cite them in **Regression Guardrails**. Flag uncovered invariants as **Gap:** entries.
5. Draft the spec file.
6. Re-read your draft against the Spec Reversing directives:
   - High-level behavior, not code narration?
   - Did you describe what code *does* without judging if it's right? If you saw something that looks like a bug, did you flag it in **Open Questions** rather than canonizing it?
7. Write the file. Move to the next feature.

After all features are written, append a summary to `INVENTORY.md` under a new section `## Specs Drafted (<date>)` listing the files created and any features skipped (with reason).

## Constraints

- One file per feature. Do not merge features into shared specs even if they look similar — let the human do that later if appropriate.
- Stay `spec-anchored`, not `spec-as-source`. Specs describe intent and constraints; they do not duplicate the code.
- No emoji. Match the tone of existing `docs/`.
- Do not modify code, tests, or any file other than the new spec files and the `INVENTORY.md` summary append.
- Do not run tests or builds — read-only research, then write.
- Disclosure: no secrets, environment **names** only, no private/internal URLs.

## Stop conditions

- `docs/specs/INVENTORY.md` does not exist → stop and direct the user to `prompts/spec-inventory.md`.
- A feature has unresolved entries in `INVENTORY.md`'s **Drift / Questions** section → ask the user before writing that feature's spec.
- A target spec file already exists → ask before overwriting; offer to update incrementally instead.
- You're more than 50% guessing about a feature's intent → stop, write what you have under **Open Questions**, and ask.
```
