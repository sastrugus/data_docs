# Healthcare Pricing by Douro Data

Douro Data harvests, parses, and quality-gates **hospital-published** machine-readable price files (the CMS hospital price transparency rule, 45 CFR §180.50) and publishes them as clean, query-ready pricing data on the Snowflake Marketplace.

## What makes this data different

Everything here is built from what hospitals themselves publish — not from payer-side Transparency-in-Coverage aggregates. That is a data-class difference, not a styling choice: when you quote this data, you are quoting the hospital's own asserted rates, with contract methodology and care setting first-class on every row. For a provider-side payer renegotiation, that is the right evidentiary basis.

We publish only records that clear a strict quality gate, and where a hospital's file shows a detectable constant-value pattern, we annotate the row rather than altering or deleting it. See [Data Quality Methodology](data-quality.md) for how that works.

## The product family

One product family, three tiers:

| Tier | Status | What you get |
|---|---|---|
| [Hospital Directory — New York](hospital-directory.md) | Live, free | One row per NY hospital whose price file we hold parsed data for: identity, CMS profile, freshness, compliance signal. |
| [Negotiated Rates — New York](negotiated-rates.md) | Live, paid trial | Tens of millions of hospital-published negotiated dollar rates, plus standard charges and compliance findings. |
| Multi-state expansion | Planned | Additional states follow demand during the trial. |

The catalog spans **150+ New York hospitals** across both rate surfaces, and every directory row holds parsed rate data — negotiated rates, standard charges, or both — with zero registration-only rows. The data itself is the source of truth for counts: `COUNT(DISTINCT CMS_CCN)` on any table gives you the current number, and the Marketplace listing carries the precise, date-stamped figures for each release.

## Where to start

- [Hospital Directory — New York (Free)](hospital-directory.md) — the peer-scoping layer
- [Negotiated Rates — New York (Paid Trial)](negotiated-rates.md) — the benchmark
- [Example Queries](example-queries.md) — eight worked examples, smoke-tested live
- [Data Quality Methodology](data-quality.md) — how we gate and annotate
- [Known Limitations](known-limitations.md) — read these here, not discover them later
- [FAQ & Query Conventions](faq.md) — counting, joining, methodology, freshness

Sales and support: [steve@dourodata.com](mailto:steve@dourodata.com)
