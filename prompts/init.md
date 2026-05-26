# Workspace Initialization Prompt

Run this prompt with an AI agent from the root of a freshly cloned workspace template. It discovers all sibling repositories, populates the workspace `AGENTS.md`, updates `.gitignore`, and audits each repo.

---

## The Prompt

```
You are initializing this workspace. The workspace is a git repository that acts as an umbrella for multiple cloned repositories belonging to one customer or project group.

All cloned repositories live under the `repos/` directory at the workspace root.

Work through the following phases in order.

---

### Phase 1: Discover Repositories

Find all git repositories already cloned inside the workspace's `repos/` directory.

1. If `repos/` does not exist at the workspace root, create it.
2. List all immediate subdirectories of `repos/`.
3. For each subdirectory, check if it contains a `.git` folder (i.e. it is a cloned repo).
4. For each discovered repo, determine:
   - **Directory name** — the folder name as-is (relative to `repos/`).
   - **Remote URL** — run `git -C repos/<dir> remote get-url origin` (if no remote, note "local only").
   - **Description** — read the repo's `README.md` first paragraph, or inspect project files (e.g. `package.json` description, `.csproj` description, `setup.py`, `Cargo.toml`) to produce a one-line summary of what the repo is.
   - **Primary tech** — the main language/framework (e.g. "C# ASP.NET Core", "React/TypeScript", "Terraform").

If no repos are found, tell the user to clone their repositories into `repos/` first, then re-run.

---

### Phase 2: Populate AGENTS.md

Read the existing `AGENTS.md` at the workspace root. Update it with the discovered repositories:

1. **Replace the example rows** in the Repositories table with actual discovered repos. Directory paths in the table are relative to `repos/`. Keep the table format:

   ```markdown
   | Directory | Repository | Description |
   |---|---|---|
   | `<dir>` | `<remote-url>` | <one-line description> |
   ```

2. Sort rows alphabetically by directory name.

3. Update the workspace name in the title if it is still a placeholder or generic. Use the workspace directory name.

4. Keep all other sections (Toolchain, Judgment Boundaries, Context Map, etc.) intact — they are part of the agent constitution and should stay.

---

### Phase 3: Verify .gitignore

Read the existing `.gitignore`. The default template already excludes the contents of `repos/` via:

```
repos/*
!repos/.gitkeep
```

This means individual cloned repos under `repos/` do not need to be listed separately. Verify this block is present. If it is missing (e.g. the workspace was created before the `repos/` convention), add it and ensure a `repos/.gitkeep` file exists so the directory is tracked.

Remove any obsolete per-repo entries that listed individual directories at the workspace root — they are no longer needed. Keep the OS and IDE sections intact.

---

### Phase 4: Audit Each Repository

For each discovered repository, run the full codebase audit defined in `prompts/audit.md`. Read that file and follow its instructions completely.

For each repo:
1. Change into the repo directory (`cd repos/<dir>`).
2. Execute all phases from `prompts/audit.md` (Discovery, Write Documentation, Generate AGENTS.md, Validate).
3. Return to the workspace root before moving to the next repo.

> **Context limits:** For workspaces with many repositories (4+), auditing all repos in a single session may hit context limits and degrade output quality for later repos. In that case, audit repos individually by running `prompts/audit.md` from each repo's root in separate sessions.

---

### Phase 5: Cross-repo Dependency Summary

After all audits are complete, review each repo's `repos/<dir>/docs/agents/overview.md` for the **Cross-repo dependencies** section. Compile a summary in the workspace `AGENTS.md` under the `## Cross-repo Dependencies` section:

1. List each dependency relationship as: `<consumer>` → `<provider>` with a short description of what is consumed (package, API, shared library, event contract, etc.).
2. If no cross-repo dependencies exist, note "No cross-repo dependencies detected" in that section.

This gives agents a workspace-level view of how repos relate to each other.

---

### Guidelines

- **Read before writing.** Don't assume — open files and verify.
- **Be specific.** Use exact commands, exact paths, exact versions.
- **Don't pad.** Skip files/rows that truly do not apply; skip generic filler.
- **Flag unknowns.** If you can't determine something, say so.
- **Preserve existing docs.** Read first, improve rather than replace blindly.
- **Disclosure:** No secrets, env *names* only, no private/internal URLs.
```
