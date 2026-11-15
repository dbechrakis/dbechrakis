# Plan 05. A season by hub, as a hierarchical Bayesian model

**Repository:** `dbechrakis/missed-connection-pricing` (existing, the flagship). **Branch:** `claude/hub-season`, from `main`. **Size:** three to four weeks, five sessions. **Deadline:** the pre-registration of WP4 must be pushed before any flight month of March 2027 or later exists in the project. Aim for the end of February 2027.

**Use the strongest model available for this plan.** The repository has frozen files and sealed months. One wrong edit damages the record that makes the project believable.

## 1. Why this exists, and why in this form

The first idea was a Bayesian rival to the miss-probability model. Reading the repository changed that, for three reasons.

1. The miss-probability model is fitted on raw flight rows that are not in the repository. An executing model in a fresh session cannot refit it.
2. The sealed reads of September to November 2026 are already fully pre-registered. Nothing may be added to them.
3. The project already wrote down the open question that a hierarchical model answers. `docs/season3-plan.md`, under "Ideas recorded and not scheduled", says: *a season by hub. Denver and Chicago have winter weather and Atlanta and Dallas have summer storms, so a single calendar pattern may hide hub patterns. With 5 to 7 observations per month and hub it needs strong shrinkage.*

Five to seven observations per cell is exactly where partial pooling earns its place or shows that it does not. And the whole question runs from one committed file, `results/risk_weeks_2019_2026.csv`.

**The question:** does the ratio of realized to expected claims follow a different calendar at each hub, and does a model that says so predict a year it has not seen better than the single frozen calendar?

The project's own history predicts a small gain or none. Its credibility model across hubs (`cred` in `mcp.level`) did not beat a constant level in `VALIDATION.md` 7A. A null result here is a result and is reported as one.

**What this plan does not do.** It changes no price, no multiplier, no frozen file and no existing pre-registration. The 2027 forward test of `PREREGISTRATION_V6_FORWARD.md` stands as written. This plan adds a second prediction for the same months, at hub level, that is scored next to it.

## 2. Read first

- `CLAUDE.md` in full. Every standing rule applies. The ones this plan can break by accident: rule 1 (sealed months), rule 3 (frozen files, new results go in a new section of Part 7), rule 5 (do not change `requirements.lock`), rule 6 (every number comes from a results file and a test checks it), rule 7 (leakage tests come with the feature), rule 9 (house style).
- `VALIDATION.md` 7A (the level models), 7G (the calendar pattern), 7I and 7J (the arms). `docs/season3-plan.md`. `PREREGISTRATION_V6_FORWARD.md` and its addendum, for the format a pre-registration takes here.
- `src/mcp/v6.py`: `monthly_log_ratio`, `fit_season`, `leave_one_year_out`, `season_report`, `read`, `require_scheduled_reads`. `src/mcp/level.py` and `src/mcp/risk.py` for the scoring functions (`crps_normal`, `interval_coverage`). `src/mcp/capital.py` for `without_pandemic`.
- `models/v6_season.json`, the frozen calendar. Twelve multipliers, shrunk by n ÷ (n + 1), fitted on 70 months.

## 3. Dependencies, and how rule 5 is kept

PyMC is a new dependency. `requirements.lock` is not touched.

- Add `requirements-bayes.txt`, following the precedent of `requirements-hist.txt`. Pin exact versions of `pymc`, `arviz` and what they pull in, as resolved on the day. State at the top that it is for `mcp.hub_season_fit` only.
- Fit in a **separate virtual environment** (`.venv-bayes`). Never install PyMC into the environment of `requirements.lock`, because it may move numpy or scipy and `tests/test_frozen_models.py` fails under other versions.
- Split the code in two modules:
  - `src/mcp/hub_season_fit.py` imports PyMC. It fits and writes a posterior summary as JSON.
  - `src/mcp/hub_season.py` uses numpy only. It builds the panel, runs the three numpy baselines, reads the frozen summary and scores. Everything the main test suite touches lives here.
- Main CI stays as it is. Add a second job, `bayes`, to `.github/workflows/ci.yml` that installs `requirements-bayes.txt` and runs `tests/test_hub_season_fit.py`. In the main suite that file is skipped with `pytest.importorskip("pymc")`.

**Gate:** if PyMC cannot be installed in the session, do WP1 and the numpy baselines of WP2, then stop and report. Do not replace the sampler with a hand-written one without the owner's word.

## 4. The model

### 4.1 Data

From `results/risk_weeks_2019_2026.csv` (columns `ym, week, hub, sold, claims, expected, w, premium, source`):

- Sum `claims` and `expected` over weeks and over both values of `sold`, by hub and month. This is the "all overnight-risk connections" view that 7G uses.
- Drop the pandemic months, March 2020 to June 2021, with the existing helper.
- z = log(claims ÷ expected), one value per hub and month. Five hubs (ATL, CLT, DEN, DFW, ORD), about 70 months, about 350 values.

**Reconciliation, before anything else.** Summing the panel over hubs and applying the repository's own `fit_season` must reproduce the twelve `effects_log` values of `models/v6_season.json` to six decimals. A test pins this. If it does not reconcile, stop and find out why. Do not go on with a panel that disagrees with the frozen calendar.

### 4.2 Structure

For hub h, month t, calendar month m(t):

> z[h, t] = μ + a[h] + s[m] + d[h, m] + e[t] + ε[h, t]

| Term | Meaning | Prior |
|---|---|---|
| μ | Overall level | Normal(0, 0.3) |
| a[h] | A hub's own level | Normal(0, τ_a), τ_a ~ HalfNormal(0.2) |
| s[m] | The calendar pattern all hubs share, twelve values that sum to zero | Normal(0, τ_s), τ_s ~ HalfNormal(0.3) |
| d[h, m] | A hub's departure from the shared calendar, summing to zero within a hub | Normal(0, τ_d), τ_d ~ HalfNormal(0.1) |
| e[t] | A shock that hits all hubs in the same month | Normal(0, τ_e), τ_e ~ HalfNormal(0.2) |
| ε[h, t] | What is left | Student t with ν degrees of freedom and scale σ, ν ~ Gamma(2, 0.1), σ ~ HalfNormal(0.3) |

- **τ_d is the quantity of interest.** If the data support τ_d near zero, the hubs share one calendar and the frozen multipliers are enough.
- **e[t] must be in the model.** Hubs are hit by the same storms and the same holiday peaks. Without the shared shock the model treats 350 values as independent and its intervals are far too narrow. For a month not yet seen, e is drawn fresh from its prior scale.
- Use the non-centred form for a, s, d and e, so that the sampler does not stall when a scale is near zero.
- The prior scales are on the log ratio. A value of 0.1 means about 10%. They were chosen from the size of the effects in 7G (the largest shared effect is about 0.3) and are fixed here. WP2 reports sensitivity to τ_d's prior scale at 0.05 and 0.2, as a table, and does not choose among them.

### 4.3 Sampling settings, fixed

Four chains, 2,000 tuning draws, 2,000 kept draws each, `target_accept` 0.95, seed 20261115. Accepted only if every R-hat is below 1.01, every bulk and tail effective sample size is above 400, and there are no divergences.

**Gate:** if a fit fails these checks, raise `target_accept` to 0.99 once. If it still fails, stop and report the diagnostics. Do not change a prior to make the sampler behave.

### 4.4 What a fit produces

`hub_season_fit` writes one JSON per fit with: the posterior mean and the 5%, 50% and 95% quantiles of every multiplier exp(μ + a[h] + s[m] + d[h, m]), the same for the scales, and for prediction the mean and standard deviation of the predictive distribution of z for each hub and calendar month of a new year (which includes a fresh e and ε). The numpy scorer treats each predictive as normal with that mean and standard deviation, as `mcp.level` does. Also written: the diagnostics, the versions, the seed and the SHA-256 of the input file.

## 5. Candidates and the comparison

All four predict z for each hub and month of a held-out year from the other years.

| Candidate | What it predicts |
|---|---|
| `none` | z = 0. The price as it is without a season. |
| `pooled` | The frozen rule: one calendar for all hubs, month deviations shrunk by n ÷ (n + 1), refitted on the other years with the repository's `fit_season`. Its predictive spread is the root mean squared residual of the hub panel in the fitting years. |
| `by_hub` | Each hub's own twelve monthly means, no pooling across hubs, the same n ÷ (n + 1) shrinkage within the hub. |
| `hier` | The model of section 4. |

**Leave one year out.** For each calendar year with at least 10 observed months (the helper `yearly_loss_ratios` uses the same floor), fit on all other years and predict that year's hub-months. Eight years are in the file. 2020 and 2021 lose most months to the pandemic rule, so expect six or seven folds. Report the count.

**Scores**, in log-ratio units, lower is better, pooled over folds and also per fold:

- Mean absolute error of the hub-month prediction.
- Continuous ranked probability score with `mcp.risk.crps_normal`.
- Coverage of the 90% interval.

**The rule for "earns its place", fixed here before any fit.** `hier` earns its place over `pooled` if its ranked probability score is lower in at least all folds but one **and** the mean gain is above 0.005. Otherwise the reading is that one calendar is enough, and that sentence is written into `VALIDATION.md` and the README. The same rule is applied to `by_hub` against `pooled`, which shows what pooling is worth.

Everything in this section is run on months the project has already read. It is exploratory and is labelled so in every heading, as 7G is.

## 6. Work packages

### WP0. Orientation (small)

- Fresh branch `claude/hub-season` from `main`. `pip install -r requirements.lock` in `.venv`, run `python -m pytest -q`, record the count.
- Find the next free letter of `VALIDATION.md` Part 7 (7N was the last on 15 November 2026). This plan's sections are called 7X below. Replace X with that letter.
- Read `docs/season3-plan.md` to its end and confirm that no later task has claimed the hub season. If one has, stop and report.

### WP1. The panel and the numpy baselines (small)

- `src/mcp/hub_season.py`: `hub_month_panel(weeks)`, `candidate_none`, `candidate_pooled`, `candidate_by_hub`, `leave_one_year_out_panel(panel, candidate)`, `score(predictions, panel)`, `earns_its_place(scores_a, scores_b, min_gain=0.005)`. CLI: `python -m mcp.hub_season dev` writes `results/hub_season_dev.json` with the three numpy candidates.
- `tests/test_hub_season.py`, on hand-built panels:
  1. The reconciliation of section 4.1 against `models/v6_season.json`.
  2. On a panel built with one shared calendar and no hub departures, `pooled` beats `by_hub`. On a panel built with strong hub departures, `by_hub` beats `pooled`.
  3. **Leakage:** changing any value of the held-out year cannot change that year's prediction, for every candidate.
  4. `earns_its_place` on three hand-written score tables.
- The suite stays green under `requirements.lock` and needs no data beyond the committed CSV.

### WP2. The hierarchical model, on the record (medium)

- `src/mcp/hub_season_fit.py`: `build_model(panel)`, `fit(panel, seed)`, `summarise(trace)`, `fit_fold(panel, held_out_year)`. CLI: `python -m mcp.hub_season_fit dev` runs every fold and the full-record fit, and writes `results/hub_season_fits/<fold>.json` and `results/hub_season_dev.json` (now with all four candidates, the sensitivity table and the "earns its place" verdicts).
- **Prior predictive check first.** Before any fit on data, draw 1,000 panels from the priors and record the 5% to 95% range of the simulated z. It should span roughly minus 1 to plus 1. If it spans far more, the priors say something absurd about claims and the executing model stops and reports. The check's summary goes in the results file.
- `tests/test_hub_season_fit.py` (skipped without PyMC), on synthetic panels of the same shape, with short chains to keep the job under ten minutes:
  1. With hub departures of scale 0.2 built in, the posterior median of τ_d is between 0.1 and 0.3, and the model's hub-month multipliers correlate above 0.8 with the truth.
  2. With no hub departures built in, the posterior median of τ_d is below 0.05, and the predictions are within 0.03 of `pooled`.
  3. With a shared monthly shock built in, the 90% predictive interval covers between 80% and 97% of a simulated new year. Without e[t] in the model the coverage falls below 75%. This test is the reason e[t] exists and must not be dropped.
  4. The summary JSON round-trips through the numpy scorer.
- Write `VALIDATION.md` 7X, "A season by hub (exploratory, on months already seen)": the table of four candidates, the verdict by the fixed rule, the posterior of τ_d with its interval, the sensitivity to its prior, the hub-months whose multiplier differs from the shared calendar with posterior probability above 0.9 (there may be none), and the limits. Pin every number with `tests/test_docs.py` style checks.
- Add a row to the status of `docs/season3-plan.md` saying the idea was taken up and where.

### WP3. Freeze the forward prediction (small)

- Fit once on the full record with the fixed seed. Write `models/v7_hub_season.json`: for each hub and each month of March to August, the predictive mean and standard deviation of z, the posterior probability that the hub's multiplier is above the frozen pooled multiplier of `models/v6_season.json`, the scales, the diagnostics and the input hash.
- Record its SHA-256. From this commit on, the file is frozen like every other file in `models/`.
- Add `tests/test_hub_season.py::test_frozen_summary_hash`.

### WP4. Pre-register the hub-level forward test (small, and the judgment step)

**Owner decides.** Do not push this document without the owner's explicit yes. Draft it, show it, and stop.

Draft `PREREGISTRATION_V7_HUB_SEASON.md` in the format of `PREREGISTRATION_V6_FORWARD_ADDENDUM.md`. Its fixed content:

- **What was seen.** Every month to August 2026, and the result of 7X. No month from September 2026 on.
- **What is predicted.** For the 30 hub-months of March to August 2027, the log ratio of realized to expected claims at hub level, under two frozen predictors: `pooled` (the multipliers of `models/v6_season.json`, the same for every hub) and `hier` (`models/v7_hub_season.json`).
- **The data.** The same rows, sample, model and cap as the two v6 windows. Nothing new is ingested. The ratio is computed per hub and month from the rows the v6 read has already built.
- **Expectations.** Their direction depends on the verdict of 7X, and the rule for writing them is fixed here so that the executing model does not choose:
  - K1. If `hier` earned its place in 7X: "`hier` has a lower mean absolute error than `pooled` over the 30 hub-months." If it did not: "The mean absolute errors of `hier` and `pooled` differ by less than 0.02."
  - K2. The 90% predictive intervals of `hier` cover between 21 and 30 of the 30 hub-months. The band is wide because the six months share shocks.
  - K3. For each hub-month that 7X listed with posterior probability above 0.9, the realized ratio falls on the predicted side of the pooled multiplier in at least two thirds of them. If 7X listed none, K3 is "no prediction" and says so.
  - K4. No prediction on whether any hub-level pattern would have changed a price. Reported.
- **Order guard.** The read of this test runs only after `mcp.v6 read` has written its result for the window. It never reads a window before v6 has.
- **Not allowed after the read.** Refitting, changing the predictors, the months, the hubs or the bands.

After the owner's yes: commit the document alone, push, add its row to `.github/workflows/preregistration-tags.yml` (tag `prereg-v7-hub-season`), and add K1 to K4 to `SCOREBOARD.md` as pending and to `src/mcp/grade.py` with thresholds copied from the document.

**Before the push, a second model reviews** the document against `models/v7_hub_season.json` and against 7X, and checks that no number in it was typed by hand.

### WP5. The read command, for the owner's machine (small)

- `python -m mcp.hub_season read spring2027` and `summer2027`. It calls the same row builder as `mcp.v6 run`, groups by month and hub, scores both predictors and writes `results/v7_hub_season_<window>.json`. It refuses to run unless `results/v6_forward_<window>.json` exists, and refuses a second run. Check first that the rows carry the hub. If they do not, stop and report, because adding it would change code the v6 read depends on.
- Tests on a hand-built table: the grouping, the scores, the two refusals.
- The read itself is the owner's, in about August and November 2027, right after each v6 read. An executing model in a fresh session has no raw data and must not attempt it.

### WP6. Tell it (small)

- `README.md`: two or three sentences under the findings, stating the verdict of 7X in the same plain register as the rest, and one line in the list of pending forward tests. Numbers checked by `tests/test_docs.py`.
- `docs/decisions.md`: one row, "A season by hub: tested with partial pooling, kept or not kept, and why".
- The profile page is not edited by this plan.

## 7. Done when

- `python -m pytest -q` is green under `requirements.lock` with the same count as before plus the new tests. The `bayes` job is green.
- `git diff --stat main -- models/pmiss*.joblib models/v6_season.json config requirements.lock 'PREREGISTRATION_*.md' ':!PREREGISTRATION_V7_HUB_SEASON.md'` is empty, and Parts 1 to 6 and sections 7A to 7N of `VALIDATION.md` are unchanged.
- 7X states the verdict by the fixed rule, whichever way it went.
- The pre-registration is either pushed and tagged with the owner's yes, or drafted and waiting, with that state reported.

## 8. Limits to state

- About 350 values from 70 months. Months are shared by all hubs, so the effective number of independent observations is nearer 70.
- The record joins two fit windows (the 2019 to 2023 backcast and 2024 to 2026), and 7I showed that the backcast arm matters. The panel uses the file as committed, which is arm A for the backcast years. A robustness line repeats the comparison with the arm C weeks of `results/backcast_weeks_C.csv` and reports whether the verdict changes.
- The forward test has 30 hub-months in one spring and summer. It can contradict the model. It cannot confirm it.
- A hub-level price has no demand evidence behind it and is not proposed.

## 9. What the owner learns

Building a hierarchical model in PyMC, the non-centred form, prior predictive checks, sampler diagnostics, posterior predictive scoring, and the discipline of freezing a posterior as an artefact. Also the experience of asking a question where the honest answer may be "the simple version was enough".
