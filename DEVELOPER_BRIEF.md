# Developer Brief — automated price refresh and full CI for the cost calculator

**Context for the developer:** a working single-file web app (`index.html`) and the Excel
workbook are already live on GitHub Pages — a non-technical owner published them by upload.
Your job is the industrial version: live weekly price refresh, tests, and CI, without
breaking what already works. The engine already exists in JavaScript inside `index.html`
(and as `engine.js` in the source package) and is fixture-tested against the workbook —
extract it rather than rewriting it.

Paste everything below the line into Claude Code (or another agentic coding tool) in an empty
directory, with `LLM_Hosting_Cost_Calculator.xlsx` present in that directory.

Read the two notes at the very bottom of this file first — there are four decisions you may
want to change before you run it.

---

## MISSION

I have a working Excel workbook, `LLM_Hosting_Cost_Calculator.xlsx`, that estimates the cost of
self-hosting large language models on Google Cloud and Microsoft Azure. It works and its numbers
are tested. I want you to turn it into a public GitHub project with:

1. A **web app** hosted on GitHub Pages — inviting and genuinely helpful to non-technical users.
2. A **weekly automated price refresh** from the official Google and Azure pricing APIs.
3. The **downloadable Excel workbook**, regenerated automatically so it never drifts from the site.

Every feature currently in the workbook must survive the move. Nothing may be quietly dropped.

Work in phases. **Stop at the end of each phase, show me what you built, and wait for my go-ahead.**
Do not run ahead. If any instruction here conflicts with something you discover in the workbook,
tell me rather than guessing.

---

## NON-NEGOTIABLES

Read these before writing any code. They are the difference between a project that stays correct
and one that rots in a month.

**1. One engine, one source of truth.**
The calculation logic must live in exactly one place — a single, pure, dependency-free module
(`src/engine/engine.ts`). The web app calls it. The Excel generator emits formulas derived from it.
Tests exercise it. If the logic ends up written twice, the two copies will disagree within weeks and
the project becomes untrustworthy. This is the most important instruction here.

**2. Prove parity before you change anything.**
Before touching the model, build a golden-fixture test harness. Extract ~40 scenarios from the
existing workbook (inputs plus every output), commit them as `tests/fixtures/golden.json`, and make
the TypeScript engine reproduce all of them to the cent. Only then continue. If a fixture cannot be
matched, stop and tell me — it means I have a bug in the workbook and I want to know.

**3. Never silently overwrite an uncertain price.**
Each machine in the catalogue carries a `confidence` field. Some rows are published list prices;
others are scaled estimates or third-party figures. The weekly job may auto-update `verified` rows.
It must **never** overwrite an `estimated` row without human review. See the automation rules below.

**4. Provenance travels with every number.**
Every price keeps `source`, `confidence`, `last_verified`, and `last_changed`. The UI surfaces this.
A user must always be able to ask "where did this number come from?" and get an answer.

**5. This is a planning tool, not a quote.**
Every page, the README, and the generated workbook carry that disclaimer. Do not imply official
endorsement by Google, Microsoft, NVIDIA, or AMD. Do not use their logos.

---

## PHASE 1 — Understand and extract

1. Read the workbook thoroughly: all 10 tabs, ~658 formulas. `Read_Me` and `Sources` explain intent
   and provenance. Do not skim.
2. Write `scripts/extract_workbook.py` that pulls the catalogue and every assumption out of the
   `GPU_Price_List` and `Rates` tabs into `data/catalog.json` and `data/assumptions.json`.
   **Extract — do not retype.** Hand-copying 31 machines × 19 fields will introduce errors.
3. Generate `tests/fixtures/golden.json` by driving the workbook with LibreOffice headless across a
   scenario grid (small/medium/large/very-large models × each precision × both clouds × each pricing
   model × inference on/off × a few edge cases including one that fits nothing).
4. Port the engine to `src/engine/engine.ts` as pure functions. Make the golden tests pass.
5. Report: fixtures passing, and anything in the workbook you think is wrong.

### The model you are porting

Memory sizing: `required_gb = ceil(params_B × bytes_per_param × (1 + headroom))`, where
bytes_per_param is 2 / 1 / 0.5 for FP16 / FP8-INT8 / INT4.

Throughput (bandwidth-bound decode):
`tokens_per_sec = (Σ GPU bandwidth GB/s ÷ model_weight_GB) × batching_gain × utilisation`

Machine selection — **rank on the cost of meeting the load, not the hourly rate**:
```
machines_needed(row) = peak_tps <= 0 ? 1 : max(1, ceil(peak_tps / tokens_per_sec(row)))
rank(row)            = hourly_rate(row) × machines_needed(row)   // + tiny tie-break favouring fewer machines
```
Filter to rows where `total_vram >= required_gb` and the cloud matches, then take the minimum rank.

Cost stack: compute (`rate × region_uplift × hours × machines`) + storage (`GB × machines × $/GB`)
+ tiered egress (per-cloud, free allowance is a credit **inside** the first tier, not additive)
+ managed-platform uplift (% of compute) + labour (`hours × $/hr`).

Support level is recommended from model size (≤13B Low / ≤70B Medium / >70B High → 20 / 60 / 160
hours per month) and is user-overridable.

### Feature inventory — all of this must exist in the web app

| Workbook tab | Must become |
|---|---|
| `Simple_Calc` | 3-question guided wizard; the landing experience (already built in index.html) |
| `Calculator` | Full form, 17 editable inputs, each with a per-field override |
| `Inference` | Traffic sizing, tokens/sec, machines needed, cost per million tokens |
| `Comparison` | GCP vs Azure side by side on the same requirement |
| `GPU_Price_List` | Browsable, sortable catalogue of all 31 machines with provenance |
| `Sizing_Guide` | Model size → GPU memory needed reference |
| `Rates` | Editable assumptions: storage, egress, region, discounts, support, throughput |
| `Report` | One-page executive summary, printable and exportable to PDF |
| Calculator Step 7 + Rates section K | Pay-per-token catalogue: 21 hosted models (Gemini, GPT, Claude) on both clouds, selectable, compared against self-hosting |
| `Comparison` (extended) | Four-way verdict: self-host and pay-per-token on each cloud |
| `Read_Me` / `Sources` | About page and a sources page with dates and confidence |

Preserve the per-field override behaviour exactly: a guided answer applies until the user sets that
one field, which then wins for that field alone, and clearing it hands the field back. Always show
which value is in force and where it came from.

---

## PHASE 2 — The web app

**Stack:** Vite + React + TypeScript, Tailwind, Recharts, deployed static to GitHub Pages. No
backend, no database, no accounts, no analytics, no cookie banner. State lives in the URL query
string so any estimate is a shareable link.

**The audience is a non-technical manager who has been asked "what would this cost?"** They do not
know what a parameter is. Design for them; let experts drill down.

Requirements:

- **Land on the wizard, not the spreadsheet.** Three questions, plain language, large tappable
  cards rather than dropdowns. A visible answer appears before they finish.
- **Live results.** The number updates as they change anything. No submit button.
- **Progressive disclosure.** "Show me the detail" expands from wizard → full calculator →
  assumptions. Never present all 17 fields at once to a first-time visitor.
- **Explain in place.** Every field has a plain-language hint and an optional "why does this
  matter?" popover. Cost drivers get one-sentence explanations, not jargon.
- **Show the shape of the cost**, not just the total: a stacked bar of compute / storage / egress /
  managed / labour. This is where a user learns that labour is often the biggest line.
- **Sensitivity view.** A small chart of total cost against hours-per-day and against requests-per-
  month. The single most valuable insight this tool offers is that idle time dominates, and a slider
  teaches that faster than a paragraph.
- **Honest warnings, inline and specific.** When the cheapest option spreads a model across many
  small GPUs, say so. When single-user speed drops below ~10 tokens/sec, say so. When the chosen
  machine's price is an estimate, say so and link to the source.
- **Comparison as a real comparison.** GCP and Azure side by side with the delta stated in words.
- **Export:** PDF of the one-page report, the `.xlsx` workbook, and a CSV of the breakdown.
- **Accessibility is a requirement, not a nice-to-have.** WCAG 2.1 AA: keyboard reachable,
  visible focus, 4.5:1 contrast, labelled inputs, no colour-only meaning, respects
  `prefers-reduced-motion`, works at 320px wide and at 200% zoom. Run axe-core in CI and fail
  the build on violations.
- **Fast.** Lighthouse ≥ 95 on performance and accessibility. It is a calculator; it should feel
  instant.

On visual design: aim for calm, credible, and legible — a financial planning tool, not a crypto
dashboard. Restrained palette, one accent colour, generous whitespace, real typographic hierarchy.
Numbers are the hero; make them large and unambiguous. Avoid gradient-heavy AI-startup styling and
avoid default-Bootstrap blandness equally. Show me two directions before committing.

---

## PHASE 3 — Weekly price automation

This is the part most likely to go wrong. Build it defensively.

### Google Cloud

```
GET https://cloudbilling.googleapis.com/v1/services/6F81-5844-456A/skus?key=$GCP_API_KEY
```
Compute Engine's service ID is `6F81-5844-456A`. An API key is required but no IAM permissions are.
Prefer the `v2beta/skus` endpoint — it returns `productTaxonomy.taxonomyCategories` (e.g. `GPUs`,
`GPUs Preemptible`, `L4`) and `geoTaxonomy.regionalMetadata.region`, which is far more robust to
filter on than v1's free-text `description` strings. Cross-check against
`https://cloud.google.com/skus/sku-groups/on-demand-gpus`.

**Watch out:** accelerator-optimised families (A2, A3, A4, G2) are priced as whole machine types,
while N1 + T4 is a machine SKU **plus** a separate accelerator SKU. Handle both. Get this wrong and
T4 rows will be undercounted by the GPU's entire cost.

### Microsoft Azure

```
GET https://prices.azure.com/api/retail/prices?api-version=2023-01-01-preview
    &$filter=serviceName eq 'Virtual Machines'
             and armSkuName eq 'Standard_NC40ads_H100_v5'
             and armRegionName eq 'eastus'
             and priceType eq 'Consumption'
```
Public, unauthenticated. Notes: filter values are **case-sensitive** on this api-version; paginate
via `NextPageLink` (1,000 records max per page); use `isPrimaryMeterRegion eq true`; exclude Windows
meters (match Linux — `productName` without "Windows"); spot rates come from
`contains(meterName, 'Spot')`; and 1-year reserved prices come from `priceType eq 'Reservation'`
with `reservationTerm eq '1 Year'` (note: reservation prices are quoted for the **whole term**, so
divide appropriately to get an hourly figure).

### Three approximations you can now replace with real data

The workbook uses estimates where I had no API. You do not have that excuse:

1. **Region uplifts** (currently ~1.10–1.33 hardcoded multipliers) → query each region directly and
   store real per-region prices.
2. **1-year commitment factors** (currently 0.70 GCP / 0.65 Azure) → real reservation and
   committed-use prices.
3. **Spot factors** (currently 0.33 / 0.19 where unpublished) → real spot meters.

Do these. They remove the workbook's three biggest fudges.

### Validation gate — the job must refuse bad data

```
reject the entire run if:  > 20% of catalogue SKUs fail to resolve
                           (a schema change upstream, not a price change)
flag for human review:     any single price moved > 25% week-on-week
                           any SKU disappeared from the API
                           any price <= 0, null, or NaN
                           any spot price >= its on-demand price
never auto-touch:          rows where confidence != "verified"
                           GPU memory bandwidth (hardware spec, not a price)
                           labour rate, support hours, batching gain, utilisation,
                           prefill advantage (these are user assumptions)
```

### Delivery mechanism

`.github/workflows/weekly-pricing.yml`, cron `0 6 * * MON`, plus `workflow_dispatch`.

The job opens a **pull request**, never commits to `main` directly. The PR body is a human-readable
diff table: SKU, old price, new price, % change, source, and any flags. Clean runs where nothing
exceeded thresholds may auto-merge; anything flagged waits for me. Append every change to
`data/CHANGELOG.md` with the date and source. If the run fails twice consecutively, open an issue.

The site displays "prices last verified: {date}" prominently, and shows a warning banner if that
date is more than 14 days old — a stale calculator that looks fresh is worse than one that admits it.

---

## PHASE 4 — Regenerate the workbook

`scripts/build_xlsx.py` rebuilds the `.xlsx` from `data/*.json`, preserving all 10 tabs, all
formulas, all formatting, drop-downs, and the printable report. It runs in CI on every data change
and attaches the result to a dated GitHub Release, so the download always matches the live site.

Add a CI parity test that drives the generated workbook through LibreOffice headless and asserts it
agrees with `engine.ts` on all golden fixtures. **This test is what prevents the two implementations
from drifting.** It must run on every pull request.

---

## PHASE 5 — Make it a real open-source project

- `README.md`: what it does, a screenshot, the honest limitations, how the weekly update works, how
  to run locally, how to contribute a price correction.
- `METHODOLOGY.md`: the full model, every formula, every assumption, and where each is weak.
  Someone must be able to audit the numbers without reading the code.
- MIT licence. `CONTRIBUTING.md`. Issue templates — one specifically for "this price is wrong",
  asking for a source link.
- CI on every PR: typecheck, unit tests, golden fixtures, xlsx parity, axe-core, Lighthouse budget.
- Branch protection on `main`; require CI green.

### Limitations that must be stated plainly, not buried

- Throughput is a bandwidth model, accurate to roughly a factor of two. Measure before committing.
- The batching-gain assumption swings the answer several-fold; serving software choice matters more
  than GPU choice.
- Sizing is by memory and throughput, not latency under real concurrency.
- No AWS, no neoclouds (CoreWeave, Lambda, RunPod — often materially cheaper), no serverless GPU,
  no TPU/Trainium.
- Excludes load balancers, monitoring, backups, support plans, setup labour, taxes, and negotiated
  discounts.
- List prices only; enterprise agreements are usually cheaper.

---

## ACCEPTANCE CRITERIA

- [ ] All golden fixtures pass against `engine.ts`
- [ ] Generated `.xlsx` agrees with `engine.ts` on all fixtures, enforced in CI
- [ ] All 10 workbook tabs represented in the web app; per-field override behaviour preserved
- [ ] Wizard usable end-to-end by someone who doesn't know what a parameter is
- [ ] Live Pages deployment; estimates shareable by URL
- [ ] Weekly workflow runs, validates, opens a PR with a readable diff, updates the changelog
- [ ] Estimated prices never auto-overwritten
- [ ] Region uplifts, spot rates, and 1-year factors sourced from APIs rather than assumed
- [ ] Lighthouse ≥ 95 performance and accessibility; axe-core clean; works at 320px and 200% zoom
- [ ] "Last verified" date shown; staleness banner after 14 days
- [ ] README and METHODOLOGY complete, with limitations stated up front

## DO NOT

- Do not write the calculation logic twice.
- Do not change any number from the workbook without telling me which and why.
- Do not add a backend, database, accounts, or analytics.
- Do not scrape pricing web pages when an API exists.
- Do not let the weekly job push to `main`.
- Do not present estimated prices with the same confidence as published ones.
- Do not commit API keys. `GCP_API_KEY` goes in repository secrets; Azure needs none.

## START HERE

Confirm you have read the workbook and this brief. Then tell me: your plan for Phase 1, anything in
the workbook that looks wrong to you, and any assumption in this brief you disagree with. Wait for
my reply before writing code.

---
---

## Also automate the hosted-model prices

The pay-per-token catalogue (21 models) has no clean public API on the Google side; Azure's
Retail Prices API covers `serviceName eq 'Foundry Models'`. For Google and Anthropic rates,
scrape is forbidden per the rules above — instead, treat those rows like the estimated GPU
rows: flag them for human review in the weekly PR with a link to the pricing page, and keep
`last_verified` per row. A stale hosted-model price with an honest date beats a fragile scraper.

## Notes for you (not part of the prompt)

**Four decisions you may want to change before pasting:**

1. **Repo visibility.** Written assuming a public repo (GitHub Pages is free for public repos;
   private Pages needs a paid plan). Add the org/repo name if you have one in mind.
2. **Stack.** Vite/React/Tailwind is a safe, well-supported default. If your team standardises on
   something else, say so in Phase 2 — the architecture doesn't depend on it.
3. **Auto-merge.** I've allowed clean price runs to auto-merge. If you'd rather review every change,
   delete that clause.
4. **GCP API key.** You'll need to create one (free, no permissions required) and add it as a repo
   secret. Azure needs nothing.

**One expectation to set:** the GCP side of the automation is genuinely fiddly — SKU descriptions are
inconsistent and GPU pricing splits across machine and accelerator SKUs depending on family. Expect
Phase 3 to take longer than Phase 2. The Azure half should be straightforward.

**Highest-value part of this brief:** the "one engine, one source of truth" rule plus the CI parity
test. Without them you will have a website and a spreadsheet that quietly disagree, which is worse
than having only one of them.
