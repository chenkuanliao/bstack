# bstack

These are the skills I use every day. They are meant to be flexible, so feel free to fork this repo and change them however you want. You should tweak them to fit how you work.

They work in Codex, OpenCode, Claude Code, and Antigravity. `council` uses [T3 Code](https://github.com/pingdotgg/t3code) delegation when available and falls back to native subagents when it is not.

## Skills

| Skill | Say | What it does |
| --- | --- | --- |
| `bstack` | "sync my bstack" / "update my stack" | Sets up symlinks across harnesses, pulls updates, and commits and pushes local `bstack` changes to your remote. |
| `add-skill` | "use add-skill to create ..." | Creates a new skill in `skills/<name>/SKILL.md` and updates this README without committing or pushing by default. |
| `council` | `/council` or `/council 8` | Gets scored feedback out of 10 from your configured delegates in parallel. Add a target score (like `/council 8`) to iterate up to 3 rounds until every delegate meets it. |
| `how` | `/how` | Spawns a read-only exploration subagent to explain how a subsystem works. |
| `why` | `/why` | Answers why code works a certain way from git commits, PR discussions, and history. |
| `bro` | `/bro` | Restates the last message in plain human language with no jargon. |
| `in-your-own-words` | `/in-your-own-words` | Has the agent restate your goals and problem before doing work. |
| `unslop` | `/unslop` | Edits writing to strip AI patterns and filler. |
| `leaving-pr-comments` | (automatic) / "with a note" | Writes concise GitHub comments, checks for secrets, asks for approval before posting, and adds an AI attribution note only when you say "with a note". |

*(In Codex, invoke manual skills with `$skill-name` or by name; in Antigravity, ask for the skill by name. `add-skill` hands off to `bstack` if you choose to sync or push after creating a skill.)*

## Install

1. Clone this repository and open your AI coding agent in the cloned folder.
2. Paste this prompt to your agent:

```text
Read and follow skills/bstack/SKILL.md to set up bstack on my machine.
```

Your agent will scan which harnesses are installed, ask which ones you want to link, and symlink each folder in `skills/` without overwriting existing local skills:

| Harness | Global Skills Directory | Detection |
| --- | --- | --- |
| **Codex** | `~/.agents/skills` | `codex` in `PATH`, `~/.agents`, or `~/.codex` |
| **OpenCode** | `~/.agents/skills` | `opencode` in `PATH` or `~/.config/opencode` |
| **Claude Code** | `~/.claude/skills` | `claude` in `PATH` or `~/.claude` |
| **Antigravity** | `~/.gemini/config/skills` | `agy` in `PATH` or `~/.gemini` |

*(Codex and OpenCode share `~/.agents/skills` and are deduplicated automatically when both are selected.)*

## Requirements

- Zero runtime dependencies (no Node.js, npm, or Python required).
- `git` for pulling/pushing with `bstack` and reading commit history with `why`.
- An authenticated [GitHub CLI](https://cli.github.com) (`gh`) or GitHub PR tools for `leaving-pr-comments` (and PR history in `why`).
- [T3 Code](https://github.com/pingdotgg/t3code) for cross-provider delegates in `council` (falls back to native subagents outside T3 Code).

## Make them yours

Treat these as a starting point and change whatever does not fit how you work:

- Use `add-skill` to create new skills, and `bstack` to sync symlinks or push updates to your own fork.
- Local skills already in your harness folders stay untouched and are never overwritten, imported, or pushed.
- Your `council` model list lives locally in `skills/council/delegates.json` (gitignored), so you can pull updates without losing or exposing your setup.

## License

[MIT](LICENSE)
