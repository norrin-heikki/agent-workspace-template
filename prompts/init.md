# Workspace Initialization Prompt

Run this prompt with an AI agent from the root of a freshly cloned workspace template. It discovers all sibling repositories, populates the workspace `AGENTS.md`, updates `.gitignore`, and audits each repo.

---

## The Prompt

```
You are initializing this workspace. The workspace is a git repository that acts as an umbrella for multiple cloned repositories belonging to one customer or project group.

Work through the following phases in order.

---

### Phase 1: Discover Repositories

Find all git repositories already cloned inside this workspace directory.

1. List all immediate subdirectories of the current working directory.
2. For each subdirectory, check if it contains a `.git` folder (i.e. it is a cloned repo).
3. For each discovered repo, determine:
   - **Directory name** — the folder name as-is.
   - **Remote URL** — run `git -C <dir> remote get-url origin` (if no remote, note "local only").
   - **Description** — read the repo's `README.md` first paragraph, or inspect project files (e.g. `package.json` description, `.csproj` description, `setup.py`, `Cargo.toml`) to produce a one-line summary of what the repo is.
   - **Primary tech** — the main language/framework (e.g. "C# ASP.NET Core", "React/TypeScript", "Terraform").

If no repos are found, tell the user to clone their repositories into this directory first, then re-run.

---

### Phase 2: Populate AGENTS.md

Read the existing `AGENTS.md` at the workspace root. Update it with the discovered repositories:

1. **Replace the example rows** in the Repositories table with actual discovered repos. Keep the table format:

   ```markdown
   | Directory | Repository | Description |
   |---|---|---|
   | `<dir>` | `<remote-url>` | <one-line description> |
   ```

2. Sort rows alphabetically by directory name.

3. Update the workspace name in the title if it is still a placeholder or generic. Use the workspace directory name.

4. Keep all other sections (`Working Across Repos`, `Adding a New Repository`, etc.) intact — they are generic and should stay.

---

### Phase 3: Update .gitignore

Read the existing `.gitignore`. Add entries for each discovered repo directory so that the workspace repo does not track cloned repos.

1. Under the `# Cloned repositories` comment, list each discovered repo directory with a trailing `/`.
2. Remove any entries that refer to directories that no longer exist.
3. Keep the OS and IDE sections intact.

---

### Phase 4: Audit Each Repository

For each discovered repository, run the full codebase audit defined in `prompts/audit.md`. Read that file and follow its instructions completely.

For each repo:
1. Change into the repo directory.
2. Execute all phases from `prompts/audit.md` (Discovery, Write Documentation, Generate AGENTS.md, Validate).
3. Return to the workspace root before moving to the next repo.

---

### Guidelines

- **Read before writing.** Don't assume — open files and verify.
- **Be specific.** Use exact commands, exact paths, exact versions.
- **Don't pad.** Skip files/rows that truly do not apply; skip generic filler.
- **Flag unknowns.** If you can't determine something, say so.
- **Preserve existing docs.** Read first, improve rather than replace blindly.
- **Disclosure:** No secrets, env *names* only, no private/internal URLs.
```
