# FAQ & Query Conventions

## How do I count hospitals?

With `COUNT(DISTINCT CMS_CCN)`, never with row counts. Some facilities share one published source file but are distinct CMS-registered hospitals, so counting rows or file sources will miscount. The CCN is the unit of "a hospital" everywhere in this product family.

```sql
SELECT COUNT(DISTINCT CMS_CCN) AS hospitals
FROM MELANGE.MARKETPLACE.NEGOTIATED_RATES
WHERE STATE_CODE = 'NY';
```

## How do I join the tables?

On `CMS_CCN` + `CODE` + `CODE_TYPE` + setting. Compare `SETTING` through `LOWER()` as a casing guard:

```sql
FROM MELANGE.MARKETPLACE.NEGOTIATED_RATES n
JOIN MELANGE.MARKETPLACE.STANDARD_CHARGES s
       ON  s.CMS_CCN        = n.CMS_CCN
       AND s.CODE           = n.CODE
       AND s.CODE_TYPE      = n.CODE_TYPE
       AND LOWER(s.SETTING) = LOWER(n.SETTING)
```

`HOSPITAL_DIRECTORY` and `COMPLIANCE_VIOLATIONS` join on `CMS_CCN` alone.

## What do the NEGOTIATED_TYPE values mean?

`NEGOTIATED_TYPE` is the contract methodology as the hospital published it — case rate, per diem, fee schedule, and so on. A NULL means the source published no methodology cell; we never attribute a methodology the hospital didn't publish.

Averaging across methodologies is wrong because the values measure different things: a per diem is a price per day, a case rate is a price per episode, a fee-schedule line is a price per service. Averaging them produces a number with no unit. Group by `NEGOTIATED_TYPE` (and `SETTING`) first — see [Example Query 5](example-queries.md#5-methodology-breakdown) — and benchmark within a methodology, not across them.

## How do I read freshness?

`LATEST_INGEST_DATE` is the date Melange last loaded the hospital's file — not the hospital's own republish date. It is populated on every directory row, so recency is checkable per-row:

```sql
SELECT CMS_CCN, HOSPITAL_NAME, LATEST_INGEST_DATE,
       DATEDIFF('day', LATEST_INGEST_DATE, CURRENT_DATE) AS days_since_ingest
FROM MELANGE.MARKETPLACE.HOSPITAL_DIRECTORY
ORDER BY LATEST_INGEST_DATE DESC;
```

Refresh is periodic during the trial — we refresh as hospitals republish.

## What does AUDIT_FLAG mean, and which filter should I use?

Two grades — `CONSTANT_VALUE_FILL` (an evidence-backed verdict of a source-side fill) and `CONSTANT_RATE_PLATEAU` (a neutral structural observation consistent with real blanket or case-rate arrangements) — plus NULL, meaning no pattern detected (not a certification). Use `WHERE AUDIT_FLAG IS NULL` for strict per-code benchmarking, or `WHERE AUDIT_FLAG IS DISTINCT FROM 'CONSTANT_VALUE_FILL'` to exclude fills only. The full rationale is on the [Data Quality Methodology](data-quality.md) page.

## Who do I contact?

Sales and support: [steve@dourodata.com](mailto:steve@dourodata.com)
