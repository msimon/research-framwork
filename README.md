# rf — research framework

A Claude Code plugin for breadth-first domain research with conversational deep-dives. Built to understand a space well enough to find real problems worth solving.

## Workflow

```
/rf:init-subject <name>          # scaffold a new subject folder
/rf:discover                     # enumerate 15-30 research areas
/rf:landscape <slug>             # substantive overview of one area
/rf:deep-research <slug> [q?]    # conversational deep-dive (session-based)
/rf:resume                       # re-enter an active deep-research session after context loss
# coming later:
/rf:synthesize                   # specialist agents rank problems worth solving
/rf:solution <problem>           # sketch existing solutions, differentiation, unit econ
```

## Install

Clone anywhere and launch Claude Code with `--plugin-dir`:

```bash
git clone <this-repo> ~/code/rf

# then, from whatever directory you want to store research subjects in:
cd ~/research/
claude --plugin-dir ~/code/rf
```

Skills appear namespaced as `/rf:<name>`. Run `/help` in Claude Code to see them.

Iterating on the plugin itself? After editing any skill or template, run `/reload-plugins` — no restart needed.

## Bundled MCP servers

The plugin auto-starts four MCP servers when enabled (via `.mcp.json` at the plugin root):

| Server | Package | Purpose |
|---|---|---|
| `pubmed` | `@cyanheads/pubmed-mcp-server` (npm) | PubMed literature, MeSH, citations |
| `clinicaltrials` | `clinicaltrialsgov-mcp-server` (npm) | ClinicalTrials.gov v2 — trials, eligibility, results |
| `openfda` | `@uh-joan/fda-mcp-server` (npm) | FDA drug approvals, labels, adverse events, recalls |
| `sec-edgar` | `sec-edgar-mcp` (pypi, via `uvx`) | 10-K/10-Q filings, structured financials |

**Requirements on your machine:** `node`/`npx` (for the three npm servers), `uv`/`uvx` (for the SEC server).

**OpenFDA needs an API key.** Free to get at <https://open.fda.gov/apis/authentication/>. Set `OPENFDA_API_KEY` in your shell env and it gets passed through.

**Disabling a server you don't want running:** comment it out of `.mcp.json` locally and `/reload-plugins`. If a package name changes upstream, edit `.mcp.json` to match.

## Typical session

```
cd ~/research/
claude --plugin-dir ~/code/rf

# in Claude Code:
/rf:init-subject rheumatology
# (answer the framing question it asks you)

/rf:discover
# — produces 15-30 candidate topics in 01-topics.md

/rf:landscape prior-auth
# — reads, researches, writes topics/prior-auth/landscape.md,
#   appends to 00-understanding.md, asks you 3 questions

/rf:deep-research prior-auth how does appeal workflow actually work at a mid-size rheum practice
# — opens a SESSION. Every subsequent message is part of this dive until
#   you invoke another /rf: skill or say "we're done".
```

## File layout of a subject

```
<subject>/
  CLAUDE.md                 # auto-loaded instructions — active-session protocol
  00-understanding.md       # YOUR mental model — edit freely, skills append suggestions
  00-lexicon.md             # running glossary — grep here for term definitions
  00-open-questions.md      # contradictions, unknowns, parked threads
  01-topics.md              # topic index with status: discovered → landscape → deep-N
  .rf-session.json          # present only while a /rf:deep-research session is OPEN
  topics/<slug>/
    landscape.md            # /rf:landscape output
    deep-<slug>-DATE.md     # one file per /rf:deep-research session
    sources.md              # accumulated citations across the topic
  synthesis/                # populated later by /rf:synthesize and /rf:solution
```

## Surviving context compaction

Long deep-research sessions can exceed Claude Code's context window and trigger compaction, which drops the in-conversation session state. The plugin handles this with three redundancies:

1. **`CLAUDE.md` inside each subject folder** is auto-loaded when you `cd` into the subject — it re-injects the session protocol on every turn. **For long sessions, `cd <subject>` before launching Claude Code.**
2. **`.rf-session.json`** is the on-disk marker of "a session is open." It holds the topic, deep-dive file path, turn count, and timestamps. Present iff a session is active.
3. **`/rf:resume`** reads `.rf-session.json` + the deep-dive file and re-establishes context. Run it any time you feel the session has drifted or after a compaction event.

## Design principles

- **Research ≠ understanding.** `00-understanding.md` is for beliefs; topic files are raw research. Skills propose additions; you decide what becomes belief.
- **Breadth before depth.** `/rf:discover` surfaces everything shallowly. `/rf:landscape` is the workhorse. `/rf:deep-research` is reserved for things that genuinely matter.
- **Conversation over dumps.** Deep-dives are sessions, not report-generation. Each turn sharpens one thread.
- **Explicit stopping rules.** Deep-dives ask "can you explain this in 3 min? know the 5 players? know the unit economics?" — otherwise they recurse forever.
- **Contradictions are signal.** `00-open-questions.md` captures them rather than hiding them in prose.

## Sharing with a teammate

They clone the repo, install `node`/`uv`, (optionally) set `OPENFDA_API_KEY`, and launch Claude Code with `--plugin-dir`. No marketplace needed for private sharing.

## Status

`v0.1.0` — scaffold, `/rf:init-subject`, `/rf:discover`, `/rf:landscape`, `/rf:deep-research` shipped. `/rf:synthesize` and `/rf:solution` (with specialist sub-agents) to follow once real research has validated the upstream skills.
