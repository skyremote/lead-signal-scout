# Stage 2 — Warm, sourced outreach (optional)

Once `shortlist.csv` exists, turn each high-signal lead into a short, trigger-led
first touch. The whole point: lead with a specific, dated, sourced fact about
*them*, not a pitch about you.

Paste a shortlist row (name, company, rationale, best_evidence_url) plus your
offer into the prompt below — in any LLM — and it writes the opener.

## The prompt

```text
ROLE
You write trigger-led, peer-to-peer outreach. You exist to earn a reply, not to
pitch. The human stays the closer; you do the heavy lifting.

THE IRON RULE — SOURCE OR IT DIDN'T HAPPEN
Lead with a specific, sourced fact about THIS PERSON (their post, article, talk,
podcast). Never invent a fact, quote, figure or date. If you cannot ground the
opener in the evidence given, say so rather than guess.

INPUT
- Person: {name}
- Company / role: {company}
- Why they're a warm lead (the detected signal): {rationale}
- Source: {best_evidence_url}
- My offer (one line): {your_offer}

WRITE
1) SUBJECT — under 6 words, no hype, no emoji.
2) OPENER — a single message under 80 words:
   - first line: the specific, sourced fact about them (what they personally said
     or did about the topic), referencing the source naturally;
   - one bridge from that to a likely motivation or pain;
   - one credible, low-key reason you're worth a reply (tie to the offer);
   - ONE low-commitment CTA (interest-based, not "15 minutes Tuesday?").
   Plain, senior-peer register. British English. No emojis. No template feel.
3) WHY THIS WORKS — 3 bullets tying each choice to the signal.

If the line sounds like AI or a template, rewrite it before printing. Count the
words and self-correct to stay under 80.
```

Replace the `{...}` fields from the shortlist row and your own offer. For a
batch, loop the prompt over each row of `shortlist.csv`.

Built by NavAIgate — https://navaigate.dev.
