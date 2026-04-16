---
description: Enumerate candidate research areas for a subject. Broad and shallow — surfaces 15-30 topics ranked by likely importance. Autonomous; uses WebSearch and any bundled MCP servers relevant to the subject. Accepts an optional subject-folder name so you don't have to cd.
---

# Discover

Find the areas worth researching inside a subject. This is the **first pass** — deliberately shallow and wide. No deep dives, no judgments on what's best to solve.

## Inputs

`$ARGUMENTS` = optional subject-folder name (e.g. `rheumatology`). If provided and `$PWD/$ARGUMENTS/00-understanding.md` exists, operate on `$PWD/$ARGUMENTS/`. If not provided, operate on `$PWD` (the current folder must be a subject folder).

Define `<subject-dir>` as the resolved path. Every `$PWD/...` reference below means `<subject-dir>/...`.

## Preconditions

Resolve `<subject-dir>`:
1. If `$ARGUMENTS` is given: check `$PWD/$ARGUMENTS/00-understanding.md` and `$PWD/$ARGUMENTS/01-topics.md` exist. If yes → `<subject-dir> = $PWD/$ARGUMENTS`. If no → stop and tell the user: "No subject folder found at `$PWD/$ARGUMENTS/`. Run `/rf:init-subject $ARGUMENTS` first."
2. If `$ARGUMENTS` is empty: check `$PWD/00-understanding.md` and `$PWD/01-topics.md` exist. If yes → `<subject-dir> = $PWD`. If no → stop and tell the user: "Not in a subject folder, and no subject name given. Either `cd` into a subject, or run `/rf:discover <subject-name>`."

## What you do

1. **Read context:**
   - `<subject-dir>/00-understanding.md` — Scope, Angle, End goal, Priors, Synthesis criteria.
   - `<subject-dir>/01-topics.md` — so you don't duplicate already-discovered topics.

2. **Framing check (conditional pushback).** Before enumerating, ask yourself: **is anything in the framing ambiguous in a way that would materially change which topics get enumerated?** If yes, push back on that one dimension with 2-3 concrete options and a one-line reason each matters. If no, skip this step silently and proceed.

   Examples of when to push back:
   - The subject is geography-sensitive and Scope left geography open, so the topic list for US vs EU would be substantially different.
   - Angle is "all of them" — the topic list would be diffuse; ask which to emphasize.
   - A flagged `[prior]` would dramatically narrow or widen the topic list and the user hasn't signaled which.

   Examples of when NOT to push back:
   - Subject is effectively global (e.g. open-source tooling) — don't ask geography.
   - Framing is thin overall but no single dimension is load-bearing for enumeration.
   - The interview in `/rf:init-subject` already resolved the relevant dimensions.

   Principle over checklist. Do not push back as a ritual. Frame pushback as teaching — options + one-line reasons, not "be more specific."

3. **Enumerate candidate topics.** Target 15-30. Use sources in parallel:
   - `WebSearch` for "landscape of X", market overviews, recent news, "how does X work", "problems in X".
   - **Bundled MCP servers if relevant to the subject:**
     - `pubmed` — biomedical/clinical subjects
     - `clinicaltrials` — drug/therapy/medical-device subjects
     - `openfda` — drug/device regulatory subjects
     - `sec-edgar` — any space with material public companies (look up top 3-5 and skim section headers of their most recent 10-K — free topic ideas)
   - Your prior knowledge, but flag anything uncertain with `(?)` so the user knows.

4. **For each topic produce:**
   - `slug` — short, lowercase, hyphen-separated (max ~4 words)
   - `category` — one of: `market`, `clinical`, `regulatory`, `operations`, `technology`, `competitive`, `economic`, `other`
   - `hook` — one line, ≤15 words, stating *why it's interesting* (not what it is). The hook is what sells the user on spending a `/rf:landscape` on it.

5. **Rank** (roughly) by:
   - Likely impact on understanding the space / building a business in it, given the user's End goal
   - Alignment with their Angle (business/clinical/etc.) — weight on-angle topics higher
   - Breadth of downstream inquiry (rich topics beat narrow ones at this stage)
   - Non-obviousness — the user already knows what they know; surface what they don't
   - Tension with their Priors — topics that would stress-test a prior are high-value

6. **Append to `<subject-dir>/01-topics.md`** as new table rows. Preserve any existing rows. Set `status = discovered`. Don't overwrite the file.

7. **Commit:** `git add <subject-dir>/01-topics.md && git commit -m "rf: discover topics for <subject>"`

8. **Summarize to the user:** print the top 10 topics with their hooks (not all 15-30 — respect their attention). Group by category if that helps scannability. Ask:
   > "Which 2-3 should we `/rf:landscape` first? Reply with slugs — I'll run landscape on them in order without you having to re-invoke."

9. **Auto-continue to `/rf:landscape` if the user picks topics.** When the user replies with one or more slugs:
   - Validate each slug exists in `<subject-dir>/01-topics.md`.
   - Read `${CLAUDE_PLUGIN_ROOT}/skills/landscape/SKILL.md`.
   - For each slug in order: execute landscape's instructions against `<subject-dir>/topics/<slug>/`. Carry `<subject-dir>` through — don't try to `cd` (shell state doesn't persist).
   - Between topics, pause briefly: "Landscape on `<slug>` done. Continuing to `<next-slug>`..." so the user can interrupt.
   - After the last topic, stop. Don't chain further — the user should decide whether to `/rf:deep-research` or pick new topics.

   If the user replies with "stop" / "I'll pick later" / "just the list" or similar, exit cleanly.

## What this skill does NOT do

- No deep research on any single topic — that's `/rf:landscape` and `/rf:deep-research`.
- No judgment on "the real problem to solve" — that's `/rf:synthesize` (later).
- Do not write to `<subject-dir>/topics/<slug>/*` — only to `01-topics.md`.
- Do not modify `00-understanding.md` or `00-open-questions.md`. Discovery is enumeration, not synthesis.

## Stopping rule

When you have 15-30 non-overlapping topics that collectively cover the space as the user framed it, stop. If you're reaching for a 31st, ask whether it's actually a sub-area of something already listed — and if so, note that in the parent's hook rather than as a new row.

## Quality bar

- Hooks must be specific. "Prior authorization is complicated" is useless; "PA denials drive 30% of rheum staff time and AI is being weaponized on both sides" is a hook.
- If you're uncertain about a claim in a hook, add `(?)` — don't invent numbers.
- If a topic is only interesting because you (Claude) find it intellectually interesting but has no clear bearing on what the user framed, drop it.
