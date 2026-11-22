---
name: add-skill
description: Create a new Agent Skill inside the bstack repository (skills/<name>/SKILL.md) and update README.md without committing or pushing by default. Use when the user asks to add or create a new skill in bstack.
---

# Add Skill (`add-skill`)

Create a new, well-structured, multi-harness Agent Skill inside the `bstack` repository.

## 1. Locate the `bstack` Repository

- If the current workspace contains `skills/add-skill/SKILL.md`, use it as `<repo>`.
- Otherwise, resolve the symlink of `~/.agents/skills/add-skill`, `~/.claude/skills/add-skill`, or `~/.gemini/config/skills/add-skill` (`cd -P` / `pwd -P`) and step up two levels.
- Always create new `bstack` skills inside `<repo>/skills/<skill-name>/` — **never** directly inside `~/.agents/skills/`, `~/.claude/skills/`, or `~/.gemini/config/skills/`.

## 2. Gather & Align on Requirements

Before writing files, inspect any reference link/file or description the user gave, and briefly clarify anything missing (skip questions already answered by the user):

1. **Skill name:** Lowercase letters, numbers, and hyphens only (e.g. `my-skill`). Check `<repo>/skills/` to avoid colliding with an existing skill.
2. **Behavior & portability:** Clarify the core workflow and adapt any harness-specific tool calls (e.g., Cursor-only tools) so the skill works cleanly across Codex, OpenCode, Claude Code, and Antigravity (preserving T3 Code orchestration tools like `orchestrator_capabilities` / `delegate_task` with native fallbacks when multi-model delegation is intended).
3. **Invocation mode:** Confirm whether to set `disable-model-invocation: true` (explicit manual invocation only, e.g. `/my-skill`) or leave it unset (model can also auto-trigger it when relevant).
4. **Local per-machine config (if needed):** If the skill needs personal settings, models, or machine-specific data, store that config in a separate file inside `skills/<skill-name>/` and plan to ignore it in `<repo>/.gitignore` rather than hardcoding it in `SKILL.md`.

## 3. Create the Skill

1. Create `<repo>/skills/<skill-name>/SKILL.md` following the Agent Skills standard:
   - YAML frontmatter with `name` (exact match to `<skill-name>`), clear `description`, and `disable-model-invocation: true` if requested.
   - Concise, actionable Markdown instructions.
   - **Privacy hygiene:** Never include secrets, personal information, or local machine paths.
2. Update the `## Skills` table in `<repo>/README.md` with the new skill, how to invoke it, and what it does.
3. If the skill uses a local per-machine config file, add its path to `<repo>/.gitignore`.

## 4. Do Not Commit or Push by Default

- **Never** run `git commit` or `git push` automatically after creating a skill.
- Present the created file path and a short summary of the skill.
- Ask the user whether they want to sync symlinks locally and/or commit and push the new skill to the remote repository — if they say yes, follow `<repo>/skills/bstack/SKILL.md` (`bstack`) to handle syncing and publishing.
