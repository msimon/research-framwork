---
description: Re-enter an active deep-research session after context loss (compaction, new conversation, etc.). Reads .rf-session.json and the deep-dive file to reconstruct session state, then continues per deep-research discipline.
---

# Resume

Recover an active `/rf:deep-research` session whose in-conversation context was lost (compaction, session restart, new shell).

Session state lives on disk (`.rf-session.json` + the deep-dive file). This skill reads those, reconstructs what was happening, and hands control back to the deep-research turn cycle.

## Subject-dir resolution

`<subject-dir> = $PWD`. The user must be `cd`'d into a subject folder. If `$PWD` doesn't contain `00-understanding.md`, stop and tell the user to `cd` into the subject directory first.

## What you do

1. **Check for an active session.** Look for `$PWD/.rf-session.json`.
   - If missing: tell the user "No active session in this subject. Run `/rf:deep-research <slug> [seed]` to open one." and stop.
   - If present: parse it. Expected fields: `topic_slug`, `deep_dive_file` (relative path), `seed`, `turn_count`, `started`, `last_turn`.

2. **Load the deep-dive file** at `$PWD/<deep_dive_file>`. Read it in full if short; otherwise read:
   - The top (title, Why this dive, Session summary if present)
   - `## Insights` section
   - `## Open threads` section
   - The **last 3 turns** of `## Conversation log` (walk from the end)

3. **Load supporting context:**
   - `$PWD/00-understanding.md` — Scope, Angle, End goal, and the `## <topic_slug>` section if it exists
   - `$PWD/00-lexicon.md` — skim so you can use terms consistently
   - `$PWD/00-open-questions.md` — entries tagged `[<topic_slug>]`
   - `$PWD/topics/<topic_slug>/landscape.md` — skim if it exists (don't re-read in depth; it was the starting point)

4. **Report session state to the user.** Keep this tight — they know what they were doing, they just need confirmation you're back on track:

   > "Resumed session on **`<topic_slug>`** — seed: *<seed>*.
   >
   > **Turn count:** <N> · **Started:** <date> · **Last turn:** <date>
   >
   > **Recent thread:** <one line summarizing the last 1-2 turns>
   >
   > **Insights so far:**
   > - <bullet per insight, or "none yet">
   >
   > **Open threads:**
   > - <bullet per open thread, or "none">
   >
   > Ready to continue. What's next — follow up on the last turn, or pivot?"

5. **Wait for the user's next message.** Once they reply, that message is **turn N+1** of the session. Apply full deep-research turn discipline:
   - Research if needed
   - Append a new turn block to the deep-dive file under `## Conversation log`
   - Update lexicon for any new terms
   - Increment `turn_count` and update `last_turn` in `.rf-session.json`
   - Promote Insights / Open threads as they firm up
   - Respond with findings + `My read` + **ONE** follow-up question

   See `${CLAUDE_PLUGIN_ROOT}/skills/deep-research/SKILL.md` — "During the session" section — for the full turn protocol. This skill does not re-run setup; it picks up the existing session.

## If the session file is stale or malformed

- **Malformed JSON or missing required fields:** tell the user, show them the file contents, and ask whether to (a) repair manually, (b) close the session (delete the file), or (c) open a fresh one.
- **Referenced deep-dive file doesn't exist:** the session is corrupt. Offer to delete `.rf-session.json` and start a new `/rf:deep-research` session.
- **`last_turn` is more than 7 days old:** flag it. "This session hasn't been touched since <date>. Resume, or close it out and start fresh?" Don't assume — let the user decide.

## What this skill does NOT do

- Does not modify session content — no synthesis, no summary generation. That happens through normal turn cycles or at session close.
- Does not open a new session. If there's no `.rf-session.json`, it stops and points at `/rf:deep-research`.
- Does not run the stopping-rule check on entry. Resume is mechanical re-entry; stopping-rule timing is counted from the session turn cadence, not from resume invocations.
