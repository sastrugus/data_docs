# Negotiated Rates — New York (Paid Trial)

The benchmark dataset: tens of millions of hospital-published negotiated dollar rates spanning 150+ New York hospitals across both rate surfaces, keyed to each hospital's CMS CCN, methodology- and setting-aware. For live counts, run `COUNT(DISTINCT CMS_CCN)` on the table itself; the Marketplace listing carries the precise, date-stamped figures for each release.

This dataset benchmarks payer-specific negotiated dollar rates; percent-of-charges contracts surface as a compliance disclosure signal, not a benchmarkable rate — negotiated dollar amounts are unaffected.

## NEGOTIATED_RATES

Each row in `MARKETPLACE.NEGOTIATED_RATES` carries:

| Column | Meaning |
|---|---|
| `CMS_CCN`, `HOSPITAL_NAME` | The hospital, CCN-keyed for joins. |
| `CODE`, `CODE_TYPE`, `DESCRIPTION` | The procedure code (CPT, HCPCS, MS-DRG, APR-DRG, APC, NDC, …) and its description. |
| `SETTING` | Care setting (inpatient / outpatient). |
| `NEGOTIATED_RATE` | The negotiated dollar amount, as published by the hospital. |
| `NEGOTIATED_TYPE` | Contract methodology as the hospital published it: case rate, per diem, fee schedule, … NULL means the source published no methodology cell. |
| `RATE_TYPE` | Rate basis — filter on `'negotiated_dollar'` for dollar benchmarking. |
| `PAYER_NAME`, `PLAN_NAME` | The payer and plan exactly as the hospital reported them. Present on every row. |
| `PAYER_CANONICAL`, `PAYER_LINE_OF_BUSINESS` | Canonical parent payer and line-of-business tag (commercial / medicare_adv / medicaid / behavioral), where mapped. |
| `AUDIT_FLAG` | Additive data-quality annotation — see below. |

## Payer canonicalization

We ship a canonical payer key plus a line-of-business tag that collapses hospital-reported payer-name variants for New York's major payers into a single parent entity — so a commercial Aetna cross-hospital spread doesn't quietly absorb Aetna Medicaid or Medicare Advantage.

About two-thirds of negotiated-dollar rows are mapped to a canonical parent payer; the remainder retain their hospital-reported `PAYER_NAME` — present on every row — so the long tail of smaller and out-of-state payers is always matchable. Nothing is dropped: every row is either mapped to a parent or carries its source payer name. We extend the canonical map as we add hospitals.

## AUDIT_FLAG

Every row carries an additive `AUDIT_FLAG` column: we never alter or remove a published value, but where a hospital's file shows a detectable constant-value pattern, we say so on the row. It has exactly two values — `CONSTANT_VALUE_FILL` (an evidence-backed verdict that the value is a source-side fill) and `CONSTANT_RATE_PLATEAU` (a neutral structural observation, consistent with real blanket, case-rate, or tiered-export arrangements) — or NULL, meaning no pattern was detected. You choose the filter; we don't choose for you.

The full definitions and both filter recipes are on the [Data Quality Methodology](data-quality.md) page.

## STANDARD_CHARGES

`MARKETPLACE.STANDARD_CHARGES` carries the four CMS-required standard charges per service item: gross, discounted cash, and the de-identified minimum and maximum. Joined to negotiated rates, it answers questions like "where does the negotiated rate sit relative to the cash price?"

## COMPLIANCE_VIOLATIONS

`MARKETPLACE.COMPLIANCE_VIOLATIONS` carries row-level findings from our CMS-transparency checker where it has run: violation code, severity, the field path in the hospital's file, the CMS regulation section, and resolution status.

Read it carefully: an absent hospital means "not yet examined," never "clean." `STANDARD_CHARGES.HAS_COMPLIANCE_CHECK = FALSE` is the explicit "not yet examined" flag, and the absence of recorded violations is not a certification of compliance.

## Worked examples

Eight hero queries — single-hospital lookup through market percentiles, cash spreads, compliance, and freshness — are on the [Example Queries](example-queries.md) page, smoke-tested live on 2026-06-11.
