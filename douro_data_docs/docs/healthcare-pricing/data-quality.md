# Data Quality Methodology

This page is where Melange differs most from the rest of the market, so it gets real depth.

## The pipeline

Every hospital in the catalog goes through the same four stages:

1. **Fetch** — we harvest the hospital's machine-readable price file from its published location.
2. **Parse** — the file is parsed faithfully. The parser transcribes what the hospital published; it does not infer, repair, or normalize values.
3. **Quality gate** — records must clear a strict quality gate before publication, and detectable constant-value patterns are annotated at the row level.
4. **Publish** — gated, annotated data lands in the Marketplace tables, stamped with `LATEST_INGEST_DATE` (our load date).

## The never-suppress principle

We never alter or delete a published value. If a hospital's file contains a $0.01 flood or a repeated magic number, those rows ship — flagged. This matters because deletion destroys evidence: you can't audit what isn't there, and a dataset that silently drops inconvenient rows can't tell you what its publisher decided you shouldn't see. We annotate, you choose.

## AUDIT_FLAG: two grades, precisely

`AUDIT_FLAG` is a nullable column on `NEGOTIATED_RATES` with exactly two values. The two grades are different kinds of statements, and the difference is the point:

**`CONSTANT_VALUE_FILL` — an evidence-backed verdict.** The value is a source-side fill, not a real per-service price: the hospital's file repeats a constant where distinct per-service prices should be. About 2.3% of New York rows carry this flag (as of 2026-06-11).

**`CONSTANT_RATE_PLATEAU` — a neutral structural observation.** One constant value appears across many codes of a payer column. That structure is consistent with real contract arrangements — blanket rates, case rates, tiered exports — so these rows are real data: annotated, never excluded. About 13% of rows carry this flag (as of 2026-06-11). It is a structural fact you should know before per-code benchmarking, not a defect.

**NULL — no pattern detected.** This is not a certification. It means our detector found no constant-value pattern on that row, nothing more.

## The two filter recipes

Both recipes are documented because the right one depends on your question. The buyer chooses; we don't choose for you.

**Strict per-code benchmarking** — excludes both verdict and observation rows. The right default for per-code medians and percentiles, because a plateau column is per-code degenerate even when its constant is a real price:

```sql
WHERE AUDIT_FLAG IS NULL
```

**Fills-only exclusion** — excludes only verdict-grade fills and keeps plateau rows. The right choice when measuring a payer's overall posture or episode/blanket arrangements, where a real global case rate is signal, not noise:

```sql
WHERE AUDIT_FLAG IS DISTINCT FROM 'CONSTANT_VALUE_FILL'
```

## Evidence on request

Behind every flag is a row-level evidence payload — the detected pattern, its extent, and its verification status. That detail lives off-share and is available on request: [steve@dourodata.com](mailto:steve@dourodata.com).
