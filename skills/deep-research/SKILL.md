---
description: Conversational deep-dive on one topic. Opens a SESSION — every subsequent user message is part of the same investigation until they run another /rf: skill or say "we're done". Use after /rf:landscape when one topic genuinely needs to go further.
---

# Deep Research

Go deep on one topic, conversationally. This skill **opens a session**: once invoked, every user message in this conversation is part of the same deep-dive until the user:
- invokes another `/rf:` skill,
- explicitly says "we're done" / "stop" / "switch topics", or
- clearly changes subject.

Treat this as a guided conversation, not a report-generation task.

## Inputs

`$ARGUMENTS` format: `<topic-slug> [optional seed question]`

- `topic-slug` must exist in the subject's `01-topics.md`. If missing or invalid, list landscape-status topics and ask.
- A `landscape.md` for the topic is *ideal* but not required. If it's missing, warn: "No landscape for this topic yet — I'd recommend `/rf:landscape <slug>` first so this dive has ground to build on. Continue anyway?"

## Subject-dir resolution

Define `<subject-dir>` as the resolved subject path:
- **Invoked directly by the user:** `<subject-dir> = $PWD`. Must be a valid subject folder. If not, stop and tell the user to `cd` into a subject first.
- **Invoked as auto-continuation from `/rf:landscape` or `/rf:discover`:** inherit `<subject-dir>` from the calling skill's conversation context. Don't try to `cd`.

Every `$PWD/...` reference below means `<subject-dir>/...`.

## Session setup (first turn ONLY)

1. **Read context:**
   - `$PWD/00-understanding.md` — especially Scope, Angle, End goal, and the topic section if it exists
   - `$PWD/topics/<slug>/landscape.md` and `sources.md` if they exist
   - `$PWD/00-open-questions.md` — filter to entries tagged `[<slug>]`

2. **Sharpen the seed question (conditional pushback).** A deep-dive is only as good as its anchoring question. Ask yourself: **is the seed question sharp enough that a session has a definite shape?** A sharp seed is specific, operational, or a falsifiable hypothesis.

   - **If sharp** → proceed directly, no pushback.
   - **If missing** → don't ask the open "what do you want to know?". Propose 2-3 concrete seeds derived from the topic's `landscape.md` Open Questions and the entries in `00-open-questions.md` tagged `[<slug>]`. Name what each would resolve. Let the user pick or redirect.
   - **If vague** (e.g. "how does X work" when landscape already covers the overview) → push back with 2-3 concrete sub-question options, one-line reason each. Don't ask generically to "narrow."

   One round of pushback max. If the user prefers broad exploration, accept and note in the deep-dive file: `*Seed narrowed by skill? No — user preferred broad exploration.*`

3. **Create the deep-dive file:** `$PWD/topics/<slug>/deep-<seed-slug>-<YYYY-MM-DD>.md`
   - `<seed-slug>` = a short slug of the seed question (e.g. "how does X work" → `how-x-works`). If no seed question, use `general`.
   - If a file with that exact name already exists (re-entering a same-day dive on the same question), append a `-2`, `-3`, etc.
   - Initialize with this skeleton:
     ```markdown
     # Deep research: <topic> — <seed question or "general">
     _Started: <YYYY-MM-DD>_

     ## Why this dive
     _One line: what are we trying to learn that landscape.md didn't answer?_

     ## Session summary
     _Written at session close. Empty while the dive is active._

     ## Conversation log
     _Turn-by-turn. User question + researched response + interpretation, one block per turn._

     ## Insights
     _Standalone findings promoted here when they're firm enough to stand alone (not just answer to one question)._

     ## Open threads
     _Questions this session spawned but didn't close. Mirrored to 00-open-questions.md at session close._
     ```

4. **Fill "Why this dive"** with a one-line version of the sharpened seed question from step 2.

5. **Create the session marker file** `$PWD/.rf-session.json`. This is the on-disk source of truth for whether a session is open — it survives context compaction, conversation restart, and `/clear`. Structure:
   ```json
   {
     "topic_slug": "<slug>",
     "deep_dive_file": "topics/<slug>/deep-<seed-slug>-<YYYY-MM-DD>.md",
     "seed": "<sharpened seed question>",
     "turn_count": 0,
     "started": "<ISO timestamp>",
     "last_turn": "<ISO timestamp>"
   }
   ```
   If this file already exists (a session is somehow already open), stop and tell the user: "A session is already active on `<existing topic_slug>`. Close it first (say 'done' or invoke another `/rf:` skill) or run `/rf:resume` to re-enter it."

6. **Recommend `cd` if the user isn't already there.** If the conversation started from outside the subject folder (e.g. auto-continued from `/rf:landscape` invoked from a parent directory), tell the user:

   > "Heads up: for long sessions, I recommend you `cd <subject-dir>` in a new terminal / or start a fresh Claude Code session from inside the subject folder. The subject's `CLAUDE.md` will then auto-load — which makes session state resilient to context compaction. You can also run `/rf:resume` from inside the subject at any time to recover."

   Don't block on this — proceed with the session regardless. It's guidance, not a requirement.

7. **Commit:** `git add <subject-dir>/ && git commit -m "rf: open deep-research session on <slug> — <seed-slug>"`

8. **Treat the sharpened seed as turn 1** and proceed to "During the session".

## During the session (EVERY turn after setup)

**Before anything else every turn:** check that `$PWD/.rf-session.json` still exists. If it's gone, the session was closed (by another skill or by the user deleting it) — stop, tell the user, and do not append to the deep-dive file. If it exists but you've lost conversation context, treat the turn as a resume: read the deep-dive file's recent turns, Insights, and Open threads before responding. (Or instruct the user to run `/rf:resume`.)

For each user message while the session is open:

1. **Decide whether research is needed — and whether the question is sharp enough to research.**
   - **Only if the question could mean two substantially different things**, push back FIRST with the interpretations and ask which. Don't manufacture ambiguity — if the question is clear in context, just answer it. Example of genuine ambiguity: *"'How does adoption work' could mean (a) prescriber decision-making at the point of care, or (b) payer formulary decisions that shape what prescribers can do. Which — or both?"*
   - If the question is clear and the landscape/sources already answer it, reference that — don't re-search.
   - If the question is clear and new, research with `WebSearch` and relevant MCPs. Be targeted.
   - If they're sharing a hypothesis or observation, engage with it — ask what makes them believe it, stress-test it, point to evidence for/against.

2. **Append to "## Conversation log"** a new turn block:
   ```markdown
   ### Turn N — <short paraphrase of the question/topic>
   _<timestamp HH:MM>_

   **User:** <verbatim, or close paraphrase if they dictated long-form>

   **Findings:** <what you learned, with citations>

   **My read:** <your interpretation — flagged as interpretation, not fact>

   **User's read** _(added when they respond):_ <their response next turn, pasted back when they engage>
   ```

3. **Update the lexicon (`<subject-dir>/00-lexicon.md`).** Scan your response for new domain-specific terms, abbreviations, or entities not already in the lexicon. Add them to the right table (Abbreviations / Terms & concepts / Entities) with a one-line definition. Skip duplicates. Same discipline as `/rf:landscape` step 5. Also: in your reply text to the user, define new terms inline on first mention — don't rely on the lexicon alone for readability.

4. **Promote to "## Insights"** anything that's now firm enough to stand on its own — i.e. you'd state it as a belief, not as an answer to a question. Don't inflate insights; 1-3 per session is healthy.

5. **Track open threads** in "## Open threads" — questions that this turn spawned but didn't close. Don't answer them; park them.

6. **Respond to the user with:**
   - The findings (concise — they'll see the full version in the file). Define any new term inline on first use.
   - Your `My read` interpretation
   - **ONE** follow-up question that sharpens the inquiry. Not three. Not a list. One. Always prefix it with "Follow-up Question:" to make it clear it's a question, not a statement.

7. **Update the session marker.** Increment `turn_count` and set `last_turn` to the current ISO timestamp in `$PWD/.rf-session.json`. This is how `/rf:resume` knows where the session stands.

8. **Commit:** `git add <subject-dir>/ && git commit -m "rf: deep-research <slug> turn N — <short paraphrase>"` where N is the turn number and `<short paraphrase>` matches the turn block header.

9. **Every ~5 turns, mirror outward:**
   - Propagate any new Insights to `<subject-dir>/00-understanding.md` under the topic section (only if they're firm enough to be beliefs, not findings).
   - Propagate Open threads to `<subject-dir>/00-open-questions.md` tagged `[<slug>]`.

## Sub-topic splitting

If the conversation is clearly branching into a distinct sub-area (not a follow-up, a genuinely different thread), surface it:

> "This feels like its own sub-topic — want me to add `<parent>-<sub>` as a new `discovered` row in `01-topics.md`? We can keep going on the current thread or switch."

Let the user decide. Don't split unilaterally.

## Stopping rule

Proactively run this check:
- Every ~10 turns, OR
- When the user seems to be wrapping up ("makes sense", "I think I get it", "ok"), OR
- When you notice you've been going in circles

Summarize what the session has covered so far (major findings, key threads), then name the open threads that remain. Ask:

> "We've covered [concise recap of ground covered]. We can keep going on the same thread if there's more to pull on.
>
> We also have these open threads parked: [list them]. Want to dig into any of these, or save them for a future session?"

Three outcomes:
- **Keep going on the current thread** — the user wants to go deeper on what they were already exploring. Continue the session as normal.
- **Pivot to an open thread** — they pick one from the parked list. Continue the session on that thread (still the same session, just a pivot).
- **Done** — they want to park everything. Run **session close**.

## Session close

1. **Write "## Session summary"** at the top of the deep-dive file: 3-6 bullets of the firmest takeaways from the dive. This is what future-you (or specialist agents in `/rf:synthesize`) will read.
2. **Promote Insights** to `$PWD/00-understanding.md` under the topic section.
3. **Promote Open threads** to `$PWD/00-open-questions.md` tagged `[<slug>]`.
4. **Update `$PWD/01-topics.md`:** bump status. `landscape` → `deep-1`, `deep-1` → `deep-2`, etc. Count is total deep-dive files for this topic.
5. **Delete `$PWD/.rf-session.json`.** The session is closed — the marker must go so future turns don't mistake this conversation for an ongoing session.
6. **Commit:** `git add <subject-dir>/ && git commit -m "rf: close deep-research session on <slug> — <turn_count> turns"`
7. **Close out the conversation** with:
   - A 2-line recap of what was added to `00-understanding.md`
   - Suggested next moves: another `/rf:deep-research` on a sibling thread, a `/rf:landscape` of the sub-topic if one was spun off, or — if several topics have real depth now — `/rf:synthesize` (coming later).

## Session discipline

- **You are IN SESSION** until the user invokes another `/rf:` skill, explicitly stops, or clearly changes subject. Default assumption every turn: this message continues the dive.
- **Don't re-run the landscape.** If the user asks something `landscape.md` covers, reference the file — don't re-search.
- **One follow-up question per turn.** Deep research is a guided conversation, not an interview.
- **Flag evidence vs interpretation.** `**Findings:**` is sourced (cite); `**My read:**` is interpretation. Never mix.
- **Don't be sycophantic.** If the user's hypothesis is wrong or partial, say so with evidence — they need a thinking partner, not a yes-machine.
