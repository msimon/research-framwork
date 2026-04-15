# Research subject: {subject}

This directory is a research subject managed by the `rf` plugin. Use `/rf:` skills to operate on it.

## File layout

- `00-understanding.md` — mental model. Skills append; user curates.
- `00-lexicon.md` — running glossary. Skills append on every relevant turn. Grep here for term definitions.
- `00-open-questions.md` — contradictions, unknowns, parked threads. Entries tagged `[<slug>]`.
- `01-topics.md` — topic index. Status: `discovered` → `landscape` → `deep-N`.
- `topics/<slug>/landscape.md` — `/rf:landscape` output.
- `topics/<slug>/deep-<seed>-DATE.md` — `/rf:deep-research` session transcripts (one per session).
- `topics/<slug>/sources.md` — accumulated citations across the topic.
- `.rf-session.json` — **if present, a deep-research session is ACTIVE in this subject.** See protocol below.

## Active session protocol — READ THIS ON EVERY TURN

Before responding to any user message in this subject:

1. **Check for `.rf-session.json`.** If present, a deep-research session is open. The user's message is a **session turn**, not a free-form question. Do NOT treat it as a new topic, do NOT skip file updates, do NOT answer conversationally without touching the files.

2. **When a session is active, every turn does all of the following:**
   - Read the deep-dive file referenced in `.rf-session.json` to see recent turns (at minimum the last 2-3 turns, plus Insights and Open threads).
   - Research if needed (WebSearch, MCPs).
   - Append a new turn block to the deep-dive file under `## Conversation log`:
     ```markdown
     ### Turn N — <short paraphrase>
     **User:** <their message, verbatim or close paraphrase>
     **Findings:** <research results with citations>
     **My read:** <interpretation, flagged as interpretation>
     ```
   - Scan findings/response for new domain-specific terms → append to `00-lexicon.md` (Abbreviations / Terms & concepts / Entities tables). Skip duplicates.
   - Update `.rf-session.json`: increment `turn_count`, update `last_turn` timestamp.
   - Every ~5 turns: promote firm Insights to `00-understanding.md`, mirror Open threads to `00-open-questions.md` (tagged `[<slug>]`).
   - Respond to the user: findings summary + interpretation + **ONE** follow-up question. Define any new term inline on first mention — don't rely on the lexicon alone.
   - Every ~10 turns OR when the user signals wrap-up: run the stopping-rule check (can you explain this in 3 min? know the 5 players? know the unit economics? parked the rest?).

3. **Session close.** If the user invokes another `/rf:` skill, or says "stop" / "done" / "switch topics" / agrees to the stopping-rule check:
   - Write `## Session summary` at the top of the deep-dive file (3-6 bullets of firmest takeaways).
   - Promote Insights to `00-understanding.md` under `## <topic>`.
   - Promote Open threads to `00-open-questions.md` tagged `[<slug>]`.
   - Bump status in `01-topics.md`: `landscape` → `deep-1`, `deep-1` → `deep-2`, etc.
   - **Delete `.rf-session.json`.** Session is closed.

4. **If `.rf-session.json` does not exist**, treat the message as a normal skill invocation or free-form question. No special handling.

## If you're not sure whether a session is active

Run `ls -la .rf-session.json` (or equivalent). The file is the source of truth. Don't rely on conversation memory — it doesn't survive compaction.

## Core conventions (apply always)

- **First use of a domain term:** spell it out with abbreviation in parens (e.g. "prior authorization (PA)"). Use the abbreviation thereafter.
- **Findings vs My read:** `**Findings:**` is sourced (cite); `**My read:**` is interpretation. Never mix.
- **One follow-up question per turn.** Not three. Not a list. One.
- **Write to files in real time**, not in a batch at the end. The user should see files update as the turn progresses.
- **Push back only when ambiguity would materially change output.** Principle over checklist.

## Plugin skills available

- `/rf:discover [subject-name]` — enumerate topics
- `/rf:landscape <slug>` — substantive overview
- `/rf:deep-research <slug> [seed]` — open a conversational session (creates `.rf-session.json`)
- `/rf:resume` — re-enter an active session after context loss
