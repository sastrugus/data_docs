# Known Limitations

We'd rather you read these here than discover them. Each one is a deliberate design position, stated plainly.

1. **Scope: New York only, by design.** This is a depth-first trial; additional states follow demand.

2. **Hospital counts differ by table** — 132 negotiated rates / 136 standard charges / 154 directory (as of 2026-06-11). Hospitals publish unevenly; the directory is the union of what we actually hold, not a registration list.

3. **Source files contain real defects.** Some hospitals publish constant-value fills — $0.01 floods, repeated magic numbers — and those rows carry the verdict-grade `CONSTANT_VALUE_FILL` flag with row-level evidence. We never delete them; you choose the filter. (Plateau-flagged rows are different: a neutral structural observation on real rates, not a defect.) See [Data Quality Methodology](data-quality.md).

4. **Payer canonicalization is partial and growing.** About two-thirds of negotiated-dollar rows are mapped to a canonical parent payer; the remainder retain their hospital-reported `PAYER_NAME`, present on every row, so the unmapped tail is always matchable. Nothing is dropped. The canonical map grows with the catalog.

5. **System affiliation is sparsely curated today** and expanding. `SYSTEM_NAME` and `SYSTEM_ROLE` are populated where assigned.

6. **The compliance signal exists only where checks have run.** An absence of recorded violations is not a certification of compliance — `HAS_COMPLIANCE_CHECK = FALSE` is the explicit "not yet examined" flag.

7. **`LATEST_INGEST_DATE` is our load date**, not the hospital's own republish date. It tells you when we last pulled the file.

8. **Notable absences (as of 2026-06-12).** A small tail of hospitals whose websites block automated retrieval is not yet in the dataset — including NewYork-Presbyterian's Manhattan campuses and the Hospital for Special Surgery. We disclose gaps rather than paper over them; these hospitals join as retrieval succeeds.

9. **Refresh is periodic during the trial.** We refresh as hospitals republish; recency is checkable per row (`INGEST_DATE` on rate rows, `LATEST_INGEST_DATE` on the directory) rather than promised by an SLA.
