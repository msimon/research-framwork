---
description: Map existing solutions for a problem, find gaps, and sketch differentiation — wedge, unit economics, competitive positioning. Accepts problems from /rf:synthesize ranking OR directly from a deep-dive insight.
---

# Solution

Take one problem and map the solution landscape: who's solving it today, how, where the gaps are, and what a new entrant's wedge could look like.

This is research + strategic sketching, not a business plan. The output should be honest enough to kill a bad idea.

## Inputs

`$ARGUMENTS` = one of:
- A problem slug from `synthesis/problems-ranked.md` (post-synthesize path)
- A topic slug from `01-topics.md` + free-form problem description (direct-from-deep-dive path)
- Empty → resolve interactively (see below)

## Subject-dir resolution

`<subject-dir> = $PWD`. Must contain `00-understanding.md`. If not, stop and tell the user to `cd` into a subject first.

## Input resolution

Parse `$ARGUMENTS` and determine the entry point:

**Entry A — From synthesize.** If `$PWD/synthesis/problems-ranked.md` exists and `$ARGUMENTS` matches a problem slug/title in it → use the ranked problem entry as the source. This is the structured path.

**Entry B — From a deep-dive insight.** If `$ARGUMENTS` is a topic slug (exists in `01-topics.md`) optionally followed by a problem description → pull the problem from the deep-dive research. The topic must have at least `landscape` status. Examples:
- `/rf:solution prior-auth-burden infusion denials that overturn 82% but 90% go unchallenged`
- `/rf:solution prior-auth-burden` (then ask which insight to build on)

**If `$ARGUMENTS` is empty:**
1. Check if `synthesis/problems-ranked.md` exists → show top 3 ranked problems, ask which.
2. If no ranked problems → check `01-topics.md` for topics at `deep-N` status → show their Insights from deep-dive files, ask: "Any of these feel like a problem worth solving? Pick one or describe it."
3. If nothing at deep status either → suggest running `/rf:landscape` or `/rf:deep-research` first.

## Preconditions

- **Entry A:** `synthesis/problems-ranked.md` exists and contains the referenced problem. Source topics have at least `landscape` status.
- **Entry B:** The topic has at least `landscape` status. If only `discovered`, warn: "The research on `<slug>` is thin — landscape it first for a stronger solution analysis."

## What you do

### Phase 1 — Load context

**Entry A (from synthesize):**
1. **Read the problem entry** from `$PWD/synthesis/problems-ranked.md` — who, pain, why it persists, scale, criteria scores, source topics, key uncertainty.
2. **Read `$PWD/00-understanding.md`** — Scope, Angle, End goal, Synthesis criteria.
3. **For each source topic**, read:
   - `landscape.md` — especially Key players, How it works, Economics
   - `deep-*.md` — Session summaries and Insights
   - `sources.md` — for citation continuity
4. **Read `$PWD/00-lexicon.md`** for term consistency.

**Entry B (from deep-dive):**
1. **Read the topic's deep-dive files** (`$PWD/topics/<slug>/deep-*.md`) — Session summaries, Insights, and the specific turns that surfaced the problem. This is the primary source.
2. **Read `$PWD/topics/<slug>/landscape.md`** — Key players, How it works, Economics.
3. **Read `$PWD/00-understanding.md`** — Scope, Angle, End goal.
4. **Read `$PWD/00-lexicon.md`** for term consistency.
5. **Construct a lightweight problem entry** from the deep-dive findings:
   - **Who** has the problem
   - **Pain** — what hurts, quantified if the deep-dive surfaced numbers
   - **Why it persists** — from the deep-dive's analysis
   - **Scale** — from the deep-dive's findings (dollar amounts, denial rates, etc.)
   - **Source topic** — the slug
   - **Key uncertainty** — what the deep-dive flagged but didn't resolve
6. **Write this entry to `$PWD/synthesis/problems-ranked.md`** (create the file if needed, append if it exists). Use the same format as synthesize output so the problem is tracked alongside any future ranked problems. Mark it `_Source: direct from deep-dive, not cross-topic synthesis._`

### Phase 2 — Research the solution landscape

This is the one skill that does **new research** beyond what the upstream files contain. The upstream research was problem-oriented; this is solution-oriented.

Use `WebSearch` and relevant MCPs to find:

- **Existing companies** attacking this problem (startups, incumbents, adjacent players pivoting in). For each: what they do, funding stage, pricing model, traction signals, notable gaps.
- **Adjacent tools** that touch the workflow but don't solve the core problem — potential partners or platforms to build on.
- **Failed attempts** — companies that tried and shut down, or features that got abandoned. Why they failed matters more than that they failed.
- **Regulatory or structural tailwinds/headwinds** that affect solution viability (new rules, policy changes, platform shifts).

Target 6–12 solid sources. Cite everything.

### Phase 3 — Gap analysis

Cross-reference the problem's pain points against what existing solutions actually cover. Identify:

- **Unserved segments** — who has the problem but no one's selling to them?
- **Underserved workflows** — what part of the workflow is still manual, error-prone, or ignored?
- **Pricing gaps** — is the market priced for enterprise but the pain is worst at SMB? Or vice versa?
- **Integration gaps** — does the solution require the user to leave their existing workflow? What's the switching cost?
- **Data gaps** — does solving this well require data that no one has aggregated yet?

### Phase 4 — Sketch differentiation

Based on gaps, draft 2–3 possible wedges. A wedge is the narrowest useful version of the product — the thing you'd ship in 8 weeks to get the first 10 paying users. For each wedge:

- **What it does** (one sentence)
- **Who the first buyer is** (specific — "the billing manager at a 5-rheumatologist practice", not "healthcare providers")
- **Why they'd switch** from current approach (or why they'd start paying for something they do manually today)
- **Rough unit economics** — what you'd charge, what it costs to deliver, where margin comes from. Use numbers from the research where available; flag estimates.
- **Defensibility** — what gets harder to replicate over time? (Data moat, network effect, regulatory lock-in, workflow embeddedness, none)
- **Biggest risk** — the one thing that kills this wedge if it turns out to be wrong

### Phase 5 — Write the solution file

Write `$PWD/synthesis/solutions/<problem-slug>.md`:

```markdown
# Solution landscape: <problem title>
_Generated: <date>_
_Source: <"problems-ranked.md #N" or "deep-dive on <slug>">_

## Problem recap
_3-4 lines from the synthesis — who, what, why, how big._

## Existing players
| Company | What they do | Stage/Funding | Pricing | Gap |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## Adjacent tools
_Tools that touch the workflow but don't solve the core problem. Potential platforms or partners._

## Failed attempts
_Who tried, what happened, why it matters for a new entrant._

## Tailwinds & headwinds
_Regulatory, market, or structural forces that help or hurt._

## Gap analysis
_Where existing solutions fall short — unserved segments, workflow gaps, pricing mismatches, integration friction, data voids._

## Wedge options

### Wedge A: <name>
**What:** <one sentence>
**First buyer:** <specific>
**Why they'd switch:** <reason>
**Unit economics:** <rough numbers>
**Defensibility:** <what compounds>
**Biggest risk:** <what kills it>

### Wedge B: ...

### Wedge C: ...

## Sources
_Numbered, same format as topic sources.md._
```

Create the `$PWD/synthesis/solutions/` directory if it doesn't exist.

### Phase 6 — Update understanding

Append to `$PWD/00-understanding.md` under a `## Solution: <problem>` heading:
- Top 2 wedge options (one line each)
- The main competitive gap
- The biggest risk

### Phase 6b — Commit

`git add <subject-dir>/ && git commit -m "rf: solution <problem-slug>"`

### Phase 7 — Review with the user

Present the solution landscape and wedges, then ask:

> "Three questions before we go further:
>
> 1. **Which wedge feels closest to something you'd actually build?** Not which is biggest — which would you personally ship first?
> 2. **Who would you sell this to first?** Do you have access to that buyer — or would you need a warm intro?
> 3. **What would kill your motivation on this?** Not market risk — personal risk. What would make you stop caring about this problem?
>
> Your answers shape whether this is a real opportunity for *you* or just an interesting market gap."

Record their answers in the solution file under a `## User's assessment` section.

### Phase 8 — Offer continuation

> "Want to turn this into a pitch? `/rf:pitch <problem-slug>` will generate a slide deck — investor or customer-facing — from the research and solution analysis.
>
> Or if another problem from the synthesis is worth exploring: `/rf:solution <other-problem>`."

If they pick pitch:
- Read `${CLAUDE_PLUGIN_ROOT}/skills/pitch/SKILL.md`
- Execute with `<subject-dir>` inherited and `$ARGUMENTS` = `<problem-slug>`

## Quality bar

- **Existing players must be real companies with citations.** Don't invent competitors. If the space is empty, say so — that's a finding.
- **Unit economics are estimates, not forecasts.** Flag every assumption. "Assumes $X/claim based on [source]" is honest. "$X/claim" alone is not.
- **Wedges must be buildable in 8 weeks by a small team.** If a wedge requires a two-year data moat before it's useful, it's not a wedge — it's a vision. Separate the two.
- **Failed attempts are gold.** A company that raised $10M and shut down teaches more than five that raised $2M and are still alive. Chase the failures.
- **Kill bad ideas.** If the gap analysis shows the problem is well-served, say so. "There's no wedge here" is a valid and valuable conclusion. Don't force-fit a solution.

## What this skill does NOT do

- Does not rank problems — that's `/rf:synthesize`.
- Does not generate pitch decks — that's `/rf:pitch`.
- Does not open a session. This is autonomous write-and-review, like `/rf:landscape`.
