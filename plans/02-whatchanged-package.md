# Plan 02. `whatchanged`: a Python package that says which segment explains a metric change

**Repository:** `dbechrakis/whatchanged` (new, the owner creates it empty and public). **Branch:** `claude/v0-1`. **Size:** four to six weeks, eight sessions. **Release target:** version 0.1.0 on PyPI.

## 1. Why this exists

The profile's motto is "What changed? Which segment explains it? What should we do next?". No project in the portfolio is a tool that answers the first two for someone else's data. Every project is a case study on one dataset.

This plan builds a small, tested, documented library that a business analyst installs with `pip install whatchanged` and points at a table. Its value for the portfolio is a different kind of evidence: an interface designed for strangers, typed and versioned code, documentation, a release process, and a benchmark that says where the methods fail.

**The gap is real, as found on 15 November 2026.**

- `shaido987/riskloc` implements Adtributor, Squeeze, HotSpot and others, as a collection of research scripts. It is not an installable library.
- Salesforce's PyRCA is a library, and it is built for causal graphs of service metrics, not for the dimensions of a business table.
- The exact mix and rate decomposition in `superstore-shipping-region-analysis/analysis/margin_drivers.py` is the owner's own tested code and is the seed of this package.

**What the package is not.** It does not find causes. It does arithmetic that is exact, a search that is heuristic, and a noise check that is statistical, and it says which is which in every output.

## 2. Owner actions

| Action | When |
|---|---|
| Create the empty public repository `dbechrakis/whatchanged` and attach it to the session | Before WP0 |
| Decide the licence. MIT is assumed, as in `uplift-analyzer` | Before WP0 |
| Create a PyPI account with two-factor login, and a TestPyPI account | Before WP8 |
| On both, add a pending trusted publisher for repository `dbechrakis/whatchanged`, workflow `release.yml`, environment `pypi` | Before WP8 |
| Create the tag that triggers the release | WP8 |

The name `whatchanged` was free on PyPI on 15 November 2026. **Gate in WP0:** check `https://pypi.org/pypi/whatchanged/json` again. If it is taken, use `metricdrivers`, check that too, and replace the name everywhere. If both are taken, stop and ask.

No secret is ever stored in the repository. Trusted publishing needs no token.

## 3. What a user sees

```python
import pandas as pd
import whatchanged as wc

orders = pd.read_parquet("orders.parquet")

result = wc.compare(
    orders,
    metric=wc.Ratio("profit", "sales"),          # or wc.Sum("revenue"), wc.Mean("basket")
    split="month", baseline="2026-09", current="2026-10",
    dimensions=["region", "category", "channel"],
)

result.change                      # the metric in each period and the difference
result.decompose(by=["category", "subcategory"])   # exact: mix against rate, sums to the change
result.explain()                   # heuristic: the few segments that account for most of it
result.noise(unit="order_id")      # statistical: is each contribution outside its usual range
print(result.summary())            # five lines a manager can read
```

And from a shell:

```bash
whatchanged orders.parquet --metric "profit/sales" --split month --baseline 2026-09 --current 2026-10 --by region,category,channel
```

The two sides of a comparison may be two periods, two regions, or the two arms of a test. `split`, `baseline` and `current` name a column and two of its values.

## 4. Design

### 4.1 Layout

```
src/whatchanged/
  __init__.py        the public names, nothing else
  metrics.py         Sum, Ratio, Mean
  cube.py            aggregate rows to cells, in pandas, or in DuckDB when given a connection
  decompose.py       exact mix and rate decomposition
  attribute.py       Adtributor, single and recursive
  noise.py           bootstrap over units, and placebo over past period pairs
  result.py          Comparison, Decomposition, Explanation, NoiseCheck (frozen dataclasses)
  synthetic.py       tables with a planted cause, for tests and the benchmark
  cli.py
  py.typed
tests/
benchmarks/
docs/
```

Core dependencies: `numpy` and `pandas` only. `duckdb` is an optional extra (`pip install whatchanged[duckdb]`). Python 3.10 to 3.13.

### 4.2 Metrics

- `Sum(column)`: additive. Cells add up to the total.
- `Ratio(numerator, denominator)`: the ratio of two sums. Not additive. This is where naive tools go wrong, and it is the main case.
- `Mean(column)`: `Ratio(column, row count)`.

### 4.3 `decompose`: exact

Generalises `margin_drivers.py`.

- For a `Ratio` and an ordered list of nested dimensions (for example category, then subcategory), the metric of a side equals the sum over leaf cells of share at level one × share within it at level two × … × the leaf's own rate.
- The change between the two sides is split over the factors (one mix factor per level, and the rate within the leaf) by averaging each factor's marginal effect over every order in which the factors can be switched from baseline to current. This is the Shapley allocation, and it makes the split independent of order.
- **The contributions sum to the change exactly.** The function asserts it to 1e-9 and raises if it fails.
- For a `Sum` the decomposition is the plain difference per cell, optionally split into "volume" and "value per unit" when the user names a unit count.
- **Cells present on one side only.** The default is `missing="error"`, as in the original code, which refuses to invent a rate. `missing="entry_exit"` reports such cells as a separate component named "entered" or "left", so that the split still sums.
- Cost: the number of orders is the factorial of the number of factors. Refuse more than six factors with a clear message.

### 4.4 `explain`: heuristic

Adtributor, from Bhagwan and others, "Adtributor: Revenue Debugging in Advertising Systems" (NSDI 2014).

**The executing model must read the paper and implement from it, not from memory and not from this summary.** The summary below is for orientation and is to be checked against sections 3 and 4 of the paper.

- For an additive metric, an element's explanatory power is its change divided by the total change. Its surprise is the Jensen-Shannon divergence term between its share of the baseline and its share of the current total.
- Within each dimension, elements are taken in order of surprise, keeping those whose explanatory power is above a small threshold, until their explanatory power together passes a large threshold. Dimensions are then ranked by the surprise of their chosen set.
- For a ratio metric the paper derives explanatory power from the partial derivative of the ratio, and surprise as the sum of the numerator's and the denominator's.
- `explain(method="recursive")` applies the same step inside each chosen element, to reach cells of more than one dimension, to a depth the user sets (default 2).
- Defaults follow the paper's thresholds. Both are arguments.
- **A third answer is required: "no segment explains it".** When the change is spread evenly over every element, no set passes the thresholds with fewer than a stated number of elements (default 3), and the function says so and does not return the least bad set.

**Gate:** if the paper's formulas for the ratio case cannot be confirmed from the paper itself, implement the additive case only, mark the ratio case as not supported in 0.1.0, and report.

### 4.5 `noise`: statistical

Two checks, each optional.

- `unit="order_id"`: resample units with replacement within each side, 1,000 times by default, and give each contribution an interval. A contribution whose interval contains zero is flagged as within sampling noise.
- `history=[...]`: given a list of earlier pairs of periods, compute each element's contribution in each earlier pair and report where the current contribution falls in that reference distribution. This answers "does this segment move this much every month?".

### 4.6 Outputs

Every result object has `.to_frame()`, `.to_dict()` and a `kind` field that is one of `"exact"`, `"heuristic"` or `"statistical"`. `summary()` writes at most five plain sentences and never uses the word "cause" or "because".

## 5. The benchmark, fixed before it is run

Commit and push the following as `docs/benchmark-plan.md` before `benchmarks/run.py` is executed for the first time. Fill only the date.

> # Benchmark plan, fixed before the first run, [date]
>
> **Generator.** `whatchanged.synthetic.planted(...)` builds a baseline table and a current table over 3 or 5 dimensions, each with 4, 8 or 20 elements, with cell sizes drawn from a Zipf law. A cause is planted in the current table and the planted elements are recorded.
>
> **Scenarios**, 100 seeds each:
>
> 1. `one_element`: one element of one dimension changes its rate.
> 2. `two_elements`: two elements of the same dimension change.
> 3. `one_cell`: one cell of two dimensions changes.
> 4. `mix_shift`: no rate changes anywhere. Volume moves from a high-rate element to a low-rate one, so the overall ratio falls while every element's own rate is unchanged.
> 5. `uniform`: every cell changes by the same relative amount. There is no localised cause.
> 6. `null`: the two tables are two draws from the same distribution.
>
> Each scenario at two noise levels (row-level variance low and high) and two effect sizes (the planted change moves the total by 2% and by 10%).
>
> **Methods.** `top_delta` (rank elements by absolute change of their own metric, the thing a dashboard does), `adtributor`, `adtributor_recursive`, and `decompose` followed by ranking its contributions.
>
> **Scores.** Precision, recall and F1 of the returned elements against the planted ones, per scenario. For `uniform` and `null`, the false alarm rate: the share of runs in which a method names any segment.
>
> **Expectations.**
>
> - B1. On `one_element` at the 10% effect and low noise, every method except `top_delta` has F1 above 0.9.
> - B2. On `mix_shift`, `top_delta` names a wrong element in more than half of the runs, and `decompose` attributes more than 90% of the change to mix.
> - B3. On `one_cell`, `adtributor` (one dimension at a time) has recall below 0.5 and `adtributor_recursive` above 0.7.
> - B4. On `uniform`, `adtributor` names a segment in fewer than 10% of runs.
> - B5. On `null`, with `noise(unit=...)` applied, fewer than 10% of runs keep any flagged segment.
> - B6. At the 2% effect and high noise, no method has F1 above 0.7 on any scenario. Reported as the floor of what the tool can see.
>
> **What counts.** Each expectation is graded from `benchmarks/results.json`. Thresholds of the methods are not tuned after the first run. A failed expectation is reported in the README next to the passed ones, and becomes a documented limit of the method.

**A second, external benchmark, time-boxed to one session.** The `riskloc` repository publishes synthetic datasets used by several papers. Check its licence first. If the licence allows use, run `adtributor` on them and report F1 next to the figures its README gives. If the licence is unclear or the data cannot be fetched, skip this and say so.

## 6. Work packages

### WP0. Scaffold (small)

- Name check (section 2). `pyproject.toml` with `hatchling`, the `src` layout, `py.typed`, MIT licence, version from one place.
- `ruff` for lint and format with the rule set of `uplift-analyzer`, `mypy --strict` on `src`, `pytest` with `hypothesis`.
- `CLAUDE.md` for the repository: prose style, no number in a document without a test, tests need no network, the three kinds of output must never be blurred, no code comments beyond what an API docstring needs. Public functions do carry docstrings, because the documentation is built from them.
- CI (`.github/workflows/ci.yml`): lint, types, tests on Python 3.10 to 3.13, on the oldest and newest supported pandas. Pin action versions by hash as the other repositories do.
- `README.md` stub, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`.

### WP1. Metrics and the cube (small)

- `metrics.py`, `cube.py` (pandas path), `result.py`.
- Tests: a metric computed from cells equals the metric computed from rows. Unknown columns, a zero denominator, an empty side and a `split` value that does not exist each raise an error that names the problem.

### WP2. `decompose` (medium)

- Port the logic of `margin_drivers.py` to the general case of section 4.3.
- **Fidelity test.** A fixture holds the Superstore table aggregated to region, category and subcategory with sales and profit (four regions by 17 subcategories, aggregates only, built once from the committed CSV of the superstore repository). The test must reproduce, for Central against West: a gap of −7.023 percentage points, category mix +0.273, subcategory mix +0.451 and within-product margin −7.747, to three decimals, as published in that repository's `docs/margin-driver-method.md`. **Verify the four numbers against `outputs/margin_drivers.json` in that repository before pinning them.**
- Property tests with `hypothesis`: the contributions sum to the change, swapping baseline and current flips every sign, relabelling elements changes nothing, scaling numerator and denominator by a constant changes nothing, and with identical sides every contribution is zero.

### WP3. Synthetic generator and `noise` (medium)

- `synthetic.py` with the six scenarios of section 5. Deterministic under a seed. Tested: the planted change moves the total by the stated amount within Monte Carlo error, and in `mix_shift` every element's own rate is unchanged.
- `noise.py`. Tests: on `null`, the bootstrap intervals contain zero for at least 90% of elements. On `one_element` at 10%, the planted element is flagged. The placebo check returns a rank between 0 and 1 and is uniform on exchangeable history.

### WP4. `explain` (medium, use the strongest model)

- Read the paper. Write `docs/adtributor-notes.md` with the formulas as implemented, each with its section number in the paper.
- Implement the single-dimension algorithm, then the recursive one, then the "no segment" answer.
- Tests: a hand-worked example with three dimensions small enough to check on paper, the paper's own worked example if it gives one with numbers, and the degenerate cases (one element, zero total change, all change in one element).
- **Gate** as in section 4.4.

### WP5. Commit the benchmark plan, then run it (medium)

1. Commit and push `docs/benchmark-plan.md`. Record the hash.
2. `benchmarks/run.py` writes `benchmarks/results.json` and `benchmarks/results.md`. A CI job reruns a reduced version (10 seeds) and checks it against committed values within a tolerance, so that a later change cannot quietly make a method worse.
3. Grade B1 to B6. Write the failures into the README's "Where it fails" section.
4. The external benchmark, time-boxed.

### WP6. The DuckDB path and the command line (medium)

- `cube.py` accepts a DuckDB connection and a table name, and pushes the aggregation into SQL. Only the cell table comes back. Test: both paths give the same cells on the synthetic tables. Identifiers are quoted and values are passed as parameters, never formatted into the query string.
- `cli.py` with `argparse`. Reads CSV or Parquet. Prints the summary, or JSON with `--json`. Tests call the entry point on a temporary file.

### WP7. Documentation and one real case (medium)

- `mkdocs` with the Material theme, published to GitHub Pages by a workflow. Pages: a five-minute start, the three kinds of answer and what each can and cannot claim, one page per function built from docstrings, the benchmark with its failures, and a comparison with `riskloc` and PyRCA that is fair to both.
- **One real case study, on public domain data.** The monthly on-time performance files of the US Bureau of Transportation Statistics. The question: between two named months, which carriers and origins account for the change in the share of flights arriving at least 15 minutes late? Show `decompose` (how much is mix of carriers, how much is each carrier's own rate), `explain`, and `noise` with the previous twelve month pairs as history.
  - **Gate:** if `transtats.bts.gov` is not reachable from the session, write the case study on the Superstore fixture only and say that the flight case is left for the owner's machine.
  - The case study states that it describes and does not explain why a carrier was late.

### WP8. Release (small, with the owner)

- `.github/workflows/release.yml`: on a tag `v*`, build the sdist and the wheel, check them with `twine check`, install the wheel in a clean environment and run the tests against it, publish to TestPyPI, then to PyPI through trusted publishing, in a protected environment named `pypi`.
- Before the tag: install from TestPyPI in a fresh virtual environment and run the five-minute start from the documentation, word for word.
- The owner creates the tag `v0.1.0`. The executing model does not.
- After the release: `pip install whatchanged` in a fresh environment, run the start again, and record the result in `CHANGELOG.md`.

## 7. Done when

- `pip install whatchanged` works from PyPI and the documented start runs unchanged.
- CI is green on four Python versions. `mypy --strict` passes.
- The Superstore numbers are reproduced under test.
- B1 to B6 are graded, and every failure is in the README.
- The documentation site is live and its code blocks are executed by a test (use `pytest --doctest-glob` or `mktestdocs`).

## 8. Scope control

If time runs short, cut in this order and say so in the changelog: the external benchmark, the DuckDB path, the recursive search, the flight case study. Never cut the exactness tests, the "no segment" answer, or the benchmark's failures.

Version 0.1.0 may ship with `decompose`, single-dimension `explain`, `noise` and the command line. The rest becomes 0.2.0.

## 9. Limits to state in the README

- The exact part is accounting. It says where a difference sits and not why it arose.
- The search is a heuristic from one paper, with thresholds. The benchmark shows the scenarios where it returns the wrong segment.
- Sampling noise is checked. Seasonality, data errors and definition changes are not.
- Dimensions with thousands of elements are slow in 0.1.0 and are not benchmarked.

## 10. What the owner learns

Designing an interface for people who will not read the source, packaging and typing a library, property-based testing, a release pipeline with trusted publishing, and writing documentation that states a tool's failures. It also gives the profile a link that ends in `pip install`.
