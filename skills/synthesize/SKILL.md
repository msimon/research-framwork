---
description: Rank problems worth solving across all researched topics. Reads understanding, landscapes, and deep-dives, applies synthesis criteria, outputs a ranked shortlist. Run after several topics have landscape or deep-N status.
---

# Synthesize

Cross-cut all research in a subject and surface the problems worth solving — ranked against the user's own synthesis criteria.

This is the payoff skill. Everything upstream (`/rf:discover` → `/rf:landscape` → `/rf:deep-research`) was building the raw material. This skill distills it into actionable problem statements.

## Inputs

`$ARGUMENTS` — optional. If provided, treated as a filter or focus directive (e.g. "only payer-side problems" or "ignore anything that needs regulatory change"). If empty, synthesize across all topics.

## Subject-dir resolution

`<subject-dir> = $PWD`. Must contain `00-understanding.md`. If not, stop and tell the user to `cd` into a subject first.

## Preconditions

- At least **1 topic** at `landscape` status or higher in `01-topics.md`. Synthesize works best with 3+ landscaped topics (more material to cross-cut), but a single deep-dived topic can surface real problems. If only 1 topic qualifies, note that the ranking is based on a narrow evidence base — don't fabricate breadth.
- `00-understanding.md` must have a `## Synthesis criteria` section. If it's missing or still the default template text, ask the user for criteria before proceeding. Don't guess.

## What you do

### Phase 1 — Load everything

1. **Read `$PWD/00-understanding.md`** in full — especially Scope, Angle, End goal, Synthesis criteria, and Key beliefs/priors.
2. **Read `$PWD/01-topics.md`** — identify all topics at `landscape` status or above.
3. **For each qualifying topic, read:**
   - `$PWD/topics/<slug>/landscape.md` — full
   - Any `$PWD/topics/<slug>/deep-*.md` files — read `## Session summary` and `## Insights` sections (skip the full conversation log unless you need to chase a specific claim)
   - `$PWD/00-open-questions.md` — entries tagged `[<slug>]`
4. **Read `$PWD/00-lexicon.md`** — so you use terms consistently.

### Phase 2 — Extract candidate problems

Scan all loaded material for **problems worth solving** — not topics, not observations, but specific pain points that a product/service/intervention could address. A good problem statement has:

- **Who** has the problem (specific actor, not "the healthcare system")
- **What** the pain is (operational, financial, clinical, regulatory)
- **Why** it persists (incentive misalignment, information gap, regulatory barrier, inertia)
- **How big** it is (quantified if possible — from the research, not invented)

Extract 8–15 candidate problems. Don't inflate — if the research only supports 5, list 5. Don't merge distinct problems to hit a count.

### Phase 3 — Score against synthesis criteria

Take the user's `## Synthesis criteria` from `00-understanding.md` and score each candidate. Use a simple 3-level scale per criterion:

| Level | Meaning |
|---|---|
| **Strong** | Clear evidence from research supports this |
| **Mixed** | Some evidence, but contested or incomplete |
| **Weak** | Speculative or no evidence found |

Don't fabricate precision. If the research didn't cover a criterion for a given problem, mark it `No data` — that's a signal, not a failure.

### Phase 4 — Rank and write

1. **Rank problems** by overall strength across criteria. Weight the user's End goal: founder → over-weight pain + willingness-to-pay + defensibility; investor → over-weight market size + exit paths; selling-into → over-weight buyer access + implementation difficulty. State your weighting logic explicitly.

2. **Write `$PWD/synthesis/problems-ranked.md`:**

   ```markdown
   # Synthesized problems: <subject>
   _Generated: <date>_
   _Criteria: <list from 00-understanding.md>_
   _Filter: <$ARGUMENTS if any, else "none">_
   _Topics included: <list of slugs and their status>_

   ## Ranking methodology
   _One paragraph: how you weighted the criteria given the user's End goal._

   ## Ranked problems

   ### 1. <Problem title>
   **Who:** <actor>
   **Pain:** <what hurts>
   **Why it persists:** <root cause>
   **Scale:** <quantified if possible>
   **Criteria scores:**
   | Criterion | Score | Evidence |
   |---|---|---|
   | <criterion> | Strong/Mixed/Weak/No data | <one-line citation or reference to topic> |
   **Source topics:** <slugs>
   **Key uncertainty:** <what you'd need to verify>

   ### 2. ...
   (repeat for all ranked problems)

   ## Below the cut
   _Problems considered but not ranked — too speculative, too small, or blocked by something structural. Listed so the user knows they were evaluated, not missed._

   ## Cross-cutting observations
   _Patterns that emerged across problems — shared root causes, common blockers, adjacent opportunities._
   ```

3. **Update `$PWD/00-understanding.md`:** append a `## Synthesis` section (or update if it exists) with:
   - Top 3 problems (one line each)
   - The main cross-cutting pattern
   - Date

4. **Append new unknowns to `$PWD/00-open-questions.md`** tagged `[synthesis]` — things you noticed were missing across multiple topics that would change the ranking if answered.

5. **Commit:** `git add <subject-dir>/ && git commit -m "rf: synthesize — <N> problems ranked"`

### Phase 5 — Review with the user

Present the ranking and ask for their reaction. This is not a hand-off — it's a checkpoint:

> "Here's the ranked list. Before you act on it:
>
> 1. **Does the #1 problem feel right?** Sometimes the data says one thing but domain intuition says another — if so, tell me why and I'll re-weight.
> 2. **Any problem missing?** Something you've been circling that the research didn't surface cleanly.
> 3. **Any problem that should be killed?** Structurally blocked, already solved, or just not interesting to you.
>
> I'll revise the ranking based on your input, then we can go deep on the top 1–2 with `/rf:solution`."

If they give feedback:
- Revise `synthesis/problems-ranked.md` in place (don't create a v2 file — edit the ranking)
- Note their input as `User's read:` annotations on the affected problems
- Re-rank if the feedback materially changes the order

### Phase 6 — Offer continuation

Once the user is satisfied with the ranking:

> "Ready to explore solutions for the top problem? `/rf:solution <slug>` will map existing solutions, find gaps, and sketch differentiation.
>
> Top candidates:
> - **<Problem 1 title>** — <one-line hook>
> - **<Problem 2 title>** — <one-line hook>
>
> Pick one, or tell me which problem to target.
>
> You can also skip synthesize in the future and go straight to `/rf:solution` from a deep-dive when a problem jumps out — it doesn't have to come through this ranking."

If they pick one:
- Read `${CLAUDE_PLUGIN_ROOT}/skills/solution/SKILL.md`
- Execute with `<subject-dir>` inherited and `$ARGUMENTS` = the chosen problem's slug/title

## Quality bar

- **Every claim in the ranking must trace back to a specific topic's research.** No "it's well known that..." without a source topic.
- **The ranking is only as good as the criteria.** If the criteria are vague, the ranking will be vague. Push back on criteria quality if needed.
- **"Below the cut" matters.** The user needs to see what was evaluated and rejected, not just what ranked. Silent omission breeds distrust.
- **Don't average away disagreement.** If two topics give contradictory signals on the same problem, flag it — don't split the difference.

## What this skill does NOT do

- Does not do new research. Works from existing files only. If the research is thin, say so — don't web-search to fill gaps (that's what `/rf:landscape` and `/rf:deep-research` are for).
- Does not write solution sketches. That's `/rf:solution`.
- Does not open a session. This is autonomous write-and-review, like `/rf:landscape`.
