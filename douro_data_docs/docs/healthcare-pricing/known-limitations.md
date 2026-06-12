# Known Limitations

We'd rather you read these here than discover them. Each one is a deliberate design position, stated plainly.

1. **Scope: New York only, by design.** This is a depth-first trial; additional states follow demand.

2. **Hospital counts differ by table.** Hospitals publish unevenly; the directory is the union of what we actually hold, not a registration list. Count with `COUNT(DISTINCT CMS_CCN)`, per table — the Marketplace listing carries the precise, date-stamped figures.

3. **Source files contain real defects.** Some hospitals publish constant-value fills — $0.01 floods, repeated magic numbers — and those rows carry the verdict-grade `CONSTANT_VALUE_FILL` flag with row-level evidence. We never delete them; you choose the filter. (Plateau-flagged rows are different: a neutral structural observation on real rates, not a defect.) See [Data Quality Methodology](data-quality.md).

4. **Payer canonicalization is partial and growing.** About two-thirds of negotiated-dollar rows are mapped to a canonical parent payer; the remainder retain their hospital-reported `PAYER_NAME`, present on every row, so the unmapped tail is always matchable. Nothing is dropped. The canonical map grows with the catalog.

5. **System affiliation is sparsely curated today** and expanding. `SYSTEM_NAME` and `SYSTEM_ROLE` are populated where assigned.

6. **The compliance signal exists only where checks have run.** An absence of recorded violations is not a certification of compliance — `HAS_COMPLIANCE_CHECK = FALSE` is the explicit "not yet examined" flag.

7. **`LATEST_INGEST_DATE` is our load date**, not the hospital's own republish date. It tells you when we last pulled the file.

8. **Notable absences (as of 2026-06-12).** NewYork-Presbyterian (all Manhattan-side campuses, via its blanket file), Mount Sinai Morningside, NY Eye & Ear, and Montefiore Nyack joined on 2026-06-12. The Hospital for Special Surgery remains absent — its file defeats our current download paths — along with a small upstate tail. We disclose gaps rather than paper over them; they join as retrieval succeeds.

9. **Refresh is periodic during the trial.** We refresh as hospitals republish; recency is checkable per row (`INGEST_DATE` on rate rows, `LATEST_INGEST_DATE` on the directory) rather than promised by an SLA.
