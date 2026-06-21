---
name: lead-signal-scout
description: Use when you have a database or CSV of leads and need to find which INDIVIDUAL people show a genuine personal, public signal on a topic — their own LinkedIn posts/comments, articles, bylines, talks or podcast appearances — not company-level pages. Default topic is environmental sustainability but it is configurable to any cause or affinity. Use for warm-lead prioritisation, person-level intent, signal-based outreach, lead scoring and enrichment. Triggers: "who in my list cares about X", "detect person-level signal", "score leads by personal interest", "find sustainability champions in my DB".
---

# Lead Signal Scout

## Overview

Turn a flat leads list into a ranked shortlist of the people who *personally and
publicly* care about a topic, so outreach goes to the warm ones first. The
engine is topic-agnostic; it ships configured for environmental sustainability.

Pipeline: **gather** (Exa semantic search finds each person's public footprint)
→ **score** (a pluggable LLM judges 0-100 with the citation) → **rank** (ordered
CSV + a shortlist above a threshold + a markdown brief). Optional Stage 2 drafts
warm, sourced outreach for the shortlist.

Core principle: **person-level, not company-level.** A company ESG report is not
a signal. The person's own post, byline, talk or comment is. And **cite or it's
zero** — every score must rest on a named source, never a vibe.

## When to use

- You have a CSV/DB of scraped or exported leads and want to prioritise.
- You want *individual* intent ("does this person care about X"), not firmographic
  or company-level ESG data.
- You want auditable scoring (a URL behind every score), not a black box.
- Works for any signal: sustainability, DEI, a specific technology, open source,
  a methodology — edit the config.

Not for: company-level firmographics (use Clay/Apollo/ZoomInfo), or scraping a
person's full private LinkedIn feed (most of it is gated — see Limitations).

## Setup (one-time)

1. `python3 -m pip install requests` (only dependency).
2. Copy `.env.example` to `.env` and add your keys:
   - `EXA_API_KEY` (required, discovery) — https://exa.ai
   - one LLM key (scoring) — `ANTHROPIC_API_KEY` (default), or `OPENAI_API_KEY`
     / `OPENROUTER_API_KEY` with `LLM_PROVIDER` set accordingly.
   - `FIRECRAWL_API_KEY` (optional, only for `--scrape`) — https://firecrawl.dev
3. `python3 scripts/signal_scout.py check` — confirms keys are found.

## Quick reference

| Goal | Command |
|------|---------|
| Verify keys | `signal_scout.py check` |
| End to end | `signal_scout.py run --in leads.csv --out out/` |
| Test on 5 leads first | `signal_scout.py run --in leads.csv --out out/ --limit 5` |
| Different topic | `... run --in leads.csv --config config/diversity-advocacy.example.json` |
| Just discovery | `signal_scout.py gather --in leads.csv --out out/` |
| Score later | `signal_scout.py score --evidence out/evidence.jsonl --out out/` |
| Use OpenRouter | `... run ... --provider openrouter` |

Run any command with `--help` for all flags (`--per-lead`, `--days`, `--scrape`,
`--limit`, `--model`).

Input CSV needs a **name** column; **company** and a **linkedin/url** column are
used if present (case-insensitive header detection). Outputs land in `out/`:
`scored_leads.csv` (all, ranked), `shortlist.csv` (above threshold), `brief.md`.

## How it works (and how to tune it)

- **Discovery** runs two Exa queries per lead: one natural-language ("posts,
  articles, talks where NAME personally talks about TOPIC") and one tighter
  name-anchored keyword sweep. Raise coverage with `--per-lead 10`; restrict to
  recent activity with `--days 365`.
- **Scoring** sends the evidence pack to the LLM with a strict rubric. Edit the
  signal in a config JSON (`signal_name`, `definition`, `keywords`, `threshold`).
  The `definition` is the most important lever — it teaches the model what
  counts as personal vs company-level.
- **Cheap by default**: scoring uses a small model (haiku / gpt-4o-mini). Override
  with `--model` for tougher judgement.

## Stage 2 (optional): warm outreach

Once you have a shortlist, draft trigger-led, sub-80-word, *sourced* outreach for
each person using their best evidence URL. See [outreach-template.md](outreach-template.md)
for the prompt — paste a shortlist row in and it writes the opener.

## Limitations (be honest with users)

- **LinkedIn is mostly gated.** Exa indexes *public* posts, reposts, articles,
  press and podcasts — not someone's full private feed. Coverage is "public
  footprint", not "everything they ever posted". Absence of signal is not proof.
- **The LLM can over-credit.** The "cite or it's zero" rule and the company-level
  exclusion reduce this, but spot-check the top of the shortlist before sending.
- **Cost scales with leads × per-lead.** Test with `--limit` first.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Empty results for everyone | Check `signal_scout.py check`; verify EXA key works. |
| Everyone scores high | `definition` too loose — tighten what counts as personal. |
| Nobody clears threshold | Lower `threshold` in config, widen `keywords`, raise `--per-lead`. |
| Huge bill | Use `--limit`, keep the cheap default model, skip `--scrape`. |
| Scoring errors per row | No LLM key, or wrong `--provider`/`--model` for your key. |

## Attribution

Built by NavAIgate — https://navaigate.dev. MIT licensed. Bring your own keys.
