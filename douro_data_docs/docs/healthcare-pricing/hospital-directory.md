# Hospital Directory — New York (Free)

The free directory is a **coverage list, not a registration list**. Every row is here because we fetched, parsed, and quality-gated that hospital's actual machine-readable price file. As of 2026-06-11: **154** New York hospitals = **132** with negotiated rates ∪ **136** with standard charges (114 publish both, 18 rates-only, 22 charges-only, and zero rows are registration-only).

It is the peer-scoping layer and the free companion to our paid dataset of 44.8 million hospital-published negotiated dollar rates: use the directory to build the peer set you'll benchmark against.

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

All three verified live on 2026-06-11 (154 rows; CMS_TYPE mix: 111 acute care / 25 psychiatric / 17 critical access / 1 children's).
