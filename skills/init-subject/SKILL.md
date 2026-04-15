---
description: Bootstrap a new research subject folder in the current working directory. Creates the scaffold, then runs an ADAPTIVE framing interview — judges which questions and dimensions actually matter for this specific subject and asks only those. Pushes back only where ambiguity would materially change the research.
---

# Init Subject

Create a new research subject folder at `$PWD/$ARGUMENTS/` and populate `00-understanding.md` via an **adaptive interview** — tailored to the subject, not a fixed checklist.

## Inputs

`$ARGUMENTS` = the subject name. Short, lowercase, hyphen-separated. If empty, ask for one before proceeding.

## Preconditions

- If `$PWD/$ARGUMENTS/` exists and is non-empty, **stop**. Don't clobber.
- Templates must exist at `${CLAUDE_PLUGIN_ROOT}/templates/subject/`. Fail loudly if missing.

## Part 1 — Scaffold (silent, mechanical)

1. Create directory structure: `$PWD/$ARGUMENTS/topics/`, `$PWD/$ARGUMENTS/synthesis/`.
2. Read templates from `${CLAUDE_PLUGIN_ROOT}/templates/subject/` — there are **five**: `00-understanding.md`, `00-open-questions.md`, `00-lexicon.md`, `01-topics.md`, `CLAUDE.md`.
3. Substitute `{subject}` → `$ARGUMENTS`, `{date}` → today's date (ISO).
4. Write the five populated template files into `$PWD/$ARGUMENTS/`. The `CLAUDE.md` is auto-loaded by Claude Code when the user is `cd`'d into the subject — it contains the active-session protocol that survives context compaction.

## Part 2 — Subject judgment (silent, internal)

**Before asking the user anything**, classify the subject along these axes. Use your own knowledge — don't web-search for this, it should take 5 seconds of judgment:

- **Domain type**: regulated industry (healthcare, finance, legal, aviation, defense) / commercial market (retail, SaaS, logistics) / theoretical or academic (mathematics, history, philosophy) / technical field (open-source, developer tooling) / something else
- **Geography-sensitive?** Would research meaningfully differ across US/UK/EU/global? Regulated industries usually yes. Developer tooling usually no. Subjects with geography in the name ("Medicare reimbursement", "EU data regulation") have it baked in — don't ask.
- **Market-like?** Does it have buyers, sellers, money flowing, companies competing? Yes → segment and economics matter. No → they're filler questions.
- **Population-varied?** Does the subject have distinct sub-populations with different dynamics? (e.g. adult vs pediatric in medicine). If unclear from the name alone, default to asking.
- **Fast-moving vs stable?** Does timeframe matter? LLM research, crypto regulation → yes. Classical mechanics, medieval history → no.

Based on that, decide:

**Which of the five framing questions to ask:**

| Question | Rule |
|---|---|
| Scope | Always ask — but only include dimensions that matter (geography only if geo-sensitive; population only if pop-varied; timeframe only if fast-moving). |
| Angle | Always ask. The option list should be tailored to the subject (the angles for rheumatology are not the angles for jazz history). |
| End goal | Always ask. |
| Priors | Ask, but frame as genuinely optional — "no, I'm starting fresh" must land as a clean answer. |
| Synthesis criteria | Ask ONLY if End goal involves evaluation/ranking (startup, investment, consulting-into). Skip for pure intellectual understanding or historical/theoretical subjects — there's nothing downstream to rank against. |

**Which Scope dimensions to probe:**

Keep only the ones that would actually change research quality for this subject. If you only need one dimension beyond a general scope statement, just ask that one. Don't fabricate dimensions that don't apply.

## Part 3 — Announce the plan (brief, to the user)

Before starting the interview, tell the user what you've decided:

> "Based on `<subject>`, I'll ask you [N] questions: [list]. I'm skipping [list] because [one-line reason, e.g. 'geography is baked into the subject' or 'this subject is effectively global']. Say `full interview` if you want me to ask everything anyway, or just answer the first question to proceed."

Keep this under 4 lines. The user should be able to scan it and react.

## Part 4 — Adaptive interview

Run only the questions you kept. **One at a time.** Write each answer to the correct section of `$PWD/$ARGUMENTS/00-understanding.md` as you go.

For **each question you ask**, the content should be:
1. A clear question
2. Options/dimensions relevant to *this* subject (not a generic template)
3. A brief example answer in the user's likely domain

**Pushback principle** (applies to every question):

> Before accepting an answer, ask yourself: *is anything in this answer ambiguous in a way that would materially change the research?* If yes, push back on that one thing with 2-3 concrete options and a brief reason each matters. If no, move on.
>
> Do not push back as a ritual. Do not push back on dimensions that don't apply to this subject. Do not push back more than once per question — accept the second answer even if still imperfect, and note the ambiguity in the file.

Below are the five questions with guidance. Tailor each to the subject using your judgment from Part 2.

### Scope

Ask for boundaries, but **only list the dimensions you decided matter** in Part 2. If geography is baked in or doesn't matter, don't mention it. If the subject has no meaningful sub-populations, don't mention population.

Example for a geography-sensitive regulated subject (rheumatology):
> "**Scope.** What are the boundaries?
> - Geography (US, UK, EU, global — these have very different dynamics here)
> - Segment (care delivery / drug development / payer / patient-facing — pick primary)
> - Population (adult vs pediatric, which conditions)
>
> Example: *US adult outpatient rheumatology, inflammatory autoimmune conditions (RA, PsA, axSpA, lupus)*. Yours?"

Example for an effectively-global technical subject (retrieval-augmented generation):
> "**Scope.** What are the boundaries?
> - Focus (research frontier / production systems / evaluation methodology / developer tooling)
> - Any specific model families or domains you care about, or keep it general?
>
> Yours?"

### Angle

Tailor the option list to the subject. Business/operations/regulatory/clinical/technical/economic are the defaults; drop the ones that don't apply and add the ones that do.

### End goal

Standard options (startup / investment thesis / selling into / intellectual understanding / other). **The answer here controls whether Synthesis criteria gets asked** — if intellectual understanding, skip it.

### Priors

> "Any working hypotheses you're bringing in? Even rough. '*No, I'm starting fresh*' is a fine answer."

If yes, capture as bullets tagged `[prior]`. If no, write a single line: *"User started without explicit priors."* Don't push back here — priors are voluntary.

### Synthesis criteria (conditional)

Only if End goal involves ranking/evaluation. Default list: pain level, data availability, implementation difficulty, regulatory/adoption barriers, buyer willingness to pay. Offer end-goal-specific additions (founder → defensibility, founder-market fit; investor → market size, exit horizon). If skipped, delete the `## Synthesis criteria` section from `00-understanding.md` so it's not a stale template header.

## Part 5 — Report and offer continuation

Show the user:
1. A 3-line summary of the framing (e.g. "`<subject>` — US adult outpatient, business-angle, founder-mode")
2. Path to `00-understanding.md`
3. **Offer to continue into `/rf:discover` inline**, so the user doesn't have to `cd` and re-invoke:

   > "Ready for discovery. I can run `/rf:discover` on this subject now (no need to `cd`), or you can do it manually later with `/rf:discover $ARGUMENTS` from this directory or `cd $ARGUMENTS && /rf:discover` from anywhere. Run it now? (yes / no)"

If the user says **yes** (or any clear affirmative):
- Read `${CLAUDE_PLUGIN_ROOT}/skills/discover/SKILL.md`.
- Execute its instructions with `<subject-dir> = $PWD/$ARGUMENTS`. Don't try to `cd` via Bash — shell state doesn't persist between calls. Pass the resolved subject directory explicitly when reading/writing files.

If the user says **no**, just report the next-step hint and exit cleanly.

## Rules

- **Do nothing outside `$PWD/$ARGUMENTS/`.**
- **Don't ask questions that don't apply to this subject.** Adaptive means adaptive.
- **Announce the plan before starting** (Part 3). The user should always be able to override with "full interview".
- **One question at a time. Write answers in real time.**
- **Pushback is conditional, not procedural.** Principle over checklist.
- **Teach while you ask** — the user may not know the dimensions that exist. Options + reasons, not bare questions.
- **Accept "I don't know" after one round of pushback.** Note ambiguity in the file, move on.
- **Don't fabricate dimensions.** If a subject has no meaningful population split, don't invent one to ask about.
