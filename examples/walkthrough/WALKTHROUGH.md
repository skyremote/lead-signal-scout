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

What the scores show — the tool **discriminates**:

| Score | Name | Tier | Why |
|------:|------|------|-----|
| 100 | Jane Goodall | Strong | Lifelong personal environmental advocacy, in her own words. |
| 100 | Bill McKibben | Strong | Author/activist; his own bylines and podcasts on climate finance. |
| 100 | Christiana Figueres | Strong | Personal climate leadership (her TED talk, interviews). |
| 70 | Satya Nadella | Medium | Discusses it publicly, but mostly corporate-framed. |
| 0 | A Generic Salesperson | None | No evidence attributable to this specific person. |

The last row is the important one: a generic/unmatched lead correctly scores **0**
because of the identity-match rule. On-topic content that can't be tied to the
actual person does not count.

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
trigger-led, sub-80-word, **sourced** first touch per shortlisted lead. Each opens
with a specific fact about that person (their Earth Day message, their TED talk,
their New Yorker piece) and bridges to the offer. You review, personalise the last
10%, and you send — the tool never sends anything.

> Always read these before sending. The drafts cite a source URL; sanity-check
> that the publication name in the copy matches the actual source.

## Anonymisation

These demo leads are public figures, so the brief names them. For a **private**
client list, the same `brief.md` reads like this — the deliverable can be shared
without exposing the list:

| Score | Lead | Company | Why |
|------:|------|---------|-----|
| 92 | J. G. | Climate non-profit | Strong personal advocacy in own posts/talks. |
| 88 | C. F. | Advisory firm | Public climate leadership, own interviews. |
| 71 | S. N. | Enterprise software | Engaged but corporate-framed. |
| 0 | (lead 41) | Industrial supplier | No personal signal attributable. |

Nothing about a lead leaves your machine except the search query sent to Exa
(name + company) and the evidence snippets sent to your chosen LLM for scoring.
Keep both providers in mind for your own data-handling policy.
