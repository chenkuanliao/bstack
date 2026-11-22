---
name: leaving-pr-comments
description: Use whenever leaving comments on GitHub.
---

# Leaving PR Comments

Applies to top-level PR comments, inline review comments, replies to bots or
reviewers, and written explanations used before resolving or dismissing
feedback.

## Attribution Note (Opt-In)

By default, **do not** include an AI attribution note.

Only add the attribution note when the user explicitly triggers it with the
keyword phrase **"with a note"** (e.g., *"leave a comment with a note"*). When
triggered, prefix the comment with this GitHub note callout:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**

Actual comment starts here.
```

When adding the note:

- Use the verified public name of the model that wrote the comment, and the
  user's name or GitHub login.
- If the harness does not expose a model name, use the harness name. Do not
  guess.
- Keep the attribution inside the note and the actual reply outside it. The
  context line must start with the robot emoji. Do not add a horizontal rule,
  Setext underline, heading, signature footer, or a second attribution.
- Never add it to PR descriptions or commit messages.

## Style & Safety

Before posting, check the entire comment, including quoted text and
attachments, for secrets and private details.

Write the comment itself in a direct style:

- Lead with the conclusion.
- Cite the relevant code, behavior, check, or reproduction.
- Keep it concise and avoid filler, canned thanks, or status narration.
- Distinguish verified facts from judgment.

## Preview & User Approval (Required)

Before posting any comment, review, or reply to GitHub:

1. Show the user the exact final Markdown of the comment in chat (including the
   `[!NOTE]` attribution block when requested, and where it will be posted,
   e.g., PR/issue number, file/line, or review thread).
2. Wait for the user's explicit approval (such as "looks good" or "post it")
   before calling any GitHub tool or CLI command to publish it. If the user
   requests edits, revise the draft and get approval on the updated version.

After approval and posting, verify that GitHub returned the comment or review
URL. Resolve a review thread only after its response was posted successfully.
