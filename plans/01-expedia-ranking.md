# Plan 01. What a position is worth: offer ranking, position bias and offline evaluation

**Repository:** `dbechrakis/offer-ranking-position-bias` (new, the owner creates it empty and public). **Branch:** `claude/build`. **Size:** five to seven weeks, nine sessions.

**Use the strongest model available for work packages 3 to 6.** Errors in these estimators are silent. A wrong weight still returns a number.

## 1. Why this exists

A travel site decides, on every page, which offer goes first. The offer shown first is clicked more because it is first, not only because it is better. Anyone who learns a ranking from past clicks, or values a new ranking from past clicks, has to separate those two things. This is the daily problem of an ancillary sales team (which add-on, in which slot, at which price), and no project in the portfolio touches it.

One public dataset allows the separation to be checked against the truth. Expedia released hotel search logs in which, for roughly a third of searches, the order of hotels was random. In those searches a hotel's position owes nothing to its quality. That gives three things that are rare in public data:

1. **The true effect of position**, measured directly.
2. **A test of the methods people use when they have no randomized data.** Each method is run on the non-random searches and compared with the truth.
3. **A test of offline policy evaluation.** The value of a new ranking is estimated from logs, and compared with a value read directly from the searches where chance happened to show that ranking's first choice first.

**The questions, in the order the project answers them.**

1. How much of a click is the position, and how much is the hotel?
2. Which of the usual corrections recovers that from ordinary logs, and how far off is each?
3. What is one position worth, expressed as a price change a traveller would trade for it?
4. Can the value of a new ranking be estimated from logs well enough to decide before an A/B test?

The honest possibilities include: the corrections recover the truth poorly, and the offline estimate disagrees with the direct one. Each is a finding and is reported as one.

## 2. Owner actions

| Action | When |
|---|---|
| Create the empty public repository `dbechrakis/offer-ranking-position-bias` and attach it to the session | Before WP0 |
| On Kaggle, join the competition **"Personalize Expedia Hotel Searches - ICDM 2013"** (`expedia-personalized-sort`) and accept its rules | Before WP1 |
| Make `train.csv` available to the session. Either download it and place it at `data/raw/train.csv`, or provide Kaggle credentials through the environment's secret settings. Never paste a token into chat or commit one | Before WP1 |
| Read the competition's data rules and confirm that publishing aggregate results is allowed | Before WP1 |
| Say yes to the pre-registration before the sealed read | WP6 |

**The data is never committed.** Not the raw file, not a sample of rows, not a table from which a row can be rebuilt. Only aggregates over at least 100 searches leave the machine. `.gitignore` covers `data/`. A test fails if any file under version control has a column named `srch_id` or `prop_id`.

## 3. The data, as expected

The file could not be downloaded while this plan was written, because it needs the owner's account. Everything in this section is an expectation from the competition's description and from papers that use the data. **WP1 verifies each line and reports any difference before any analysis.**

- `train.csv`: about 9.9 million rows, one per hotel shown in a search, about 400,000 searches, 54 columns. `test.csv` has no outcomes and is not used.
- Keys: `srch_id` (a search), `prop_id` (a hotel).
- Outcomes: `click_bool`, `booking_bool`, `gross_bookings_usd`.
- Design: `position` (where the hotel was shown), `random_bool` (1 when the order was random for that search). About 30% of searches are random.
- Hotel: `prop_starrating`, `prop_review_score`, `prop_brand_bool`, `prop_location_score1`, `prop_location_score2`, `prop_log_historical_price`, `price_usd`, `promotion_flag`, `prop_country_id`.
- Search: `date_time`, `site_id`, `visitor_location_country_id`, `srch_destination_id`, `srch_length_of_stay`, `srch_booking_window`, `srch_adults_count`, `srch_children_count`, `srch_room_count`, `srch_saturday_night_bool`, `srch_query_affinity_score`, `orig_destination_distance`, the visitor's history (`visitor_hist_starrating`, `visitor_hist_adr_usd`, mostly missing).
- Competitor fields `comp1_rate` to `comp8_rate_percent_diff`, mostly missing.

**Known hazards, each to be checked and written down in WP1.**

1. **How searches were sampled.** The release may hold only searches with at least one click, and bookings may be over-represented in the non-random searches. If so, click and booking rates are not those of the site, and the random and non-random searches are not two arms of one experiment. The plan's main comparisons do not need them to be. The ones that would are marked.
2. **`price_usd` is not one thing.** In some countries it is per night and in others for the whole stay, with or without taxes. The plan uses the logarithm of price inside a search, where a constant factor cancels. Absurd values (zero, or above 10,000) are dropped with their whole search.
3. **Positions may have gaps.** Some slots held sponsored or opaque offers that are not in the file. Lists differ in length.
4. **"Random" may not be perfectly random.** The plan tests it instead of assuming it, as `uplift-analyzer` did for its randomized test.
5. **Missing values** are heavy in review scores, location score 2, distance and visitor history. Missing is kept as its own indicator. Nothing is imputed with a mean.

## 4. Standing rules for this repository

Write these into `CLAUDE.md` in WP0.

1. **The sealed part.** Searches are split once by a hash of `srch_id` into DEV (60%) and SEALED (40%). They are written to separate directories. Only `rank.sealed` may open the sealed directory, it runs once, and it refuses a second run. A test scans the source tree and fails if any other module names the sealed path.
2. **Pre-register before the sealed read.** `PREREGISTRATION.md` is committed, pushed and tagged first.
3. **Frozen after the pre-registration:** every estimator's code path used by `rank.sealed`, `config/frozen.yaml`, and the document itself. A bug found later is fixed with the numbers before and after, in `VALIDATION.md`.
4. **Every number in a document comes from a file in `results/`, and a test checks it.**
5. **Tests need no data.** `rank.synthetic` generates searches with a known position effect, known tastes and a known logging policy.
6. **Every estimator ships with a recovery test** on synthetic data, and with a test that shows the naive version is wrong on the same data.
7. **Report failures as prominently as passes**, in the same commit as the result.
8. **Prose:** plain sentences, full stops in place of dashes and semicolons, no number typed by hand.
9. **No row-level data in git**, as in section 2.

**What sealing means here, stated honestly in the README.** The whole file is on the analyst's disk from the first day. Sealing is enforced by code and by the commit history, not by the data not existing yet. It protects against tuning methods on the part they are judged on. It is weaker than the flagship project's sealing, where the months had not happened.

## 5. Layout

```
src/rank/
  ingest.py        CSV to Parquet with DuckDB, the split, the audit
  synthetic.py     searches with known truth
  design.py        randomization checks
  position.py      the position curve from random searches
  observational.py naive, adjusted and click-model estimates from non-random searches
  choice.py        conditional logit, position worth
  rankers.py       the ranking policies
  ope.py           replay, position-based weighting, direct method, doubly robust
  sealed.py        the single read
  grade.py         grades the pre-registration from results
  demo.py          builds the static page from results
tests/  results/  config/  docs/  site/
PREREGISTRATION.md  SCOREBOARD.md  VALIDATION.md  CLAUDE.md  README.md
```

Python 3.12. Dependencies, pinned in `requirements.lock`: `duckdb`, `numpy`, `pandas`, `pyarrow`, `scipy`, `scikit-learn`, `lightgbm`, `matplotlib`, `pytest`, `ruff`. Nothing else without a reason written in `docs/decisions.md`.

## 6. Methods

Notation. A search s shows hotels i at positions k. c is a click, b a booking. θ_k is the chance that position k is looked at, relative to position 1.

### 6.1 Randomization checks (`design.py`)

- **Within random searches, is position independent of the hotel?** Fit a gradient boosting classifier of "shown in the top five" on the hotel's features, with searches as groups in cross-validation. Under randomization the area under the curve is 0.5. Compare with the same classifier on non-random searches, where it should be far above 0.5. Give the random value an interval by permuting positions within searches 200 times.
- **Are random and non-random searches the same kind of search?** A classifier of `random_bool` on search-level features. An area well above 0.5 means they differ, and any comparison of outcome rates between them is not a clean experiment.
- **Reading.** If position is predictable in random searches by more than the permutation interval, every estimate that uses them is reported twice: as is, and with inverse weights from the classifier.

### 6.2 The position curve (`position.py`)

From random searches: click rate and booking rate by position, for positions 1 to 40, with intervals from a bootstrap over searches. θ_k is the click rate at k divided by the click rate at 1. This is the truth that section 6.3 is judged against.

### 6.3 Estimating the curve without randomization (`observational.py`)

Each method sees only non-random searches and returns θ_k for k from 1 to 10.

| Method | What it does |
|---|---|
| `naive` | Click rate by position, divided by the rate at position 1. Confounded, because the ranker puts better hotels first. |
| `adjusted` | A gradient boosting model of click on position and all hotel and search features. θ_k is the average prediction with every row set to position k, divided by the same at position 1. |
| `click_model` | The position-based click model: a click happens when the position is looked at and the hotel is relevant. Fitted by expectation maximisation, with relevance given by a gradient boosting model of the features that is refitted at each step (the "regression EM" of Wang and others, WSDM 2018). |
| `swap_pairs` | Uses the fact that the same hotel appears at different positions in different searches for the same destination. For pairs of adjacent positions, compares click rates of the same hotel and destination when shown at k and at k+1, and chains the ratios. |

**Score:** the mean absolute error of log θ_k over k from 2 to 10 against the randomized truth, with a bootstrap interval over searches.

The executing model must read the regression EM paper before writing `click_model`. If its identification conditions are not met by this data, report that and keep the other three.

### 6.4 What a position is worth (`choice.py`)

A conditional logit on random searches. The traveller chooses among the hotels shown and an outside option of not booking.

> utility of hotel i in search s = β · log(price) + γ · features + δ(position) + m_i

- **Outcome.** Booking, when DEV random searches hold at least 2,000 bookings. Otherwise the first click, and the README says so. **Gate:** count first, then choose, and write the choice into `config/frozen.yaml` before fitting.
- **Position term.** One coefficient per position for 1 to 10, and a linear term beyond.
- **Price.** The logarithm of price. A constant factor within a search cancels, which handles the per-night and per-stay problem.
- **Price is not random.** A hotel that is good in ways the file does not record charges more and is chosen more, which pulls β toward zero. Two controls: `prop_log_historical_price`, and m_i, the hotel's own average log price over DEV (the Mundlak device), so that β is identified from a hotel's price moving over time. The result is reported with and without them. The limit is stated: this is not an experiment on price.
- **The quantity.** The price change, in percent, that moves utility as much as one position does: δ(k+1) − δ(k) divided by β. Reported for the step from 1 to 2, the average step in the top five, and the average step from 6 to 10, each with a bootstrap interval. Also in dollars at the median price, labelled as an illustration.
- **An outside reference.** Ursu (Marketing Science, 2018) studies the same field experiment with a search model and reports a position effect of about two dollars. The versions of that paper differ, so the executing model reads the published one and quotes it exactly. It is context and not a target.
- **Implementation.** Write the likelihood and its gradient in numpy and optimise with `scipy`. No choice modelling package. Test on synthetic searches with known β, γ and δ.

### 6.5 The ranking policies (`rankers.py`)

| Policy | How it ranks |
|---|---|
| `random` | The logged order in random searches. |
| `expedia` | The logged order in non-random searches. It cannot be evaluated offline, because the file does not say how Expedia would have ordered a random search. It appears only in the comparison of samples at the end of 6.6. |
| `price` | Cheapest first. |
| `quality` | Highest review score first, ties broken by star rating, then by lower price. |
| `learned_random` | LightGBM with the LambdaRank objective, trained on DEV random searches. Labels: 5 for a booking, 1 for a click, 0 otherwise. Position is not a feature. |
| `learned_raw` | The same, trained on DEV non-random searches as they are. It learns the position bias as if it were quality. |
| `learned_debiased` | The same, trained on DEV non-random searches with each click weighted by 1 ÷ θ_k, where θ comes from the best method of 6.3 on DEV. Weights capped at 20. |

Settings fixed in `config/frozen.yaml` before training: 500 trees, learning rate 0.05, 63 leaves, minimum 100 rows per leaf, searches as groups. No tuning on the outcome of the evaluation.

### 6.6 Offline evaluation (`ope.py`)

All evaluation happens on random searches, where every hotel had the same chance of every position.

**The target with a ground truth.** V1(π) is the click rate (and the booking rate) of the hotel that policy π would place first, when it is shown first.

- **`replay`, the truth.** Keep the random searches in which the hotel logged at position 1 is the one π would put first. Average the outcome of that hotel. With about 25 hotels per search this keeps about one search in 25. It is unbiased under randomization and uses no model.
- **`pbw`, position-based weighting.** For every random search, take π's first choice wherever it was shown, and weight its click by θ_1 ÷ θ_k. This uses every search. It is right only if a click factors into position and hotel.
- **`direct`.** The choice model of 6.4 predicts the outcome of π's first choice at position 1.
- **`dr`.** The direct prediction plus the weighted residual.

**The test:** does each of `pbw`, `direct` and `dr` agree with `replay`, for each of the five policies that can be computed on a random search (`price`, `quality`, `learned_random`, `learned_raw`, `learned_debiased`)?

**The decision number, with no ground truth.** Expected clicks and bookings per 1,000 searches under each policy's whole list, by `pbw` and by `direct`. This rests on the click model and is labelled so. It is reported with the result of the test above next to it, so that a reader knows how far to trust it.

**One clean comparison if the design allows.** If section 6.1 finds random and non-random searches alike, the difference in booking rate between them is the measured value of Expedia's ranker over a random order. If they differ, this is reported as a contrast between two samples and not as an effect.

### 6.7 The synthetic generator (`synthetic.py`)

Searches with: a known θ, hotel features, a known utility, a logging policy that is either random or sorts by a noisy version of true utility, and clicks from the position-based model. Options to break the model on purpose (clicks that depend on the hotel above). Every estimator's recovery test and every "naive is wrong" test uses it.

## 7. Work packages

### WP0. Scaffold (small)

- The layout of section 5, `CLAUDE.md` with section 4, `requirements.lock`, `pytest.ini`, `ruff`.
- CI: lint, tests without data, the test that no row-level column is in git.
- `.github/workflows/preregistration-tags.yml`, copied in shape from the pricing repository.
- `README.md` stub that states the question and that nothing has been run.

### WP1. Ingest, split and audit (medium)

- `rank.ingest`: read `data/raw/train.csv` with DuckDB, record its SHA-256 and row count, cast types, and write Parquet partitioned by split under `data/parquet/dev/` and `data/parquet/sealed/`. The split is `sha256("expedia-v1:" + srch_id)` taken modulo 100, with 0 to 59 as DEV.
- **The audit reads DEV only** and writes `results/audit.json`: rows, searches, hotels, share of random searches, list length, the positions present and their gaps, click and booking rates by `random_bool`, the share of searches with no click, missing shares per column, the range of `price_usd`, and the number of bookings in DEV random searches.
- For the sealed part, write only its row count and search count.
- Write `VALIDATION.md` Part 1: section 3 of this plan, line by line, marked confirmed or corrected.
- **Gates.** Stop and report if: the file's columns differ from section 3, fewer than 20% of searches are random, or DEV random searches hold fewer than 30,000 searches.

### WP2. The design checks and the truth (medium)

- `rank.design` and `rank.position` on DEV. Results in `results/design_dev.json` and `results/position_dev.json`.
- Tests on synthetic data: the check passes on random logging and fails on sorted logging. The position curve recovers a known θ within its interval.
- `VALIDATION.md` Part 2. One figure: click rate by position, random against non-random.

### WP3. Estimating the curve from ordinary logs (large)

- `rank.observational`, four methods, on DEV non-random searches, scored against the DEV truth of WP2.
- Tests on synthetic data, for each method: recovery when its assumption holds, and the size of its error when the logging policy sorts by quality. `naive` must be shown wrong.
- Tuning is allowed here, on DEV only. Every setting that is finally used goes into `config/frozen.yaml`.
- `VALIDATION.md` Part 3, with the table of errors.

### WP4. The choice model (large)

- `rank.choice` on DEV random searches. The gate of 6.4 first.
- Tests: recovery of known coefficients, the gradient against finite differences, and a case where an unrecorded quality pulls β toward zero and the hotel-average control reduces the pull.
- `results/choice_dev.json`, `VALIDATION.md` Part 4.

### WP5. Policies and offline evaluation (large)

- `rank.rankers` builds the two rule policies and trains the three learned policies on DEV. Split DEV once more by the same hash (0 to 39 to train the policies, 40 to 59 to evaluate them), so that no policy is evaluated on searches it learned from.
- `rank.ope` on the evaluation part of DEV random searches.
- Tests: `replay` is unbiased on synthetic random logs. `pbw` agrees with `replay` when clicks follow the position-based model and disagrees when the generator breaks it. `dr` is right when either the weights or the model is right.
- `results/ope_dev.json`, `VALIDATION.md` Part 5.
- **Gate:** if `replay` keeps fewer than 1,500 searches for a policy on the DEV evaluation part, its interval is too wide to test anything. Report the counts. The owner decides whether to continue with clicks only.

### WP6. Pre-register, then the single sealed read (medium, the judgment step)

**Owner decides. A second model reviews the document before the push.**

1. Freeze: tag the commit, write the hash of `config/frozen.yaml` and of each trained policy into `PREREGISTRATION.md`.
2. Write `PREREGISTRATION.md` from the template of section 8. Every band is filled by the rule given there from DEV results files. No band is chosen by hand.
3. The owner says yes. Commit the document alone, push, run the tag workflow (`prereg-sealed`).
4. Run `python -m rank.sealed` once. It applies the frozen code to the sealed part and writes `results/sealed/*.json` and a marker that blocks a second run.
5. `python -m rank.grade` fills `SCOREBOARD.md`.
6. `VALIDATION.md` Part 6, in the same commit as the results, failures first.

### WP7. Tell it (medium)

- `README.md`: "Decision in 60 seconds" as in the other repositories, then the four questions with one number and one limit each, the scoreboard, and how to reproduce.
- `docs/decision-memo.md`, two pages, for a commercial reader: what a slot is worth, which correction to use on ordinary logs and how wrong it was here, when an offline estimate of a new ordering can replace an A/B test and when it cannot, and what an ancillary team would have to log to do this on its own shelf (the position shown, and a small share of random orderings).
- `docs/method.md`: each estimator in a paragraph, with its reference and its assumption.
- Numbers in all documents pinned by tests.

### WP8. The demo (small)

- `rank.demo` builds `site/index.html` from `results/` only: the position curve with the truth and each method, a slider that turns a position change into an equivalent price change, and the policy table with the direct and the offline estimates side by side. Static, no server, GitHub Pages workflow.
- A screenshot for the profile page at `docs/portfolio-overview.jpg`.

## 8. The pre-registration template

Used in WP6. Text in brackets is filled from the named DEV results file by the stated rule.

> # Pre-registration of the sealed read
>
> Written [date], after every DEV result in `VALIDATION.md` Parts 1 to 5 and before any row of the sealed part was read by anything except the row count of WP1. Frozen: `config/frozen.yaml` ([hash]), the three learned policies ([hashes]), the code at tag [tag].
>
> **What the read does.** Applies the frozen estimators to the sealed 40% of searches. Policies are not retrained. The position curve used for weighting is the DEV one.
>
> **Expectations.**
>
> - S1. **Randomization.** In sealed random searches, the area under the curve of "shown in the top five" from hotel features is within 0.02 of [DEV value].
> - S2. **The naive curve is too steep.** From sealed non-random searches, the naive θ_k is below the sealed randomized θ_k at every k from 2 to 10.
> - S3. **The best correction stays the best.** [The method with the lowest DEV error] has the lowest error on the sealed part among the four.
> - S4. **And it is a real improvement.** Its sealed error is below [half of the naive DEV error, as a number].
> - S5. **The worth of a position replicates.** The sealed estimate of the price change equivalent to the step from position 1 to 2 lies inside [the DEV 95% bootstrap interval, widened by half its width on each side].
> - S6. **Offline evaluation agrees with the direct read.** For at least four of the five policies that can be evaluated offline (`price`, `quality`, `learned_random`, `learned_raw`, `learned_debiased`), the `pbw` estimate of V1 for clicks lies inside the 95% interval of `replay` on the sealed part.
> - S7. **The same for the model-based estimate.** As S6, for `direct`. [If on DEV `direct` was inside for fewer than four policies, S7 reads instead: "`direct` lies outside the interval for at least two policies", which predicts its failure.]
> - S8. **De-biasing helps.** [If on DEV the `replay` value of `learned_debiased` exceeded that of `learned_raw` by more than one standard error: "`learned_debiased` has a higher sealed `replay` value than `learned_raw`." Otherwise: "No prediction. Reported."]
> - S9. **The order of policies.** The two policies with the highest sealed `replay` value for clicks are [the two highest on DEV], in either order.
> - S10. No prediction on bookings per 1,000 searches under whole lists. Reported with the outcome of S6 and S7 beside it.
>
> **Grading.** `rank.grade` grades each from `results/sealed/` with thresholds copied from this document.
>
> **Not allowed after the read.** Retraining, changing a setting, a second run, dropping a policy or a position, or re-reading an expectation in a kinder way. A bug is fixed with the numbers before and after.
>
> **Not claimed.** That 2013 hotel search behaves like any shelf today. That the price coefficient is causal. That the sealed part was unseen in the sense of not existing.

## 9. Done when

- The test suite is green with no data present.
- `SCOREBOARD.md` shows S1 to S10 graded, and `VALIDATION.md` Part 6 was committed with the results.
- No file in git holds a row of the source.
- The README answers the four questions, each with a number, an interval and a limit.
- The demo is live.

## 10. Session plan

| Session | Work |
|---|---|
| 1 | WP0, and WP1 once the file is there |
| 2 | WP2 |
| 3 and 4 | WP3 |
| 5 | WP4 |
| 6 and 7 | WP5 |
| 8 | WP6, with review |
| 9 | WP7 and WP8 |

Each session ends with the suite green, the results files committed, and one paragraph at the top of `VALIDATION.md` saying what is done and what is next.

## 11. Limits to state in the README

- Hotel searches from 2012 and 2013 on one site. The method transfers. The numbers do not.
- The release was sampled in ways that are only partly documented. Rates are not the site's rates.
- The ground truth for policy value covers the first slot only. Whole-list values rest on a click model that the first-slot test can contradict and cannot prove.
- Price was not randomized. The worth of a position is expressed against a price coefficient that is likely too small, which makes a position look worth more.
- Sealing is by code and history on data that was on disk throughout.

## 12. What the owner learns

Click models and position bias, conditional logit written by hand, learning to rank, counterfactual learning from biased logs, and off-policy evaluation with a ground truth. This is the method set of marketplace, search and ancillary merchandising teams, and it is the largest single step away from what the portfolio already shows.
