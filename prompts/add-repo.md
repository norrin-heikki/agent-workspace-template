# Add Repository Prompt

Run this prompt with an AI agent from the workspace root to add a new repository.

---

## The Prompt

```
You are adding a new repository to this workspace.

All cloned repositories live under the `repos/` directory at the workspace root.

The user will provide a git clone URL (and optionally a directory name). If no directory name is given, derive one from the URL: lowercase, hyphens instead of spaces or special characters.

Work through the following steps in order.

---

### Step 1: Clone

Ensure the `repos/` directory exists at the workspace root (create it if not), then clone the repository into it:

```bash
git clone <url> repos/<directory>
```

If `repos/<directory>` already exists and contains a `.git` folder, skip cloning — the repo is already here.

---

### Step 2: Update AGENTS.md

Read the workspace `AGENTS.md`. Add a row to the Repositories table:

1. **Directory** — the folder name used for cloning (relative to `repos/`).
2. **Repository** — the clone URL.
3. **Description** — read the repo's `README.md` first paragraph, or inspect project files (`package.json`, `*.csproj`, `setup.py`, `Cargo.toml`, etc.) to produce a one-line summary.

Keep rows sorted alphabetically by directory name. Do not modify other sections.

---

### Step 3: Verify .gitignore

`.gitignore` already excludes the contents of `repos/` via the `repos/*` pattern, so no per-repo entry is required. Confirm the pattern is present; if it is missing, add it and ensure `repos/.gitkeep` exists.

---

### Step 4: Audit

Change into the new repo (`cd repos/<directory>`), then read `prompts/audit.md` (from the workspace root, i.e. `../../prompts/audit.md`) and follow its instructions completely. This generates `docs/agents/` and `AGENTS.md` inside the repo.

Return to the workspace root when done.
```
