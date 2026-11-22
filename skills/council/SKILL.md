---
name: council
description: "Run a multi-model council of delegates to review plans, code, PRs, or ideas and score quality out of 10. Without a target score (/council), runs 1 read-only feedback round; with a target score (/council 8), iterates up to 3 rounds (or a user-specified cap) until all delegates reach the target. Also sets up and manages the local delegate roster."
disable-model-invocation: true
---

# Council

Bring in a user-configured roster of AI delegates to review a plan, codebase change, PR, doc, or decision in parallel, score it out of 10, and synthesize clear, actionable feedback.

## 1. Roster Configuration (`delegates.json`)

The delegate roster is stored locally beside this skill at `<skill-dir>/delegates.json` (gitignored so personal model/provider choices are never committed to the public repo).

**Locate `<skill-dir>`:**
- If `./skills/council/SKILL.md` exists in the current workspace, use `./skills/council`.
- Otherwise, resolve the symlink of `~/.agents/skills/council`, `~/.claude/skills/council`, or `~/.gemini/config/skills/council` (`cd -P` / `pwd -P`).

**Roster schema (`<skill-dir>/delegates.json`):**
```json
{
  "delegates": [
    {
      "name": "Display Name",
      "provider": "provider-or-harness-name",
      "model": "model-id",
      "options": {
        "effort": "high"
      }
    }
  ]
}
```
- Keep entries minimal: `name`, `provider` (harness/provider), `model`, and `options` (thinking/reasoning settings, `{}` if none).

### First-Time Setup & Roster Management
- **Missing or empty roster:** If `<skill-dir>/delegates.json` does not exist or `delegates` is empty:
  1. In T3 Code, call `orchestrator_capabilities` first so you can show the user the actual available providers, models, and thinking/effort options on their machine.
  2. Ask the user interactively which delegates they want (how many, and for each: provider/harness, model, and thinking/reasoning level). Wait for their reply.
  3. Save `<skill-dir>/delegates.json`.
  4. If the user also provided a review task, proceed to run the council; otherwise confirm the saved roster and finish.
- **Managing the roster anytime:** If the user asks to view, add, remove, edit, or reconfigure delegates (e.g., *"list council delegates"*, *"add a delegate"*, *"remove X"*, *"change Y's thinking level"*, *"setup council"*):
  - In T3 Code, call `orchestrator_capabilities` when adding or changing models/options so choices match valid catalog entries.
  - Update `<skill-dir>/delegates.json` and show the updated roster.

---

## 2. Launching Delegates

- **In T3 Code:** Call `orchestrator_capabilities` once per thread before the first `delegate_task` to resolve each delegate's `providerInstanceId`/`driverKind`, `model`, and `options`. Launch **all** configured delegates in **parallel** with `delegate_task` (`role: "review"`). Always start **fresh** `delegate_task` runs for every round — never reuse an earlier round's delegate thread.
- **Outside T3 Code:** Use the harness's native subagent tools to run the configured delegates in parallel (or inform the user if cross-provider/model delegation is unavailable in the current harness).
- **Unavailable members:** Never silently substitute a different provider or model. If a configured delegate is unavailable, report it clearly and ask the user how to proceed.

### Delegate Brief
Give every delegate the exact artifact or context under review (plan, diff, doc, design question) plus any specific instructions the user provided. Instruct every delegate to:
1. Work **read-only** (never modify files).
2. Return a **quality score out of 10** and briefly explain what is holding it back from a 10.
3. Provide concise, high-signal feedback covering:
   - **Issues / risks** (blocking flaws or gaps)
   - **Suggestions / tradeoffs**
   - **Strengths** (what works well)

---

## 3. Execution Modes

Determine the mode from the user's invocation:

### Mode A: Feedback Only (No Target Score — e.g., `/council` or `/council review this plan`)
- Run **1 round only**. Do **not** edit or modify the plan, doc, or code.
- Launch all configured delegates in parallel and collect their responses.
- Present a clean, scannable summary in the chat reply:
  1. **Score table** (if a delegate gives no score, write `no score` — never invent one):
     ```markdown
     ## Council review

     | Delegate | Score |
     | --- | --- |
     | Delegate A | 7/10 |
     | Delegate B | 8/10 |
     ```
  2. **Categorized feedback:**
     - **Consensus** — points multiple delegates agreed on
     - **Disagreements & Tradeoffs** — where delegates differed, with your concise take
     - **Strengths** — what the council liked
     - **Recommended Next Steps** — concrete options for what to refine next

### Mode B: Iterative Refinement (Target Score Given — e.g., `/council 8` or `/council 9 review this plan`)
- Parse the target score `T` out of 10 (every delegate must reach `>= T`).
- **Round cap:** Default to **at most 3 rounds**. Honor a higher or lower cap if the user explicitly specifies one (e.g., *"max 5 rounds"*) or asks for extra rounds afterward.
- **Per-round loop (`round = 1 .. max_rounds`):**
  1. Launch **all** configured delegates in parallel (read-only) on the current version of the artifact.
     - Always re-run **all** delegates on each fresh round so fixes for one delegate don't introduce regressions for another.
     - On Round 2+, include a brief summary of what changed since the previous round plus any intentional pushback/objections to prior feedback so delegates don't re-litigate settled points.
  2. **Never edit files while any delegate is still inspecting the same checkout.** Wait until the round finishes before making edits.
  3. Record each delegate's score (`no score` if omitted — never invent one, and never count `no score` as meeting `T`).
  4. **Check stop conditions:**
     - **All delegates scored `>= T`** → stop; target met.
     - **Reached `max_rounds`** → stop; report the current state, remaining gaps, and note that the user can ask for extra rounds if desired.
  5. **Update & continue:** Evaluate the feedback in the main thread, **fight scope creep** (push back on out-of-scope requests rather than bloating the plan/code to chase a number), apply the valid improvements you agree with, and begin the next fresh round.
- Present the final summary in the chat reply:
  ```markdown
  ## Council review

  Rounds: 2 (Target: 8/10)

  | Delegate | Round 1 | Round 2 |
  | --- | --- | --- |
  | Delegate A | 7/10 | 9/10 |
  | Delegate B | 8/10 | 9/10 |
  | Delegate C | 6/10 | 8/10 |
  ```
  Follow the table with a concise summary of:
  - **Key improvements made** across rounds
  - **Remaining issues or disagreements** (if stopped at the round cap or pushed back on scope)
  - **Current status** (target met vs. stopped at round cap)

---

## 4. Guardrails

- **Chat reply only:** Present results in the chat reply. Do not automatically edit PR descriptions or post PR comments unless the user explicitly asks.
- **Fight scope creep:** Keep changes tightly focused on the user's actual goal across rounds.
- **Honest reporting:** Never fabricate scores, hide delegate disagreements, or silently swap models.
