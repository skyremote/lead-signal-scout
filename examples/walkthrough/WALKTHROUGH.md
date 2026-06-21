# Walkthrough — a real run, end to end

Every file in this folder is **real output from an actual run** (Exa discovery +
LLM scoring + outreach), not hand-written. You can reproduce it in a couple of
minutes with your own keys.

The demo leads ([../leads.sample.csv](../leads.sample.csv)) are well-known public
figures, so the run is reproducible and nothing private is exposed. With a real
client list, the names live only in your CSV and never leave your machine except
as search queries — see [Anonymisation](#anonymisation) below.

## 1. Discover + score

```bash
python3 scripts/signal_scout.py run \
  --in examples/leads.sample.csv \
  --out examples/walkthrough/ \
  --provider openai --per-lead 5
```

Produces:

- [`evidence.jsonl`](evidence.jsonl) — the raw public footprint Exa found per lead.
- [`scored_leads.csv`](scored_leads.csv) — every lead, scored 0-100, ranked.
- [`shortlist.csv`](shortlist.csv) — only those above the threshold (60).
- [`brief.md`](brief.md) — the human-readable summary.

What the scores show — the tool **discriminates** (figures from this committed run):

| Score | Name | Tier | Why |
|------:|------|------|-----|
| 100 | Jane Goodall | Strong | Lifelong personal environmental advocacy, in her own words. |
| 100 | Bill McKibben | Strong | Author/activist; his own bylines and podcasts on climate finance. |
| 85 | Christiana Figueres | Strong | Personal climate leadership (her talks, interviews). |
| 0 | Satya Nadella | None | Speaks on climate, but corporate-framed — not personal advocacy. |
| 0 | A Generic Salesperson | None | No evidence attributable to this specific person. |

The bottom two rows are the point. A generic/unmatched lead scores **0** (the
identity-match rule — on-topic content that can't be tied to the person doesn't
count), and a CEO who only speaks on the topic in a corporate frame also scores low
— that's *person-level* signal working as intended.

> **Scores vary run to run.** LLM scoring is non-deterministic, so your numbers
> won't match these exactly (Satya Nadella, a borderline case, scored 70 on an
> earlier run and 0 here). The *ranking* — real advocates high, non-personal low —
> is what's stable. Use a stronger `--model` for tighter consistency.

## 2. Draft warm outreach (Stage 2)

```bash
python3 scripts/signal_scout.py outreach \
  --shortlist examples/walkthrough/shortlist.csv \
  --evidence examples/walkthrough/evidence.jsonl \
  --out examples/walkthrough/ \
  --offer "sustainability rewards and gift cards that turn company spend into measurable environmental impact" \
  --provider openai
```

Produces [`outreach.md`](outreach.md) and [`outreach.csv`](outreach.csv) — a
trigger-led, **sourced** first touch per shortlisted lead (target under 80 words).
Each opens with a specific, sourced fact about that person and bridges to the offer.
You review, personalise the last 10%, and you send — the tool never sends anything.

> Always read these before sending. The drafts cite a source URL; sanity-check
> that the publication name in the copy matches the actual source.

## Anonymisation

These demo leads are public figures, so the brief names them. With a **private**
client list you would mask the names before sharing the deliverable. Below is the
masked view of *this* run's real `brief.md` — the same scores, names redacted:

| Score | Lead | Company | Why |
|------:|------|---------|-----|
| 100 | J. G. | Conservation non-profit | Lifelong personal environmental advocacy, in her own words. |
| 100 | B. M. | Climate org | Own bylines and podcasts on climate finance. |
| 85 | C. F. | Climate advisory | Personal climate leadership (talks, interviews). |
| 0 | S. N. | Enterprise software | Speaks on climate, but corporate-framed — not personal. |

Nothing about a lead leaves your machine except the search query sent to Exa
(name + company) and the evidence snippets sent to your chosen LLM for scoring.
Keep both providers in mind for your own data-handling policy.
