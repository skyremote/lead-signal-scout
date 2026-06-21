# Lead Signal Scout

Find which **people** in a leads database personally and publicly care about a
topic — their own LinkedIn posts, articles, talks or podcast appearances — then
score, rank, and shortlist them so your outreach goes to the warm ones first.

Ships configured for **environmental sustainability**, but it works for any
cause or affinity by editing one config file (DEI, a specific technology, open
source, a methodology — anything).

It does person-level signal, not company-level ESG pages. Every score carries the
source URL it's based on, so it's auditable, not a black box.

Built by [NavAIgate](https://navaigate.dev). MIT licensed. **Bring your own API keys.**

---

## What it does

```
leads.csv ──▶ gather (Exa finds each person's public footprint)
          ──▶ score  (an LLM judges 0-100 with the citation)
          ──▶ rank   ──▶ scored_leads.csv + shortlist.csv + brief.md
```

You get a ranked CSV of everyone, a shortlist of the people above your threshold,
and a readable brief. An optional Stage 2 ([outreach-template.md](outreach-template.md))
drafts warm, sourced outreach for the shortlist.

## You need

- **Python 3.9+**
- An **Exa** API key (discovery) — https://exa.ai
- **One** LLM key for scoring — Anthropic (default), OpenAI, or OpenRouter
- *(optional)* a **Firecrawl** key for deep page reads — https://firecrawl.dev

## Setup (5 minutes)

```bash
# 1. Get the code
git clone https://github.com/<your-org>/lead-signal-scout.git
cd lead-signal-scout

# 2. Install the one dependency
python3 -m pip install -r requirements.txt

# 3. Add your keys
cp .env.example .env
#    then open .env and paste your keys in

# 4. Confirm the tool can see them
python3 scripts/signal_scout.py check
```

`check` prints exactly which keys are found and which are missing. When it says
"Ready to run", you're set.

## Run it

```bash
# Test on the first 5 leads of your file (cheap, ~1 minute):
python3 scripts/signal_scout.py run --in your_leads.csv --out out/ --limit 5

# Then the whole list:
python3 scripts/signal_scout.py run --in your_leads.csv --out out/
```

Open `out/shortlist.csv` (or `out/brief.md`) for the warm leads. That's it.

### Your input CSV

Just needs a **name** column. If it also has **company** and a
**linkedin/url** column, those improve accuracy. Headers are detected
case-insensitively. A sample is in [examples/leads.sample.csv](examples/leads.sample.csv).

```csv
name,company,linkedin_url
Jane Smith,Acme Corp,https://www.linkedin.com/in/janesmith
```

## Change the topic

The "signal" is defined in a small JSON file. Sustainability is the default
([config/sustainability.json](config/sustainability.json)); a second example is
[config/diversity-advocacy.example.json](config/diversity-advocacy.example.json).
Copy one, edit `signal_name`, `definition` (the key lever — it teaches the model
what counts as *personal* vs company-level), `keywords` and `threshold`, then:

```bash
python3 scripts/signal_scout.py run --in your_leads.csv --config config/my-signal.json --out out/
```

## Use a different LLM

```bash
# OpenRouter (one key, any model):
python3 scripts/signal_scout.py run --in leads.csv --provider openrouter --out out/

# OpenAI:
python3 scripts/signal_scout.py run --in leads.csv --provider openai --out out/
```

Or set `LLM_PROVIDER` / `LLM_MODEL` in your `.env`. Scoring defaults to a cheap,
fast model; pass `--model` to use a stronger one.

## Useful flags

| Flag | Does |
|------|------|
| `--limit N` | Only process the first N leads (test runs). |
| `--per-lead N` | Exa results per query per lead (default 6; raise for coverage). |
| `--days N` | Only count sources newer than N days. |
| `--scrape` | Firecrawl-read each lead's top page for fuller context (uses credits). |
| `--config FILE` | Use a different signal definition. |

Run `python3 scripts/signal_scout.py <command> --help` for the full list.

## Honest limitations

- **LinkedIn is mostly gated.** This sees *public* posts, reposts, articles,
  press and podcasts — not someone's full private feed. No signal found is not
  proof they don't care.
- **Spot-check the top of your shortlist** before you send. The rubric is strict
  ("cite or it's zero", company-level excluded) but LLMs can still over-credit.
- **Cost scales** with leads × `--per-lead`. Start with `--limit`.

## Also works as an agent skill

This repo is also a drop-in [agent skill](SKILL.md) (Claude Code, Codex, etc.).
Put the folder in your skills directory and your agent can run the whole pipeline
for you and reason over the results.

---

Built by [NavAIgate](https://navaigate.dev) — AI-native consultancy. Questions or
want it tuned to your stack? Get in touch.
