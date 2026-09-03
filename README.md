# LLM Hosting Cost Calculator

**What would it cost to run an AI model?** This calculator answers that in plain
language: rent GPU machines on Google Cloud or Microsoft Azure, or pay per token
for a hosted model (Gemini, GPT, Claude) — including the people needed to look
after it.

**Use it here:** open `index.html`, or visit the GitHub Pages address if this
repository has Pages switched on.

## What it covers

- 31 GPU machines across both clouds, from a $0.53/hour T4 to a $110/hour H200 node
- 21 hosted pay-per-token models from Google, OpenAI and Anthropic
- Machine sizing by memory **and** throughput — not just "does it fit" but "can it keep up"
- Support and labour cost, which is the largest line item for small deployments
- A four-way comparison: self-host or pay per token, on either cloud

The site is backed by a full **Excel workbook** — ten tabs including a printable
executive report, a sizing guide, and the source for every price. The website and
workbook were built from the same data and tested to agree. The workbook is not
hosted here: it is available on request from **Mark Ibrahim** via the notes section
on the site.

## Honesty notes

- All prices are **US-region list prices, verified August 2026**. No negotiated
  discounts, taxes, or promotional rates. The site shows a staleness warning
  automatically once the data is more than ~6 weeks old.
- Throughput uses a memory-bandwidth model — accurate to roughly a factor of two.
  Measure before you commit real capacity.
- Some multi-GPU Azure rates are scaled from published single-GPU rates and are
  labelled as estimates.
- This is an independent planning tool, not affiliated with or endorsed by Google,
  Microsoft, NVIDIA, AMD, OpenAI or Anthropic. It is a planning estimate, not a quote.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole calculator — one self-contained file |
| `HOW_TO_PUBLISH.md` | Click-by-click publishing instructions, no technical knowledge needed |
| `DEVELOPER_BRIEF.md` | For a developer later: automated weekly price refresh, tests, CI |

## Credit

Designed and developed by **Mark Ibrahim**.

## License

MIT — use it, copy it, adapt it. No warranty; see the honesty notes above.
