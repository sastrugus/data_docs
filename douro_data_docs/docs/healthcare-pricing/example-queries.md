# Example Queries

Eight worked examples against the paid trial, smoke-tested live on 2026-06-11 (8/8 pass). They use a four-hospital demo peer set — Mount Sinai (330024), Bellevue (330204), NewYork-Presbyterian/Queens (330055), White Plains (330304) — with Bellevue as the cash-comparison anchor. The SQL below uses `YOUR_DB` as the database name. Replace YOUR_DB with the database name you chose when mounting the share.

Two conventions you'll see throughout: `SETTING` is compared through `LOWER()` as a casing guard, and the strict-benchmarking filter `AND AUDIT_FLAG IS NULL` appears commented where it applies — uncomment it for per-code work, or use `AUDIT_FLAG IS DISTINCT FROM 'CONSTANT_VALUE_FILL'` to exclude fills only. See [Data Quality Methodology](data-quality.md) for when to use which.

## 1. Single-hospital lookup

Everything we hold for one hospital, with standard charges joined alongside:

```sql
SELECT
    n.CMS_CCN, n.HOSPITAL_NAME, n.CODE_TYPE, n.CODE, n.DESCRIPTION, n.SETTING,
    n.PAYER_NAME, n.PAYER_CANONICAL, n.PAYER_LINE_OF_BUSINESS, n.PLAN_NAME,
    n.NEGOTIATED_RATE, n.NEGOTIATED_TYPE, n.RATE_TYPE,
    s.GROSS_CHARGE, s.DISCOUNTED_CASH
FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES n
LEFT JOIN YOUR_DB.MARKETPLACE.STANDARD_CHARGES s
       ON  s.CMS_CCN        = n.CMS_CCN
       AND s.CODE           = n.CODE
       AND s.CODE_TYPE      = n.CODE_TYPE
       AND LOWER(s.SETTING) = LOWER(n.SETTING)
WHERE n.STATE_CODE = 'NY'
  AND n.CMS_CCN = '330024'
ORDER BY n.CODE_TYPE, n.CODE, n.PAYER_NAME;
```

## 2. Payer lens on one hospital

One hospital, one canonical payer, one line of business — commercial Aetna at Mount Sinai, dollar rates only:

```sql
SELECT
    HOSPITAL_NAME, CODE_TYPE, CODE, DESCRIPTION, SETTING,
    PAYER_NAME, PAYER_CANONICAL, PAYER_LINE_OF_BUSINESS, PLAN_NAME,
    NEGOTIATED_RATE, NEGOTIATED_TYPE, RATE_TYPE
FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES
WHERE STATE_CODE = 'NY'
  AND CMS_CCN = '330024'
  AND PAYER_CANONICAL = 'AETNA'
  AND PAYER_LINE_OF_BUSINESS = 'commercial'
  AND RATE_TYPE = 'negotiated_dollar'
  AND CODE_TYPE IN ('CPT','HCPCS','MS-DRG','APR-DRG','APC','NDC')
ORDER BY CODE_TYPE, CODE, PLAN_NAME;
```

## 3. Cross-hospital payer comparison

The flagship: what one payer pays each hospital in the peer set for the same codes, same setting:

```sql
SELECT
    r.CODE_TYPE, r.CODE, r.DESCRIPTION, r.SETTING, r.HOSPITAL_NAME,
    r.PAYER_NAME, r.PLAN_NAME, r.NEGOTIATED_RATE, r.NEGOTIATED_TYPE, r.RATE_TYPE
FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES r
WHERE r.STATE_CODE = 'NY'
  AND r.CMS_CCN IN ('330024','330204','330055','330304')
  AND r.PAYER_CANONICAL = 'AETNA'
  AND r.PAYER_LINE_OF_BUSINESS = 'commercial'
  AND r.RATE_TYPE = 'negotiated_dollar'
  AND r.NEGOTIATED_RATE > 0
  AND r.NEGOTIATED_TYPE IS NOT NULL
  AND r.CODE_TYPE IN ('CPT','HCPCS','MS-DRG','APR-DRG','APC')
  AND r.CODE IN ('27447','29881','45378','70553','99285')
  -- AND r.AUDIT_FLAG IS NULL  -- strict per-code benchmarking recipe
ORDER BY r.CODE, LOWER(r.SETTING), r.NEGOTIATED_RATE DESC;
```

## 4. Price position within the market

The other flagship: where one hospital's rate sits against the market's 10th / 50th / 90th percentile, per code and setting:

```sql
WITH aetna_dollars AS (
    SELECT CMS_CCN, HOSPITAL_NAME, CODE_TYPE, CODE, DESCRIPTION, SETTING,
           LOWER(SETTING) AS SETTING_KEY, NEGOTIATED_RATE
    FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES
    WHERE STATE_CODE = 'NY'
      AND RATE_TYPE = 'negotiated_dollar' AND NEGOTIATED_RATE > 0
      AND PAYER_CANONICAL = 'AETNA' AND PAYER_LINE_OF_BUSINESS = 'commercial'
      AND CODE_TYPE IN ('CPT','HCPCS','MS-DRG','APR-DRG','APC')
      AND CODE IN ('27447','29881','45378','70553','99285')
      -- AND AUDIT_FLAG IS NULL  -- strict per-code benchmarking recipe
),
market AS (
    SELECT CODE_TYPE, CODE, SETTING_KEY,
           PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY NEGOTIATED_RATE) AS p10,
           PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY NEGOTIATED_RATE) AS median,
           PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY NEGOTIATED_RATE) AS p90,
           COUNT(DISTINCT CMS_CCN) AS n_hospitals
    FROM aetna_dollars GROUP BY CODE_TYPE, CODE, SETTING_KEY
)
SELECT a.HOSPITAL_NAME, a.CODE_TYPE, a.CODE, a.DESCRIPTION, a.SETTING,
       a.NEGOTIATED_RATE AS my_rate, m.p10, m.median, m.p90, m.n_hospitals,
       CASE WHEN a.NEGOTIATED_RATE <= m.p10 THEN 'at/below 10th pct'
            WHEN a.NEGOTIATED_RATE <  m.median THEN 'below median'
            WHEN a.NEGOTIATED_RATE <  m.p90 THEN 'above median'
            ELSE 'at/above 90th pct' END AS position
FROM aetna_dollars a
JOIN market m ON m.CODE_TYPE=a.CODE_TYPE AND m.CODE=a.CODE AND m.SETTING_KEY=a.SETTING_KEY
WHERE a.CMS_CCN = '330024'
ORDER BY a.CODE, LOWER(a.SETTING);
```

## 5. Methodology breakdown

What contract methodologies each hospital publishes, by setting — run this before averaging anything (see [FAQ](faq.md) on why mixing methodologies is wrong):

```sql
SELECT
    HOSPITAL_NAME, LOWER(SETTING) AS setting, NEGOTIATED_TYPE, RATE_TYPE,
    COUNT(*) AS row_count, COUNT(DISTINCT CODE) AS distinct_codes,
    COUNT(DISTINCT PAYER_CANONICAL) AS distinct_canonical_payers,
    COUNT(DISTINCT PAYER_NAME) AS distinct_payer_strings
FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES
WHERE STATE_CODE = 'NY'
  AND CMS_CCN IN ('330024','330204','330055','330304')
GROUP BY HOSPITAL_NAME, LOWER(SETTING), NEGOTIATED_TYPE, RATE_TYPE
ORDER BY HOSPITAL_NAME, setting, row_count DESC;
```

## 6. Cash-vs-negotiated spread

How far each negotiated rate sits above (or below) the hospital's own discounted cash price:

```sql
SELECT
    n.HOSPITAL_NAME, n.CODE_TYPE, n.CODE, n.DESCRIPTION, n.SETTING,
    s.DISCOUNTED_CASH, n.NEGOTIATED_RATE,
    (n.NEGOTIATED_RATE - s.DISCOUNTED_CASH) AS negotiated_minus_cash,
    ROUND((n.NEGOTIATED_RATE - s.DISCOUNTED_CASH)/NULLIF(s.DISCOUNTED_CASH,0)*100,1) AS pct_over_cash
FROM YOUR_DB.MARKETPLACE.NEGOTIATED_RATES n
JOIN YOUR_DB.MARKETPLACE.STANDARD_CHARGES s
       ON  s.CMS_CCN        = n.CMS_CCN
       AND s.CODE           = n.CODE
       AND s.CODE_TYPE      = n.CODE_TYPE
       AND LOWER(s.SETTING) = LOWER(n.SETTING)
WHERE n.STATE_CODE = 'NY'
  AND n.CMS_CCN IN ('330024','330204','330055','330304')
  AND n.RATE_TYPE = 'negotiated_dollar' AND n.NEGOTIATED_RATE > 0
  AND s.DISCOUNTED_CASH IS NOT NULL
  AND n.PAYER_CANONICAL = 'AETNA' AND n.PAYER_LINE_OF_BUSINESS = 'commercial'
  AND n.CODE IN ('27447','29881','45378','70553','99285')
ORDER BY pct_over_cash DESC;
```

## 7. Compliance posture

Row-level CMS-transparency findings for the peer set, worst first. No rows does not mean certified clean — `HAS_COMPLIANCE_CHECK = FALSE` is the explicit "not yet examined" flag:

```sql
SELECT
    CMS_CCN, HOSPITAL_NAME, CMS_TYPE, VIOLATION_CODE, SEVERITY, FIELD_PATH,
    DESCRIPTION, CMS_REGULATION_SECTION, RESOLUTION_STATUS,
    COMPLIANCE_SCORE, IS_COMPLIANT, CHECKED_AT
FROM YOUR_DB.MARKETPLACE.COMPLIANCE_VIOLATIONS
WHERE STATE_CODE = 'NY'
  AND CMS_CCN IN ('330024','330204','330055','330304')
ORDER BY
    CASE SEVERITY WHEN 'critical' THEN 0 WHEN 'high' THEN 1 ELSE 2 END,
    COMPLIANCE_SCORE ASC, CMS_CCN;
```

## 8. Data freshness

How recently we loaded each hospital's file. `LATEST_INGEST_DATE` is our load date, not the hospital's self-reported republish date:

```sql
SELECT
    CMS_CCN, HOSPITAL_NAME, SYSTEM_NAME, SYSTEM_ROLE, CITY, STATE_CODE,
    CMS_TYPE, CMS_OWNERSHIP, CMS_RATING, LATEST_INGEST_DATE,
    DATEDIFF('day', LATEST_INGEST_DATE, CURRENT_DATE) AS days_since_ingest,
    AVG_COMPLIANCE_SCORE, IS_FULLY_COMPLIANT
FROM YOUR_DB.MARKETPLACE.HOSPITAL_DIRECTORY
WHERE STATE_CODE = 'NY'
  AND CMS_CCN IN ('330024','330204','330055','330304')
ORDER BY LATEST_INGEST_DATE DESC;
```
