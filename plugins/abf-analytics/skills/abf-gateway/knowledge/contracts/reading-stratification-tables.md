# Reading Stratification Tables

How to read a `get_stratification_analytics_data` response (and a platform CSV export) correctly. This is a reading discipline, not a calculation guide: values returned by the analytics are authoritative and are stated as-is. Agent-side recomputation of anything the view already provides is a fallback that runs only with the analyst's explicit confirmation (last section).

Consult before narrating any multi-dimension analytics (vintage, cohort, MOB × date, or any view with more than one `group_by` dimension) — misreading grain and aggregation semantics on these tables is the leading cause of wrong analysis, especially once the row count is large enough that no one eyeballs every row.

## 1. Read the view metadata before the data

The CSV alone under-specifies its own semantics. Half of them live in the `list_transaction_analytics` entry for the view, which you already fetched during discovery:

- `group_by` tells you the dimensions and therefore the grain of every row.
- `aggregations` tells you, per measure, the `aggregation_function` - and that function is the single most informative bit about what a column means:

| `aggregation_function` | What the column is |
|---|---|
| `sum` | A cell subtotal - the measure summed over the assets in that row's cell only |
| `avg` | A cell average over the assets in that cell |
| `max` | Usually a constant being carried along (a cohort-level or benchmark value); the max is how a per-asset constant survives aggregation |
| `null` | A server-derived expression (ratios like CGL) - computed by the backend, not aggregated from rows |

Do not start reading numbers until each column in the response is matched to its metadata entry.

## 2. Determine the grain

One row = one cell of the `group_by` cross-product. In a vintage view grouped by `MOB x Pool Addition Date`, the row `(13, 2023-11)` describes the assets of the November 2023 vintage observed at month-on-book 13 - not the vintage, and not the month-on-book bucket across vintages. Every statement you make from a row must be scoped to its cell unless the column's role (below) says otherwise.

## 3. Classify every column into a role

| Role | How to recognize it | How to read it |
|---|---|---|
| Dimension | Listed in `group_by` | Coordinates of the cell; never a measure |
| Cell measure | `sum`/`avg` aggregation; varies row to row | Applies to that cell's assets only |
| Cohort constant | `max` aggregation; repeats identically across all rows sharing a cohort value | Describes the whole cohort; the repetition is the tell |
| Derived ratio | `aggregation_function: null`, `data_type: percentage` | Server-computed; semantics come from the glossary, not the name |
| Benchmark | Constant-like but varying within a cohort (e.g. UW CGL) | Sub-group level; see section 7 |
| `Count` | Always present | Row count of assets in the cell |

The two misreadings this classification prevents, both observed in practice:

- Reading a cell measure as a cohort total. `Issue Amount = 16,988` on the `(31, 2023-11)` row is the issuance of the six assets in that cell, not the November 2023 vintage's issuance.
- Ratioing cell measures at row level to produce a cohort metric. `Write Off / Issue Amount` on a single row is the loss rate of that cell's six assets - a meaningless number for the vintage.

## 4. Ask which assets are in the rows

Every view has an underlying population, and it may be filtered by construction. A loss vintage view's rows may contain only assets that had a loss event; a delinquency view's rows only delinquent assets. Cell measures are scoped to that population. Consequence: the sum of a view's cell measures is the population's total, which is not necessarily the portfolio's or the cohort's total. When the population matters to the conclusion you are about to draw ("the vintage originated X", "losses are Y% of the pool"), and the view's description or metadata does not state the population, say what the table says ("the view reports...") rather than what you assume it means.

## 5. Cumulative vs periodic - mixed grains in one row are normal

In vintage tables it is standard for a row to mix time-grains: the dollar columns are per-MOB slices (what happened at that month) while the derived ratio is cumulative through that MOB (the curve value). The tell is monotonicity: scan the ratio along the ordered dimension within one cohort - non-decreasing means cumulative; rising and falling means periodic. Verify once per table, then read consistently. Never add cumulative column values across MOBs (double-counting) and never read one MOB's slice as the curve.

## 6. Glossary discipline's gap

The glossary wins over column display names — see `../reference/common-pitfalls.md` § Trusting column name over glossary for the base rule. The gap that rule doesn't cover: derived ratio columns are frequently the ones with no glossary entry at all. A missing entry means the column's numerator, denominator, and population are unverified. In that case present the value attributed to its source ("the CGL Vintage view reports 58.0% at MOB 31", with the frontend `url`) and do not assert what it is a percentage of. Interpretation you cannot ground in the glossary, the view metadata, or the asset-class KPI catalog is not stated as fact.

## 7. Benchmarks and the total row

A benchmark column like UW CGL is typically `max`-aggregated and varies by asset sub-group within a cohort. Two consequences: parent and Total rows show the maximum of their children for that column, not a weighted benchmark; and averaging it across rows produces a number that exists nowhere in the platform. Before drawing a benchmark line or quoting "the" benchmark for a cohort, confirm the column is constant within that cohort; if it varies, present the range or leave it out. The `total_row` generally applies each column's own aggregation function - sums for sums, max for max - so it is not a weighted-average row, and derived ratio columns are often blank there.

## 8. Scales

Covered in `../reference/common-pitfalls.md` § Percentage scale confusion and the `stratification://data-types` resource - read both once per session before comparing raw values across columns.

## 9. Plausibility is part of reading, not calculating

Reading a table includes noticing when it cannot mean what it appears to say. Anchors available without computing anything: the asset-class KPI catalog's definitions and typical ranges, benchmark columns sitting in the same table (a realized ratio ten times the underwriting benchmark next to it is a flag, whatever the cause), curve shape (cumulative loss curves flatten with seasoning; a consumer-loan curve heading past 50% is presumptively an artifact), and early-life cohorts (under ~6 months on book) being structurally noisy.

When a value fails the plausibility read, the correct behavior is: deliver it as attributed data with its source `url`, state the anomaly in one line, and offer verification - "the view reports 58% CGL at MOB 31, which is implausible for this asset class; I can verify against the platform if you want". Do not silently reinterpret it, do not silently recalculate it, and do not build narrative on top of it.

## 10. Fallback - verification by recomputation (analyst-confirmed only)

Recomputing or reconciling a value the analytics already provides happens only after the analyst confirms they want it. The platform remains the source of truth throughout; the point of the fallback is to locate a discrepancy, never to replace the platform's number with an agent-computed one.

Confirmed steps, in order of cost:

1. Reconstruction: reported ratio x denominator column vs cumulative numerator, on 2-3 sampled cells. Detects scale and transcription issues.
2. Denominator coverage: compare the cohort-level denominator against the sum of the view's own cell-level amounts for that cohort. Approximate equality means the denominator covers only the view's (filtered) population - the known failure mode behind ratio inflation.
3. Platform comparison: open the response's frontend `url`, or ask the analyst for a platform export of the same view, and compare identical cells.

If a discrepancy is confirmed: report both numbers with sources, prefer the platform figure, and flag the view to the MCP integration owner. The discrepancy itself is a finding worth surfacing; the agent's recomputed value is a diagnostic, not a deliverable.

## Appendix - reading a platform CSV export

Analysts paste platform exports for cross-checks. Their layout differs from the API response:

- Hierarchical rows: a parent row carries the primary group value (e.g. the MOB number) in the `Group` column with group-level subtotals; the date rows nested under it are the secondary dimension (vintages). A `Total` row closes the file.
- Parent and Total rows show each column's aggregation result (max for `max`-aggregated columns like UW CGL), and the derived ratio is typically blank on parent rows.
- Cohort constants (e.g. Vintage Issue Amount) repeat on every nested row of the cohort; cell-level dollar columns are event-slice subtotals, exactly as in section 3.
- The export is the reconciliation baseline: when the API and the export disagree on the same cell, the export (the platform) wins.

## Worked example

Row from a CGL Vintage view (grouped MOB x Pool Addition Date): `MOB 31, vintage 2023-11, Issue Amount 16,988.58, Write Off 7,123.80, UW CGL 0.0355, Vintage Issue Amount 10,102,229.92, Count 6, CGL 0.580043`.

Correct reading: six assets of the November 2023 vintage are observed at month-on-book 31; those six were issued for 16,988.58 and have written off 7,123.80; the view carries 10.10M as the vintage-level issue amount; the view reports a cumulative CGL of 58.0% at this point on the curve; UW CGL here is a sub-group benchmark, not the vintage's single benchmark.

What the reading flags without any computation: 58% cumulative gross loss on consumer loans is not a plausible curve value, and it sits sixteen times above the UW benchmark in the same row - so this row is delivered as attributed data with its source link and an anomaly note, and verification is offered to the analyst. (In the real incident, the analyst-confirmed fallback then showed the vintage-level denominator matched the sum of the view's own rows - a filtered-population denominator - and the platform export gave 3.38% for the same cell.)
