# RISE Humanities Data Benchmark Dataset: Complete Stored Results

Release `2026-10-29.6` · schema `1.3.0` · data through 2026-10-29 · source commit `41141a5d4`

Every stored result of the Humanities Data Benchmark, as joinable **Parquet** tables.

| Table | Rows | Unit of observation |
|---|---|---|
| `runs` | 3,805 | one stored run directory |
| `requests` | 166,380 | one stored request file |
| `scores_long` | 3,902,043 | one numeric metric observation, as recorded |
| `rescored_fields` | 404,680 | one field observation from a subsequent scoring pass |
| `metrics` | 124 | one metric definition |

## What is in this package

- `CHANGELOG.md`
- `CITATION.cff`
- `README.md`
- `coverage.csv.gz`
- `datapackage.json`
- `diagnostics.jsonl.gz`
- `examples/business_letters_cost.py`
- `manifest.json`
- `metrics.parquet`
- `requests.parquet`
- `rescored_fields.parquet`
- `runs.parquet`
- `scores_long.parquet`
- `source_manifest.jsonl.gz`
- `payloads/` (137 files)

Text files are gzipped so that every file clears the size limits of the repository this
release is committed to; `packaging.json` maps each one back to the name and SHA-256 the
build recorded, so it can be reconciled with `manifest.json`.

`payloads/` holds the complete original request and scoring JSON as gzipped JSON Lines, one file per benchmark and run month (`<benchmark>/<YYYY-MM>.jsonl.gz`), plus `run_scoring.jsonl.gz` and the re-scoring records in `rescored_detail.jsonl.gz`: 137 files. Most of their bytes are third-party model output; see Licence.

## Reading it

```python
import pyarrow.parquet as pq
runs = pq.read_table("runs.parquet").to_pylist()
```

`pyarrow` is the only dependency. `examples/business_letters_cost.py` is a worked analysis
that runs against this directory as it stands.

## Scope and missing values

The full export retains every stored run directory and request file, including failed
requests, unscored runs, and runs whose test configuration cannot be resolved. Status
columns distinguish these cases so that inclusion criteria can be applied explicitly.
Completeness refers to the stored archive; attempts that were not saved cannot be recovered.
The one exception is the framework's own test benchmarks (`test_*`), scaffolds that exercise
the harness and measure nothing; their runs are left out.

**Null values represent missing or unavailable information, not zero.** For example, a null
`input_tokens` value indicates that no usable input-token count was recorded. Missing values
should remain distinct from measured zeros when calculating aggregates.

## Keys and joins

- `runs.run_id` is `<test_id>@<date>`. A test run on another date is a different run.
- `requests` joins to `runs` on `run_id`, and its own key is `(run_id, object_id)`.
- `scores_long` joins on `run_id` and, for request- and field-level rows, `object_id`.
  Its `metric_id` joins to `metrics.metric_id`. Select the appropriate `level` before
  joining or aggregating to avoid counting both summary and component observations.
- `rescored_fields` joins to `requests` on `(run_id, object_id)` and is keyed by
  `(run_id, object_id, field_path)`. It has no `metric_id`; its `score` column contains
  the per-field value supplied by the benchmark's scorer.
- `object_id` is a **string**. Leading zeros are significant (`00414956` is an identifier,
  not a number) and some ids contain non-ASCII characters.
- `line` is populated only when an object id is exactly `line_<digits>`. It is presentation
  metadata and never a join key.

## Recorded and recomputed evaluations

`scores_long` preserves numeric observations from the original run records.
`rescored_fields` provides supplementary field-level evaluations produced by a subsequent
scoring pass over stored responses, using the scoring code and ground truths available
at that time. The dataset exporter imports these evaluations from the comparison-detail
files; it does not execute scorers or modify the original results.

In this release, the original records contain field-level detail for four benchmarks.
The supplementary evaluations extend field-level coverage to twelve benchmarks by
processing inputs without stored field detail. This coverage does not imply that every
request has a field-level evaluation. If the comparison-detail files are unavailable,
`rescored_fields` is empty.

The following columns describe the supplementary evaluations:

| Column | Interpretation |
|---|---|
| `score` | Per-field value returned by the scorer; null where no numeric similarity is assigned, including evaluations based on counts |
| `rescored_date` | Date of the subsequent scoring pass, distinct from the original run date and export date |
| `scorer_revision` | Recorded commit that last changed the benchmark's scorer file |
| `ground_truth_revision` | Recorded commit that last changed the benchmark's ground truths |
| `scorer_dirty` | Whether uncommitted scorer or ground-truth changes were reported; true means the revision identifiers do not fully describe the evaluated state |
| `reproduces_stored_score` | Whether the recomputed numeric metrics for the input equal its stored numeric metrics; null when no comparison result is available |

A false `reproduces_stored_score` value identifies a discrepancy, but does not by itself
establish its cause. A true value indicates agreement in the compared metrics, not that
the scorer or ground truths are unchanged. This input-level flag is repeated for each
field belonging to the input.

Recorded and recomputed evaluations represent different evaluation contexts. Analyse them
separately unless the methods and reference data have been shown to be comparable.

## Metrics and aggregation

`scores_long` stores one numeric observation per row at the run, request, or field level.
The `metrics` table defines each observation's meaning, unit, direction, and aggregation
rule. Consult these definitions before comparing or combining values.

- `metric_role` separates a `performance` measurement from a `count` and from a
  `parameter`. `iou_threshold` is a setting the scorer was given; `mean_iou` is what it
  measured. They are not comparable.
- `aggregation` specifies the supported operation: `sum` for counts,
  `mean_over_requests` for applicable performance metrics, `not_summable` for ratios,
  and `not_aggregatable` for parameters.
- `direction` indicates whether higher or lower values represent better performance.
  Character error rate (`cer`) is lower-is-better; it should not be pooled directly with
  higher-is-better similarity scores.
- Metric ids are **scoped by benchmark** because the same name means different things.
  `f1_macro` averages three extraction categories in `business_letters` but each advert's
  F1 over edits in `book_advert_xml`, and their counts are of different things (extractions,
  edits). A shared name or scale does not establish comparability across tasks.

F1 scores measure performance rather than the number of correct extractions. Analyses
requiring counts should use the corresponding true-positive metrics where available.

## Cost estimates

Request-level costs are provided in two separate column families:

| Columns | Interpretation |
|---|---|
| `stored_*` | Cost estimates copied from the request record: priced by the runner at the time, or filled in afterwards at the price in force on the run's date where the runner recorded none |
| `derived_*` | Cost estimates recalculated during export from recorded token counts and dated pricing entries, using a consistent method across the archive |

Some providers bill reasoning tokens outside `output_tokens`. `reasoning_tokens` holds
that uncounted remainder, so `input_tokens + output_tokens + reasoning_tokens` equals
`total_tokens`, and it is zero for providers that count reasoning inside the output.
Reasoning is charged at the output price and included in both total-cost columns. Where no
`total_tokens` was recorded the remainder is unknown: `reasoning_tokens` is null and
`cost_provenance` is `derived_reasoning_unknown`, with a total covering input and output
only.

**Derived costs are estimates, not verified billing amounts.** The exporter selects the
most recent matching price-table entry on or before the run date, with no maximum age.
`pricing_bucket_date` identifies the entry and `pricing_age_days` reports its age at the
time of the run. Gaps in the pricing history therefore affect the reliability of estimates.
The pricing-table version and source hash are recorded in `manifest.json` and
`source_manifest.jsonl` so that calculations can be reproduced.

Historical reconstruction uses dated pricing entries; it does not reconstruct the
evaluation methods used for the original scores. A derived cost reflects the available
price history rather than a verified historical charge, and it may change if that history
is corrected. Supplementary evaluations carry the provenance described in
[Recorded and recomputed evaluations](#recorded-and-recomputed-evaluations).

`cost_provenance` records whether a cost was derived or why it is unavailable. If only
one token count is available, the corresponding cost component is retained and the total
remains null. Missing prices are also represented as null, which a derived cost of
`0.0` is not: a zero is a recorded price of zero. Some models carry no per-token charge,
whether because they run on institutional hardware or because the provider serves them
free within rate limits. Distinguish the two before excluding zero-cost records.

Cost aggregates are **observed subtotals over saved requests**, not a complete record of
expenditure. The runner does not save a request when no answer is returned, and a same-day
rerun may overwrite an earlier result. Saved failures remain included because failed
attempts can incur costs. Report cost coverage alongside any subtotal or efficiency ratio.

## Dates and repeated runs

`date` is the name of the directory in which a run was stored, not an execution timestamp.
`timestamp_raw` preserves the recorded timestamp. `timestamp_utc` is **null for every row
in this release** because the stored timestamps lack UTC offsets and the source machine's
timezone is unknown. Sub-day ordering across machines cannot be established reliably.

The same test can occur on multiple dates, and same-day reruns may have overwritten earlier
results. These observations should not be treated as independent replicates without
further justification. State how repeated runs are selected or combined in an analysis.

## Selection and comparability

The default analytical view selects runs with `hidden = false`. Require an explicit false
value: visibility is null where it could not be resolved, and the treatment of nulls in
inequality filters varies between tools. Hidden and unresolved runs remain in the export;
report their exclusion when presenting results.

`configured_provider` and `configured_model` identify the test configuration.
`response_provider` and `response_model` preserve the identity reported in the stored
response. These values can differ for aliased or routed models. Specify which identity
is used for grouping and comparison.

Configuration and benchmark metadata describe the **export snapshot**, which may differ
from the configuration at execution time. Prompts, ground truths, and scoring code changed
over the archive's history, and each benchmark records that history as versions:
`benchmark_version` is the version a run was made with, `benchmark_version_scored` the one
whose scorer and ground truth its stored scores stand under: the latest re-scoring made on
or after the run's day (a re-scoring checks every stored run and records itself in
`scoring.json` only where a score changed). Scores share a basis when the major number of
`benchmark_version_scored` is the same, or 0.y before 1.0.0. Newer runs record the version
they were made with (`benchmark_version_basis = recorded`). For the others a version is
assigned from the run's first request time, read as Europe/Zurich, against the time its
change was committed (`inferred`). `ambiguous` marks runs made earlier on the day a version
took effect, runs recorded while the benchmark's files had uncommitted changes, and runs
resumed from answers made with another benchmark state; for these the next version is also
possible. The versions themselves, with dates, commits
and summaries, are listed in each benchmark's `meta.json` in the source repository.

Derived costs use dated pricing entries, as described above, whereas scores retain their
original evaluation context. Historical cost reconstruction does not resolve changes in
models, prompts, or source documents. Comparisons of scores across dates should
group by `benchmark_version_scored`.

## Coverage and diagnostics

`coverage.csv` reports row counts, non-null counts, null counts, invalid-value counts, and
the fraction of non-null values for each column in `runs`, `requests`, `scores_long`,
and `rescored_fields`.
Coverage is reported globally and, where the table contains the relevant columns, by
benchmark, configured provider, date, and their combination. `scores_long` has global
coverage only. Null grouping values are retained under the label `<null>`.
`rescored_fields` has global and per-benchmark coverage; its scoring date is recorded
as `rescored_date`, which is not a coverage grouping dimension.

`diagnostics.jsonl` records issues detected during export, including their severity and
the action taken. An empty file indicates that no diagnostic entries were generated.

## Reproducibility

To rebuild this dataset, use the source repository at the commit recorded in
`manifest.json`, with the recorded inputs and dependencies. Run from the repository root:

```console
python -m scripts.compare.generate_compare_detail
python -m scripts.export_dataset --dataset-version 2026-10-29.6
```

`source_manifest.jsonl` records a SHA-256 hash for each consumed input. `committed` says
whether Git held the file unchanged at the source commit (null where Git could not be
asked); only those inputs can be recovered from the commit. `origin` `generated` marks the
re-scored field detail written by the first command, with the scorer and ground-truth
revisions it was produced from; `source` marks everything else. `source_worktree_dirty` in
`manifest.json` covers tracked files only. `manifest.json`
records output hashes and build metadata; the manifest itself is excluded from the output
hash list. These records support verification of the files and identification of changes
between builds. Builds using selection filters are marked as partial and are not releases.

## Licence

The project-authored tables, metric dictionary, coverage and manifest files, documentation,
and ground truths are released under [Creative Commons Attribution 4.0 International (CC-BY-4.0)](https://creativecommons.org/licenses/by/4.0/).
This licence is separate from the GPL-3.0 licence of the benchmark software.

The payload sidecars also contain responses returned by third-party models, retained as
evidence of model behaviour. The project does not claim authorship of these responses or
grant rights beyond those it holds; reuse may depend on the relevant provider's terms.
Benchmark input documents are not included in this export.

## Versions

Three version numbers answer three different questions.

- **The dataset version** (`2026-10-29.6`) names a release: `<data cutoff>.<serial>`, the last
  day of runs it holds, and a serial that counts rebuilds at the same cutoff. Each release is
  a full snapshot, not an addition to the previous one: stored results are corrected and
  re-scored between releases, so the same run can carry a different score in two releases.
  [CHANGELOG.md](CHANGELOG.md) lists what changed in each.
- **The schema version** (`1.3.0`) describes the format, in semantic versioning: MAJOR
  when a table or column is removed, renamed or changes meaning, MINOR when one is added,
  PATCH for documentation and metadata only. Code written against one schema reads every
  release with the same MAJOR.
- **Benchmark versions** say which scores can be compared. `benchmark_version_scored` in
  `runs` is the benchmark version whose scorer and ground truth produced a run's stored
  scores. Compare scores only within one MAJOR benchmark version; the dataset version says
  nothing about this.

## Citation

Use the citation metadata in [CITATION.cff](CITATION.cff) and identify the dataset version
used in the analysis. Cite the version DOI of the release you used; the concept DOI [10.5281/zenodo.23821532](https://doi.org/10.5281/zenodo.23821532) names the dataset as a whole and resolves to the newest release. CITATION.cff lists both.
