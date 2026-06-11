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
| [Negotiated Rates — New York](negotiated-rates.md) | Live, paid trial | 44.8M hospital-published negotiated dollar rates across 132 hospitals, plus standard charges and compliance findings. |
| Multi-state expansion | Planned | Additional states follow demand during the trial. |

Key figures, verified live as of 2026-06-11: the directory covers **154** New York hospitals — **132** with negotiated rates ∪ **136** with standard charges (114 publish both, 18 rates-only, 22 charges-only, zero registration-only rows).

## Where to start

- [Hospital Directory — New York (Free)](hospital-directory.md) — the peer-scoping layer
- [Negotiated Rates — New York (Paid Trial)](negotiated-rates.md) — the benchmark
- [Example Queries](example-queries.md) — eight worked examples, smoke-tested live
- [Data Quality Methodology](data-quality.md) — how we gate and annotate
- [Known Limitations](known-limitations.md) — read these here, not discover them later
- [FAQ & Query Conventions](faq.md) — counting, joining, methodology, freshness

Sales and support: [steve@dourodata.com](mailto:steve@dourodata.com)
