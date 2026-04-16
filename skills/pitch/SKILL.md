---
description: Generate a pitch deck for a solved problem — investor or customer-facing. Two-step flow: markdown draft with user review, then Google Slides generation. Run after /rf:solution.
---

# Pitch

Turn a `/rf:solution` output into a pitch deck. Two audiences, two structures. Two-step flow: draft in markdown (user reviews and iterates), then generate real Google Slides.

## Inputs

`$ARGUMENTS` format: `<problem-slug> [investor|customer]`

- `problem-slug` must match a file in `$PWD/synthesis/solutions/<slug>.md`. If missing, list available solutions and ask.
- Audience is `investor` or `customer`. If omitted, ask: "Who's this deck for — **investor** (fundraising, market-size story, defensibility) or **customer** (buyer, pain-first, ROI, implementation)?"

## Subject-dir resolution

`<subject-dir> = $PWD`. Must contain `synthesis/solutions/`. If missing, tell the user to run `/rf:solution` first.

## Preconditions

- `$PWD/synthesis/solutions/<slug>.md` exists.
- `$PWD/00-understanding.md` exists.
- `$PWD/synthesis/problems-ranked.md` exists (for cross-problem context).

---

## Step 1 — Generate the markdown deck

### Load context

1. **Read the solution file** `$PWD/synthesis/solutions/<slug>.md` — full. This is the primary source: problem recap, existing players, gaps, wedges, user's assessment.
2. **Read `$PWD/00-understanding.md`** — Scope, Angle, End goal, Key beliefs.
3. **Read `$PWD/synthesis/problems-ranked.md`** — the entry for this problem (criteria scores, scale).
4. **Read source topic files** referenced in the solution — landscape.md `## Economics` and `## Key players` sections for hard numbers and competitive context.
5. **Read `$PWD/00-lexicon.md`** — so slide copy uses terms consistently and defines them for the audience.

### Choose the deck structure

Design the slide sequence based on the audience, the subject, and the strength of the research. There is no fixed template — the structure should follow from what the material actually supports and what the audience needs to hear.

**Guiding principles by audience:**

- **Investor deck:** Lead with the problem and market. The audience needs to believe the pain is real, the market is big enough, and you have a credible wedge. End with an ask. Typical building blocks: problem, market size, status quo workflow, solution, wedge/GTM, business model, competition, defensibility, team, ask — but drop or merge blocks based on what the research supports. If you don't have traction data, don't include a traction slide with filler.
- **Customer deck:** Lead with their pain, not your product. The audience needs to feel the cost of inaction, see how you fit into their workflow, and trust the ROI. End with next steps. Typical building blocks: pain, cost of status quo, what's changing, what you do, how it works, results/ROI, implementation, social proof, pricing, next steps — but adapt to the buyer. A billing manager cares about different things than a practice administrator.

**Before writing slides**, propose the sequence to yourself: does each slide earn its place? Does the order tell a story? Would cutting any slide lose something the audience needs? 8 strong slides > 13 with padding.

**Examples of adaptation:**
- Regulated industry + investor → you might need a dedicated regulatory-tailwind slide (a new CMS rule, a compliance deadline) because that's the "why now"
- Technical product + customer → "how it works" might need two slides (integration + workflow) because switching cost is the real objection
- Pre-product + investor → skip the product walkthrough, expand on wedge and first-10-customers narrative
- Crowded market + either audience → competition matrix deserves a full slide; empty market → fold competition into one bullet on the solution slide

### Write the markdown deck

Write `$PWD/synthesis/pitches/<slug>-<audience>.md`:

```markdown
# Pitch: <problem title> — <audience> deck
_Generated: <date>_
_Source: synthesis/solutions/<slug>.md_
_Audience: <investor|customer>_

---

<!-- Slide 1 -->
## <slide title>

<slide content — concise, one idea per slide>

**Speaker notes:** <what to say when presenting this slide — context, transitions, anticipated questions>

---

<!-- Slide 2 -->
## <slide title>

...
```

**Slide content rules:**
- **One idea per slide.** If a slide has two points, split it.
- **Numbers over narratives.** "35% of claims denied" beats "many claims get denied." Pull numbers from the research — cite the source topic in speaker notes, not on the slide.
- **Define terms for the audience.** Investor decks: assume smart-but-not-domain-expert (spell out abbreviations on first use, skip deep jargon). Customer decks: assume they know the domain (use their language, don't explain PA to a billing manager).
- **Speaker notes are mandatory.** Every slide gets notes: what to emphasize, what question this slide preempts, how to transition to the next slide.
- **Competition slide uses a matrix**, not a feature list. Rows = capabilities that matter to the buyer. Columns = you vs 2-3 competitors. Use checkmarks/dashes, not paragraphs.
- **Team slide (investor only) is a placeholder.** Write: "*(Fill in: why your background makes you the right person to solve this. What unfair advantage do you bring — domain access, technical skill, personal experience with the problem?)*"
- **No filler slides.** If you don't have material for a slide, drop it and note why in a comment. 8 strong slides > 13 with padding.

Create `$PWD/synthesis/pitches/` directory if it doesn't exist.

### Fact-check pass

Before showing the deck to the user, audit every factual claim in the slides:

1. **Extract claims.** Walk each slide and list every specific number, statistic, company name, market size, mechanism description, or causal statement.

2. **Trace to source.** For each claim, find where it came from in the research files — deep-dive turns, landscape, solution file, sources.md. If a claim doesn't trace back to any file, flag it as **unsourced**.

3. **Verify against originals.** For claims that are load-bearing (problem slide numbers, market size, ROI figures, competition positioning), go back to the original external source (via `WebSearch` or the URL in sources.md) and confirm the number still holds. Research can be weeks or months old — numbers get revised, companies pivot, studies get retracted.

4. **Fix or flag.** For each claim:
   - **Verified** — number matches source. Add the source reference to speaker notes if not already there.
   - **Stale or wrong** — source says something different now. Update the slide and speaker notes with the current number.
   - **Unsourced** — you wrote it but can't trace it. Either find a source, reframe as a qualified estimate (and say so on the slide), or remove it.
   - **Rounded or derived** — you simplified a range or computed a number from two sources. Note the derivation in speaker notes so the user can defend it if challenged.

5. **Append a fact-check summary** to the bottom of the pitch markdown:
   ```markdown
   ## Fact-check log
   | Slide | Claim | Source | Status |
   |---|---|---|---|
   | 2 | "35% denial rate for rheum biologics" | deep-dive turn 1 → Counterforce | Verified |
   | 3 | "$4.2B market" | solution file → derived from practice count × burden | Derived — show math in notes |
   | ... | ... | ... | ... |
   ```
   This log is for the user's confidence, not for the audience — it doesn't go into Google Slides.

**Commit:** `git add <subject-dir>/ && git commit -m "rf: pitch draft <problem-slug> — <audience>"`

### Present to the user for review

> "Draft deck written to `synthesis/pitches/<slug>-<audience>.md` — <N> slides.
>
> **Slide overview:**
> 1. <title> — <one-line summary>
> 2. ...
>
> Review the file and tell me what to change. Common feedback:
> - "Slide 3 is too dense" → I'll split it
> - "Add a slide on X" → I'll insert it
> - "The framing on slide 2 is wrong — it's more about Y" → I'll rewrite
> - "Looks good" → I'll generate Google Slides
>
> Edit the markdown directly if you prefer — I'll read your changes before generating slides."

### Iterate until approved

- Accept feedback, edit the markdown in place.
- If the user edits the file directly, read it before the next action.
- Don't rush to slide generation. The markdown is the thinking — the slides are just rendering.
- **One round of unprompted suggestions max.** After writing the draft, if you see an obvious structural issue (e.g. no clear ask slide, competition slide missing a major player from the research), flag it once. After that, follow their lead.

The user signals approval by saying "looks good", "generate", "let's do slides", or similar.

---

## Step 2 — Generate Google Slides

### Check for Google Slides MCP

Look for `google-slides` tools (e.g. `create_presentation`, `batch_update_presentation`). You can detect this by trying to use them — if they're not available, you'll know.

**If Google Slides tools ARE available → generate slides:**

1. **Parse the markdown deck** into a slide-by-slide structure: title, content blocks, speaker notes.

2. **Call `create_presentation`** with the deck title.

3. **For each slide, call `batch_update_presentation`** to:
   - Create a new slide
   - Add title text box (large, top)
   - Add body content (bullet points, text blocks)
   - Add speaker notes
   - For the competition matrix: create a table

4. **Apply minimal formatting:**
   - Clean sans-serif font (the user will style it themselves)
   - Title text larger than body text
   - Consistent margins
   - Don't over-design — the user knows Google Slides and will customize

5. **Report the result:**
   > "Google Slides deck created: <URL>
   >
   > <N> slides generated. Speaker notes included. The deck is intentionally unstyled — add your branding, adjust layout, drop in screenshots/diagrams as needed.
   >
   > The markdown source of truth is at `synthesis/pitches/<slug>-<audience>.md` — if you regenerate later, your Google Slides edits won't carry back, so keep major structural changes in the markdown."

**If Google Slides tools are NOT available → run setup flow:**

> "Google Slides MCP isn't configured yet. I've generated the markdown deck — you can present from that or set up Google Slides generation (one-time, ~10 minutes).
>
> **Quick setup:**
>
> 1. Create a GCP project at [console.cloud.google.com](https://console.cloud.google.com)
> 2. Enable the **Google Slides API**
> 3. Create an **OAuth client ID** (Desktop app type)
> 4. Clone and build the MCP server:
>    ```
>    git clone https://github.com/matteoantoci/google-slides-mcp.git ~/code/google-slides-mcp
>    cd ~/code/google-slides-mcp && npm install && npm run build
>    ```
> 5. Get your refresh token (run this in your terminal — it opens a browser):
>    ```
>    ! cd ~/code/google-slides-mcp && GOOGLE_CLIENT_ID='<your-id>' GOOGLE_CLIENT_SECRET='<your-secret>' npm run get-token
>    ```
> 6. Add these to your shell profile:
>    ```
>    export GOOGLE_SLIDES_CLIENT_ID='<your-id>'
>    export GOOGLE_SLIDES_CLIENT_SECRET='<your-secret>'
>    export GOOGLE_SLIDES_REFRESH_TOKEN='<your-token>'
>    ```
> 7. I'll add the server to the plugin's `.mcp.json`. Run `/reload-plugins` and then `/rf:pitch <slug> <audience>` again.
>
> Want me to walk you through it step by step, or skip slides for now?"

If they choose to proceed with setup:
- Walk them through each step interactively.
- After step 5 (they have the three env vars), add to the plugin's `.mcp.json`:
  ```json
  "google-slides": {
    "command": "node",
    "args": ["<absolute-path-to>/google-slides-mcp/dist/index.js"],
    "env": {
      "GOOGLE_CLIENT_ID": "${GOOGLE_SLIDES_CLIENT_ID}",
      "GOOGLE_CLIENT_SECRET": "${GOOGLE_SLIDES_CLIENT_SECRET}",
      "GOOGLE_REFRESH_TOKEN": "${GOOGLE_SLIDES_REFRESH_TOKEN}"
    }
  }
  ```
- Tell them to `/reload-plugins`, then re-run step 2 of this skill.

If they skip: exit cleanly. The markdown deck is complete and usable on its own.

---

## Quality bar

- **Every number on a slide must trace back to the research.** Speaker notes cite the source topic. No invented statistics.
- **The problem slide is the most important slide.** If the audience doesn't feel the pain, nothing after it matters. Spend the most effort here.
- **Investor decks sell the market; customer decks sell the pain.** Don't mix framings.
- **Competition matrices must be honest.** If a competitor does something better than you, show it. Credibility > cheerleading.
- **Speaker notes should prepare for hard questions.** "They'll ask about X here — your answer is Y because [source]."
- **10 strong slides > 15 with padding.** Drop slides that don't earn their place.

## What this skill does NOT do

- Does not do new research. Works from existing solution + synthesis files. If the research is thin, the deck will be thin — go back to `/rf:deep-research` or `/rf:solution`.
- Does not design visual layouts, custom themes, or add images. It creates structured content slides. The user styles them in Google Slides.
- Does not replace the user's judgment on messaging. The draft is a starting point — the review loop is where the real deck gets made.
