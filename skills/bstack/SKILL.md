---
name: bstack
description: Manage bstack — first-time setup across harnesses, pulling and syncing symlinks, and committing/pushing local skill updates to the remote repository. Use when the user asks to set up bstack, sync or update their skills stack, or push local skill changes.
---

# Manage Skills Stack (`bstack`)

Single skill to initialize, sync, and publish updates for the `bstack` repository across AI coding harnesses.

## Supported Harnesses

| Harness | Global Skills Directory | Detection Criteria |
| --- | --- | --- |
| **Codex** | `~/.agents/skills` | `codex` in `PATH`, `~/.agents`, or `~/.codex` |
| **OpenCode** | `~/.agents/skills` | `opencode` in `PATH` or `~/.config/opencode` |
| **Claude Code** | `~/.claude/skills` | `claude` in `PATH` or `~/.claude` |
| **Antigravity** | `~/.gemini/config/skills` | `agy` in `PATH` or `~/.gemini` |

*(Note: Codex and OpenCode share `~/.agents/skills`; deduplicate so it is only synced once when both are selected.)*

## Scope & Local Skill Isolation

- **`bstack` only manages skills inside `<repo>/skills/`.** Only files inside `<repo>` (including edits made through symlinks pointing into `<repo>/skills/`) are synced, committed, or pushed.
- **Never touch or import non-`bstack` local skills.** Pre-existing or locally created skills sitting directly in `~/.agents/skills/`, `~/.claude/skills/`, or `~/.gemini/config/skills/` (anything that is not a symlink into `<repo>/skills/`) are strictly out of scope. Never copy, move, import, overwrite, delete, or push them into `<repo>` unless the user explicitly asks to migrate a specific local skill by name.
- **New `bstack` skills always live in `<repo>/skills/`.** Creating a new skill is handled by `skills/add-skill/SKILL.md` (`add-skill`); once created in `<repo>/skills/<skill-name>/SKILL.md`, `bstack` handles symlinking it to linked harnesses via **Safe Symlink Sync** and committing/pushing via **Workflow B** when requested.

## Step 1: Locate the Repository & Inspect State

1. **Locate `<repo>`:**
   - If the current workspace contains `skills/bstack/SKILL.md`, use it as `<repo>`.
   - Otherwise, resolve the symlink of this skill's own directory (checking `~/.agents/skills/bstack`, `~/.claude/skills/bstack`, or `~/.gemini/config/skills/bstack` via `cd -P` / `pwd -P`) and step up two levels.
   - Always canonicalize `<repo>` (`pwd -P`) so symlinks use canonical absolute paths (`<repo>/skills/<skill-name>`).

2. **Check current state:**
   - **Linked harnesses:** Check which of the supported global skills directories already contain at least one symlink pointing inside `<repo>/skills/`.
     - If **none** exist, this is a **first-time setup** on this machine.
   - **Git state:** Check `git -C "<repo>" status --short`, configured remotes (`git -C "<repo>" remote`), and any unpushed commits.

3. **Choose the right workflow:**
   - **First-time setup** (no harnesses linked yet, or the user explicitly asks to set up / initialize / change harnesses) → run **Workflow A (Setup & Harness Selection)**.
   - **Create a new `bstack` skill** (the user asks to create/add a new skill to `bstack`) → follow `<repo>/skills/add-skill/SKILL.md` first, then run **Safe Symlink Sync** and/or **Workflow B** only if the user confirms they want to sync/push.
   - **Publish local changes** (the user says they added, edited, or deleted a skill in `bstack` and wants to update/sync/push it) → run **Workflow B (Commit, Push & Refresh Symlinks)**.
   - **Pull & sync** (already set up, clean working tree, and user asks to update or sync their stack) → run **Workflow C (Pull & Sync)**.
   - If already set up with uncommitted local changes and the user's intent is genuinely unclear (e.g., *"sync my stack"* without mentioning local edits), ask a brief clarifying question: whether to commit & push their local edits or only refresh local symlinks.

---

## Workflow A: First-Time Setup (Interactive)

1. If `<repo>` has a Git remote and a clean working tree, run `git -C "<repo>" pull --ff-only` first (warn and continue if offline or pull fails).
2. Detect all four harnesses using the table above, labeling each as `(detected)` or `(not installed)`.
3. Present the detected harnesses to the user and **ask which harnesses they want to set up** (suggesting the detected ones by default). Wait for their reply before making changes.
4. Once confirmed, run the **Safe Symlink Sync** procedure below for the chosen harnesses and present a concise summary.

---

## Workflow B: Commit, Push & Refresh Symlinks

1. Verify `<repo>` is a Git repository with a configured remote. If no remote is configured, tell the user and stop.
2. Inspect `git -C "<repo>" status --short` and `git -C "<repo>" diff` silently to understand what changed inside `<repo>` only (never scan harness directories for non-`bstack` local skills to import, and do not dump raw diffs to the user unless there is an issue to resolve).
   - If there are no uncommitted changes and no unpushed commits inside `<repo>`, tell the user `<repo>` has no pending changes to push (noting that `bstack` only tracks skills inside `<repo>/skills/` and leaves non-`bstack` local skills untouched). Run **Workflow C** if they also wanted to pull/sync.
3. **Quick safety check before committing:**
   - Verify any added or modified `skills/<name>/SKILL.md` has valid YAML frontmatter with `name` (matching its folder name) and `description`.
   - Ensure no secrets, credentials, or unintended personal/local machine paths are included (author attribution in `LICENSE` is fine). If an issue is found, flag it and stop before committing.
4. **Commit & push:**
   - Draft a clear Conventional Commit message summarizing the actual changes (e.g., `feat(skills): add <name>` or `chore(skills): update <name>`).
   - Stage changes (`git -C "<repo>" add -A`) and commit (`git -C "<repo>" commit -m "<message>"`).
   - Push using `GIT_TERMINAL_PROMPT=0 git -C "<repo>" push` (or `-u <remote> <branch>` if no upstream is set).
5. **Handle push errors cleanly:**
   - **Permission / authorization error** (`Permission denied`, `403`, `Write access to repository not granted`, `Authentication failed`, `Could not read from remote repository`):
     - Revert the commit just created (`git -C "<repo>" reset --mixed HEAD~1`, or `git -C "<repo>" update-ref -d HEAD && git -C "<repo>" reset` for an initial commit) so local file edits remain intact.
     - Clearly inform the user they do not have push permission to this remote repository (only the repo owner/maintainers can push) and suggest forking the repository and updating their Git remote to save their own copy.
   - **Remote ahead** (`non-fast-forward` / `fetch first`):
     - Inform the user and offer to pull with rebase (`git pull --rebase`) before retrying the push.
6. **Refresh local symlinks if needed:**
   - If any skill folders were added or removed and harnesses are already linked on this machine, automatically run **Safe Symlink Sync** on the currently linked harnesses so symlinks stay up to date.
7. Report a concise summary: commit hash, commit message, pushed remote/branch, and any symlink updates.

---

## Workflow C: Pull & Sync Existing Setup

1. If `<repo>` has a Git remote:
   - If uncommitted changes exist, warn and skip `git pull`.
   - Otherwise run `git -C "<repo>" pull --ff-only` (warn and continue if offline or pull fails).
2. Run **Safe Symlink Sync** on all harnesses currently linked to `<repo>/skills/` (and if any newly installed harness is detected that isn't linked yet, mention it in case the user wants to add it).
3. Report a concise summary.

---

## Safe Symlink Sync Procedure

1. **Validate selected harnesses & deduplicate targets:**
   - If a selected harness is **not installed**, do **not** create its config folder. Print a clear error/warning telling the user to install that harness first and re-run later, then continue syncing the remaining valid harnesses.
   - Deduplicate shared target directories (e.g., `Codex, OpenCode` → `~/.agents/skills`) so each physical directory is synced once.
2. **Dead symlink cleanup:**
   - Ensure the valid harness's `skills/` directory exists (`mkdir -p`).
   - Remove any broken symlink (`[ -L "$entry" ] && [ ! -e "$entry" ]`) whose resolved target points inside `<repo>/skills/`, logging `[removed] <name> (broken symlink)`.
   - **Never** touch regular files/folders or symlinks pointing elsewhere.
3. **Per-skill symlinking (never overwrite local skills):**
   - For each directory `<repo>/skills/<skill-name>`:
     - Already a symlink pointing to `<repo>/skills/<skill-name>` → `[ok] <skill-name>`.
     - Existing local file, folder, or symlink pointing elsewhere → **skip** and log `[skipped] <skill-name> (local skill exists — not overwritten)`.
     - Missing → create `ln -s "<repo>/skills/<skill-name>" "<target>/<skill-name>"` and log `[added] <skill-name>`.
4. **Summary:** Report counts for **added**, **ok**, **skipped**, **removed**, and **errored**.
