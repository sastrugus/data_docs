# Hospital Directory — New York (Free)

The free directory is a **coverage list, not a registration list**. Every row is here because we fetched, parsed, and quality-gated that hospital's actual machine-readable price file — every row holds negotiated rates, standard charges, or both, and zero rows are registration-only. It covers 150+ New York hospitals; counts differ by table because hospitals publish unevenly, and the directory is the union of what we actually hold.

It is the peer-scoping layer and the free companion to our paid dataset of tens of millions of hospital-published negotiated dollar rates: use the directory to build the peer set you'll benchmark against. The Marketplace listing carries the precise, date-stamped figures for each release.

## Column reference

One row per hospital in `MARKETPLACE.HOSPITAL_DIRECTORY`:

| Column | Meaning |
|---|---|
| `CMS_CCN` | CMS Certification Number — the join key everywhere. Count hospitals with `COUNT(DISTINCT CMS_CCN)`. |
| `HOSPITAL_NAME` | Hospital name. |
| `ADDRESS`, `CITY`, `STATE_CODE`, `ZIP_CODE` | Location. |
| `CMS_TYPE` | CMS facility type (acute care, psychiatric, critical access, children's). |
| `CMS_OWNERSHIP` | CMS ownership category. |
| `CMS_RATING` | CMS star rating, 1–5; null where CMS has not rated. |
| `SYSTEM_NAME`, `SYSTEM_ROLE` | Curated health-system affiliation (flagship / satellite / community) — populated where assigned; curation is growing. |
| `LATEST_INGEST_DATE` | The date **we** last loaded this hospital's file — not the hospital's own republish date. Populated on every row. |
| `AVG_COMPLIANCE_SCORE`, `IS_FULLY_COMPLIANT` | Rolled-up signal from our compliance checker, populated where checks have run. Absence of recorded violations is not a certification of compliance. |

## Quick start

Three queries to orient yourself. Replace the database name with your mounted database name.

How many hospitals? Count CCNs, not rows — some facilities share one published source file but are distinct CMS-registered hospitals:

```sql
SELECT COUNT(DISTINCT CMS_CCN) AS hospitals
FROM HOSPITAL_DIRECTORY_NEW_YORK_FREE.MARKETPLACE.HOSPITAL_DIRECTORY;
```

Build a peer set — acute-care hospitals, best-rated first:

```sql
SELECT CMS_CCN, HOSPITAL_NAME, CITY, CMS_OWNERSHIP, CMS_RATING
FROM HOSPITAL_DIRECTORY_NEW_YORK_FREE.MARKETPLACE.HOSPITAL_DIRECTORY
WHERE CMS_TYPE = 'Acute Care Hospitals'
ORDER BY CMS_RATING DESC NULLS LAST, HOSPITAL_NAME
LIMIT 25;
```

Freshness and transparency posture across the directory:

```sql
SELECT CMS_CCN, HOSPITAL_NAME, LATEST_INGEST_DATE,
       AVG_COMPLIANCE_SCORE, IS_FULLY_COMPLIANT
FROM HOSPITAL_DIRECTORY_NEW_YORK_FREE.MARKETPLACE.HOSPITAL_DIRECTORY
ORDER BY LATEST_INGEST_DATE DESC, HOSPITAL_NAME;
```

All three are verified live against the share — the first gives you the current hospital count straight from the data.
