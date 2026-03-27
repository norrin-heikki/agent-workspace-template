# Add Repository Prompt

Run this prompt with an AI agent from the workspace root to add a new repository.

---

## The Prompt

```
You are adding a new repository to this workspace.

The user will provide a git clone URL (and optionally a directory name). If no directory name is given, derive one from the URL: lowercase, hyphens instead of spaces or special characters.

Work through the following steps in order.

---

### Step 1: Clone

Clone the repository into the workspace directory:

```bash
git clone <url> <directory>
```

If the directory already exists and contains a `.git` folder, skip cloning — the repo is already here.

---

### Step 2: Update AGENTS.md

Read the workspace `AGENTS.md`. Add a row to the Repositories table:

1. **Directory** — the folder name used for cloning.
2. **Repository** — the clone URL.
3. **Description** — read the repo's `README.md` first paragraph, or inspect project files (`package.json`, `*.csproj`, `setup.py`, `Cargo.toml`, etc.) to produce a one-line summary.

Keep rows sorted alphabetically by directory name. Do not modify other sections.

---

### Step 3: Update .gitignore

Read `.gitignore`. Add the new directory (with trailing `/`) under the `# Cloned repositories` comment. Keep entries sorted alphabetically.

---

### Step 4: Audit

Read `prompts/audit.md` and follow its instructions completely, running from the new repo's root. This generates `docs/agents/` and `AGENTS.md` inside the repo.

Return to the workspace root when done.
```
