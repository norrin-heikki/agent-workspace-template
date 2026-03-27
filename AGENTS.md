# Workspace — Agent Instructions

This is a multi-repo workspace. AI agents use this file to understand the workspace and navigate across repositories.

## Initialization

To set up this workspace for the first time, run the prompt in [prompts/init.md](prompts/init.md). It will:

1. Discover all cloned repositories in this directory
2. Populate the table below
3. Update `.gitignore`
4. Run a full codebase audit on each repo

To clone a repository manually:

```bash
git clone <url> <directory>
```

Use the **Directory** column as the target folder name.

## Repositories

<!-- This table is populated by prompts/init.md. Add rows manually or re-run the init prompt. -->

| Directory | Repository | Description |
|---|---|---|

## Working Across Repos

Each cloned repository has its own `AGENTS.md` with repo-specific instructions. **Read the target repo's AGENTS.md before making changes in it.**

For detailed codebase documentation, see `docs/agents/` inside each repo (generated via [prompts/audit.md](prompts/audit.md)).

## Prompts

| Prompt | Purpose |
|---|---|
| [prompts/init.md](prompts/init.md) | First-time workspace setup — discover repos, populate this file, audit everything |
| [prompts/add-repo.md](prompts/add-repo.md) | Add a new repo — clone, register, audit |
| [prompts/audit.md](prompts/audit.md) | Audit a single repo — run from that repo's root |

## Adding a New Repository

Run [prompts/add-repo.md](prompts/add-repo.md) with the clone URL. It handles cloning, registration, `.gitignore`, and auditing in one step.
