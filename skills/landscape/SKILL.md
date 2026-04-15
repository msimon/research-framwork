---
description: Produce a substantive overview of one topic — players, economics, workflow, dynamics, open questions. The workhorse skill; most topics only ever need /rf:landscape. Autonomous; uses WebSearch and bundled MCP servers relevant to the topic.
---

# Landscape

Write a real overview of one topic. Broader than `/rf:deep-research`, substantially deeper than `/rf:discover`. Most topics will only ever need this — reserve `/rf:deep-research` for things that matter.

## Inputs

`$ARGUMENTS` = the topic slug (must exist in the subject's `01-topics.md`).

If `$ARGUMENTS` is empty or doesn't match a row, list the `discovered` topics and ask which.

## Subject-dir resolution

Define `<subject-dir>` as the resolved subject path:
- **Invoked directly by the user:** `<subject-dir> = $PWD`. Must be a valid subject folder (`00-understanding.md` exists). If not, stop and tell the user to `cd` into a subject or use `/rf:discover <subject>` to chain from there.
- **Invoked as auto-continuation from `/rf:discover` or `/rf:init-subject`:** inherit `<subject-dir>` from the calling skill's conversation context. Don't try to `cd` — read and write files using the resolved absolute path.

Every `$PWD/...` reference below means `<subject-dir>/...`.

## Preconditions

- `<subject-dir>/00-understanding.md` exists.
- If `<subject-dir>/topics/<slug>/landscape.md` already exists, ask: "Refresh the existing landscape, or skip?" Don't overwrite silently.

## What you do

1. **Read context:**
   - `$PWD/00-understanding.md` — especially Scope, Angle, End goal. Let these shape what sections of the landscape get emphasized.
   - The row for this topic in `$PWD/01-topics.md` (hook, category)
   - `$PWD/00-open-questions.md` — any existing open questions tagged `[<slug>]`

2. **Scope check (conditional pushback).** Before researching, ask yourself: **is the topic well-scoped enough that this landscape has a definite shape?** If yes, proceed silently. If no — if the same slug could produce substantially different landscapes depending on interpretation — push back once with 2-3 specific cuts, one-line reason each, and ideally a suggestion based on the user's End goal. Example:

   > "Before I write this landscape, which cut of `biosimilar-adoption` do you want?
   > - **Commercial/uptake dynamics** — why adoption is slow despite cost savings; PBM and rebate incentives
   > - **Regulatory/manufacturing** — approval pathway, interchangeability, supply
   > - **Clinical** — efficacy parity, switching studies, prescriber attitudes
   >
   > I'd suggest the first given your End goal (founder-mode) — it's where the operational leverage is. Which?"

   Skip this check silently when the topic is already clear. Don't push back as a ritual.

3. **Research substantively.** Use in parallel where possible:
   - `WebSearch` — target 4-8 solid sources. Industry reports, trade press, company websites, long-form explainers.
   - **Bundled MCPs relevant to the topic category:**
     - Clinical/medical → `pubmed` for evidence-base, `clinicaltrials` for active/recent trials
     - Regulatory → `openfda` for drug/device approvals + labels, `sec-edgar` for company risk disclosures
     - Market/competitive/economic → `sec-edgar` for 10-K filings of the top 3-5 players (section extraction: Business, Risk Factors, MD&A)
   - Cite everything. Flag uncertainty. Don't pad thin sections.

4. **Write `<subject-dir>/topics/<slug>/landscape.md`** with this structure (keep sections honest — if you don't have material for one, say so, don't hallucinate).

   **Term discipline for the landscape body:**
   - When introducing a domain-specific term for the first time, spell it out with the abbreviation in parentheses: e.g. "prior authorization (PA)". Use the abbreviation thereafter.
   - When mentioning a proper noun (company, tool, policy) for the first time, give a one-clause gloss: e.g. "EviCore (a utilization-management vendor)".
   - Assume the reader is intelligent but not a domain insider. If something needs a term to understand, define it there — don't hide it in the lexicon.

   ```markdown
   # Landscape: <topic>
   _Generated: <date>_

   ## One-paragraph summary

   ## Key players / companies
   _Who matters, one-line role each. 5-10 entries._

   ## How it works
   _Workflow, value chain, money flow, or mechanism — whichever is the right frame._

   ## Economics
   _Revenue models, unit costs, where value accrues, what's paid vs free._

   ## Current dynamics
   _Recent shifts, contested narratives, trends. Dated._

   ## Open questions
   _What I couldn't resolve, where sources disagree, what to chase with /rf:deep-research._

   ## Sub-topics worth splitting out
   _Candidates for their own row in 01-topics.md. Only list real ones — don't pad._
   ```

5. **Update the lexicon (`<subject-dir>/00-lexicon.md`).** As you write the landscape, track every domain-specific term introduced and append to the lexicon. Rules:
   - **Abbreviations** (PA, UM, PBM, CMS-0057-F, etc.) → Abbreviations table. Columns: Abbrev / Expansion / One-line meaning.
   - **Terms & concepts** (gold carding, step therapy, buy-and-bill, treat-to-target, etc.) → Terms & concepts table. One-line meaning only.
   - **Entities** (EviCore, Cohere Health, ACR, CMS, Epic Payer Platform, etc.) → Entities table. One-line "what it is / what it does".
   - **Don't duplicate** — check the existing lexicon and skip entries already present. If an existing entry is thinner than what you'd now write, improve it.
   - **One-line definitions.** If the definition would need more than one line, it belongs as an Open Question or a sub-topic, not in the lexicon.
   - **No judgment calls.** Flat factual definitions — "a utilization-management vendor owned by Evernorth", not "the big bad payer gatekeeper."

6. **Write `<subject-dir>/topics/<slug>/sources.md`** with numbered citations:
   ```markdown
   # Sources: <topic>

   1. **<title>** — <URL or MCP reference>
      _What it told me:_ <one line>
   2. ...
   ```
   This file accumulates across future `/rf:deep-research` sessions on the same topic — append, don't overwrite if it already exists.

7. **Append a section to `<subject-dir>/00-understanding.md`** under a `## <topic>` heading. Rules:
   - **5-8 lines maximum.** This is the user's mental model — volume hurts it.
   - Firmest takeaways only. No "it depends" hedges. If something's genuinely uncertain, omit it.
   - Written as claims, not as "research showed that..."
   - If a `## <topic>` section already exists, **append** (don't overwrite) and mark the date.
   - If the research stress-tested a `[prior]` from `## Key beliefs / priors`, update that prior: either promote it (remove `[prior]` tag if validated) or flag it (add `[contested]` tag with a one-line reason).

8. **Append new unknowns/contradictions to `<subject-dir>/00-open-questions.md`** under the right section. Tag each with `[<slug>]` so they're greppable. Don't duplicate existing entries.

9. **Update `<subject-dir>/01-topics.md`:** change this topic's status from `discovered` to `landscape`. Leave other rows alone.

10. **If sub-topics emerged** in the landscape's "Sub-topics worth splitting out" section, ask the user: "These sub-topics emerged — add them as new `discovered` rows in `01-topics.md`? (list them)" Only add if they confirm.

11. **Comprehension check (before any strategy questions).** The user just received dense expert material. Before asking them to form hypotheses or positions, make sure they can read the landscape. Report what was written and proactively flag the lexicon:

    > "Landscape written. Files updated: [brief list].
    >
    > **New lexicon entries this landscape added:** [list the new abbreviations + a couple of the most opaque terms/entities]. Full list in `00-lexicon.md` — keep it open while reading the landscape.
    >
    > Before I ask your take on any of this:
    > 1. Does the **structure** in `How it works` hold together for you, or are there steps/handoffs that don't yet make sense?
    > 2. Any **terms** — beyond what I already flagged — that you want spelled out in plain language?
    > 3. Anything that felt **assumed** that shouldn't be?
    >
    > Answer whichever apply. Say `clear` if you're oriented and ready for the strategy-level questions."

    **Iterate until the user signals orientation.** If they ask what a term means, explain in plain language AND improve the lexicon entry inline (upgrade from a thin definition to a clearer one). If they point to a confusing step, re-explain and consider whether the landscape's `How it works` section needs editing for clarity — if yes, edit it. If they flag an assumption, add it to `00-open-questions.md` tagged `[<slug>]`.

    Don't move to step 12 until the user says `clear`, `got it`, `move on`, or similar. This may take 2-5 turns. That's fine — comprehension is the point.

12. **Strategy-level questions (only after comprehension is settled).** Now ask 3 questions that force the user to form a position. Draw them from the user's End goal (founder → buyer hypothesis, wedge, defensibility; investor → market size sanity-check, exit paths; intellectual → most surprising finding, where sources disagree most). Wait for their response. Answers get appended to `<subject-dir>/00-understanding.md` under `## <topic>` as lines prefixed `User's read:`.

13. **Offer deep-research auto-continuation.** After their strategy-level answers are written, offer:

    > "Want to go deeper on `<topic>` now? I can open a `/rf:deep-research` session.
    >
    > Candidate seeds from the landscape's open questions:
    > - **<Question A>** — would resolve <what>
    > - **<Question B>** — would stress-test <which prior or claim>
    > - **<Question C>** — would nail down <which unknown>
    >
    > Pick one, give me a different seed, or say `no` if you'd rather move on (next landscape, or stop)."

    If they pick a seed or give their own:
    - Read `${CLAUDE_PLUGIN_ROOT}/skills/deep-research/SKILL.md`.
    - Execute its instructions with `<subject-dir>` inherited and `$ARGUMENTS` = `<slug> <their chosen seed>`.
    - The session opens in this same conversation; subsequent turns continue the dive (per deep-research's session discipline).

    If they say no, exit cleanly. Don't pressure.

## Quality bar

- Better to write less than to pad. An honest "I couldn't find data on X" is more useful than fabricated specifics.
- Numbers need citations. If a source says "40%" but another says "15%", note both in Open questions, don't average.
- The understanding-file section is the most important output. If it's weak, the landscape was weak.

## What this skill does NOT do

- Does not open a conversational session — this is autonomous write-and-return. For conversation, the user runs `/rf:deep-research` next.
- Does not write to `deep-*.md` files. Those belong to `/rf:deep-research`.
- Does not promote topics to `deep-N` status — only `landscape`.
