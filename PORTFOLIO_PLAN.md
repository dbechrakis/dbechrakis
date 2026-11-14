# Portfolio review and plan

Written 14 November 2026. This file lives on branch `claude/tender-hypatia-z5ndyk` only. It is a working document and should not be merged into `main`, because `main` is the public profile page.

Scope of the review. For each repository the reviewer read the README, the validation record's size and place, the layout, the dependency files, the lint and test setup and the workflow list, at the head of the default branch on 14 November 2026. The flagship was read in full and re-computed in part (its own plan is `docs/season3-plan.md` in that repository). The code of the other eight repositories was not executed in this review. A task that depends on a behaviour the reviewer did not run says "verify first".

Part 1 is what the review found. Part 2 is the task list. Each task states the files, what done means, where to stop, and whether something must be fixed in writing before data is looked at.

## Status, 14 November 2026

The pull requests of the six repositories were opened and merged on 14 November 2026 with your approval, after their checks passed. The profile page was merged from a branch that holds only `README.md`. This file stays on its own branch. Not redeployed: the hosted hotel app, because the Hugging Face deploy job was skipped on the merge.

| Task | State | Where |
|---|---|---|
| P1 profile page | Done on this branch. Decision: Superstore stays featured and AAPL stays in More work | `README.md` |
| U1 corrected Qini | Done. The visit coefficient falls from 531 to 377 and the conversion interval now includes zero | uplift-analyzer, `core/dr_scores.py`, `VALIDATION.md` |
| U2 policy value on unseen users | Not done. The existing split-half check (100 halvings, about 20% loss) already answers most of it | |
| U3 three real-model evaluation runs | Not done. Needs the local model server | |
| H1 rolling backtest | Done. Recall at the threshold runs 66% to 90% across four cutoffs | hotel-booking-cancellation-ml |
| H2 `country` | Done as you decided. Removed from the deployed model, which loses 0.023 average precision | hotel-booking-cancellation-ml |
| H3 monitoring action replay | Done. Recall 65% to 87%, at the cost of flagging 73% instead of 44% of bookings | hotel-booking-cancellation-ml |
| S1 human audit | The blind sample of 60 claims and the scoring command are ready. **The judging is yours** | steam-reviews-nlp-rag, `evaluation/audit/human_audit_sample.csv` (not in git) |
| S2 Steam requirements | Partly. The six packages the serving file pins are pinned. The rest stay open because the notebook environment was not saved | steam-reviews-nlp-rag |
| S3 uncited sentences | Not done. Needs the Groq secret | |
| A1 AAPL panel | Done and merged. No model beats the random walk on any of 20 tickers after correction, gradient boosting is worse, only the conformal one-day intervals pass Kupiec on every ticker. The interval-score ranking is only partly the AAPL one | aapl-forecasting-backtest |
| A2 adaptive conformal | Not done | |
| A3 live-method rule | Done, written and not applied | aapl-forecasting-backtest |
| M1 movie pins and series table | Done | movie-analytics-data-pipeline |
| N1 M4 tests | Done | m4-nbeats-forecasting |
| X1 lint | Done for hotel, Steam and AAPL. Six Steam files stay exempt because their hashes are pinned in the evidence. The archive move was not done | |
| X2 validation headers | Done for six repositories | |

One mistake was made and fixed on the way: the first lint change in Steam altered six files whose hashes the provenance check pins, and `scripts/verify_retrieval_evidence.py` failed. It was caught before anything was merged, and the files were restored.

## Part 1. Findings

### 1.1 The profile page

**What works.** One flagship with a live demo, four featured projects with an image and a one-line result each, a table of how each project is served and checked. Results that went against the author are stated. That is rare and it is the profile's strongest trait.

**What to change.**

1. **The flagship block is too dense for a first read.** Seven bullets carry about thirty numbers. A reader who gives the page one minute cannot tell which number matters. Three bullets would do: the question and the data, the one clean test result, and the habit (pre-registered, read once, failures reported). The rest belongs behind the links.
2. **One flagship claim is now out of date.** The bullet on the backcast says the cover as sold did not lose money in 2019 to 2023 (60.6%). The flagship's own later section (`VALIDATION.md` 7I) shows that three of four ways of fitting the model ran 68% to 71%. The clause must be corrected (flagship task T8).
3. **Small inconsistencies.** The table says 472 tests and the suite now has more. The last flagship bullet says "September to December 2026" where the pre-registrations say September to November. Verify both against the flagship before editing.
4. **No single sentence says what job the profile is for.** The headline lists three roles. The work points one way: pricing and experimentation analytics for a commercial team. Say that, once.
5. **The "How I build" list reads as a keyword list.** Each tool is already shown in a project. Shorten it to one line per verb or link each skill to the project that proves it.
6. **The team projects sit next to the solo ones with the same weight.** Superstore and Steam are marked as team work, which is right. Consider moving Superstore to "More work" and promoting the AAPL backtest, which is solo, has a live track record and a result (conformal intervals that keep their coverage) that a pricing team understands.
7. **Repository settings the review could not see.** Descriptions, topics, pinned repositories and the social preview image are set in GitHub's interface. The owner checks them (task O-P1).

### 1.2 Across all repositories

1. **Lint strength differs.** `uplift-analyzer` runs the full ruff rule set and a format check. Hotel, Steam and AAPL run `--select E4,E7,E9,F` only, which catches syntax errors and unused names and little else.
2. **Dependency pinning differs.** Uplift pins with hashes. Hotel, AAPL, M4 and World Bank pin versions. `steam-reviews-nlp-rag/requirements.txt` is unpinned (the serving file is pinned). `movie-analytics-data-pipeline/requirements.txt` uses lower bounds only. `superstore` lists three unpinned packages.
3. **Each repository invents its own layout for evidence.** Most have `VALIDATION.md` and an `evidence.yml` workflow, which is good. The names of result folders differ (`outputs`, `results`, `snapshots`, `evals`). No change is proposed. A short common section at the top of each `VALIDATION.md` ("what was rerun, what was not, last run date") would let a reader compare them.
4. **Archived office documents sit in the code.** Hotel keeps a `.docx` and a `.pptx` in `docs/` and an older `figures/` directory. AAPL keeps `coursework/`. They are honest history and they dilute the first impression. Move each to one `archive/` directory with a one-line README.

### 1.3 Per repository

**uplift-analyzer.** The strongest after the flagship. Method gaps the README itself names, each closable with data on hand:

- The Qini curve, the learner comparison and the targeting policy use raw labels although assignment is slightly predictable from the features. The effect estimates were corrected for this and the model evaluation was not.
- The profit-maximizing targeting share is chosen and valued on the same held-out users. The README reports a split-half loss of about 20%.
- Learners are compared at default settings on a 500,000-user sample of 13.98 million.
- The evaluation of the language model with repair on is one run on synthetic data.

**hotel-booking-cancellation-ml.** Sound single-split validation and an unusually good monitoring replay. Gaps:

- One chronological cutoff. The README says repeated temporal backtesting is needed.
- `country` is flagged as a probable leak and is still an input of the deployed model.
- The monitoring replay shows the alert would have fired. It does not show what following the stated action (re-set the threshold on recent matured outcomes) would have recovered.
- The deployed model is the logistic regression, and the tracked comparison selects the random forest on validation. The reason is given. A calibrated gradient-boosting candidate with the same serving contract was not tried.

**steam-reviews-nlp-rag.** Good retrieval gate. Gaps:

- Claim support is judged by a model and no person has adjudicated a sample. The adjudication files exist and are empty.
- 40% of answer sentences carry no citation. The prompt was not changed and re-measured.
- The sentiment results are recorded history and were not rerun.
- `requirements.txt` is unpinned.

**aapl-forecasting-backtest.** Clean leakage tests and a live track record. Gaps:

- One asset. The README says a finding would need other tickers.
- The live forecast uses GARCH-conformal at 95% although plain GARCH scored slightly better there in the backtest. The README says so. A rule fixed in advance for which method goes live would remove the question.
- The conformal method uses a fixed window of 1,000 outcomes. Adaptive conformal inference, which adjusts the level after each miss, is the natural comparison and is absent.

**m4-nbeats-forecasting.** A faithful implementation with an honest protocol. Gaps:

- Two test files for a from-scratch network.
- Point forecasts only, six members, three of six frequencies. All stated. Low priority for the profile's direction.

**movie-analytics-data-pipeline.** The engineering path is the point. The analysis rests on 100 films from one run. Gaps:

- The sample is tiny and the dependencies are unpinned lower bounds, so a rerun in a year may not build.
- The monthly snapshot job exists. The README still reports the first comparison only.

**world-bank-indicators-analysis.** A descriptive study with a careful null (weak structure). No method gap worth new work. Keep as is.

**superstore-shipping-region-analysis.** The README states at the top that the dataset is a demonstration set whose profit is nearly a function of discount. That honesty is right and it also caps what the project can show. No new analysis is proposed.

## Part 2. Tasks

General rules for every task. One branch per repository, named `claude/portfolio-review`. Read that repository's `CLAUDE.md` and `VALIDATION.md` first and follow its style rules. Every number that enters a README comes from a committed results file. Run the repository's own check command (`make check` where it exists) before each commit. Open a pull request only when the owner asks. Never change a seed, a split or a threshold to improve a result. If a result gets worse, report it as the result.

"Fixed in writing first" below means: before running the analysis, commit a short section to that repository's `VALIDATION.md` that states the design, the metric and what will count as better, worse or no different. These repositories have no tag workflow, so the commit's push time is the record.

Priority order: P1, U1, U2, H1, H2, S1, A1, then the rest.

### P1. Profile page rewrite

- **Repository:** `dbechrakis/dbechrakis`. **File:** `README.md`.
- **Change:**
  1. Replace the headline's three roles with one sentence: what kind of problem the author solves and for whom.
  2. Cut the flagship block to three bullets (question and data, the one clean pre-registered result with its number, the discipline and that failures are reported with one example). Keep the image and the five links. Every number must already appear in the flagship's README on its default branch at the time of editing. Do not quote a number that exists only on an unmerged branch.
  3. Fix the test count and the "September to December" wording after checking the flagship.
  4. Shorten "How I build" to four lines, each skill linked to the project that shows it.
- **Gate:** the choice between keeping Superstore as featured or promoting AAPL is the owner's (O-P2). Do not move projects without it.
- **Done when:** the page reads in under one screen to the featured table at desktop width, every link returns 200 (check with a script), and no number differs from its source.

### U1. Uplift: evaluate the models with corrected scores

- **Repository:** `uplift-analyzer`. **Fixed in writing first:** yes.
- **Idea.** Replace raw outcomes in the Qini curve and the policy value with doubly robust scores (outcome model plus propensity weighting, cross-fitted), so that the model evaluation is corrected for the same imbalance as the effect estimates.
- **Files:** new `core/dr_scores.py`, changes in `core/metrics.py` and `core/policy.py` to accept a score column, `core/validate.py`, tests in `tests/`, `VALIDATION.md`, the README findings 3 and 4.
- **Fixed design:** same 500,000 sample, same seed 42, same 70/30 split, 5-fold cross-fitting, the existing propensity and outcome learners at their current settings. Report the Qini coefficient and its bootstrap interval with raw labels and with corrected scores side by side.
- **Tests:** on a simulation with a known effect and a known imbalance, the raw Qini is biased and the corrected one recovers the truth within its interval. Equivalence with the raw Qini when the propensity is constant.
- **Gate:** the repository rule says all statistics live in `core/` and the language-model layer computes nothing. If a tool description or `SYSTEM_PROMPT` must change to expose the new result, stop and report, because that requires rerunning `make eval`.
- **Done when:** both versions are in `outputs/validation.json`, the README shows both, and the limitation "Qini and policy results use the raw labels" is replaced by what was found.

### U2. Uplift: value the targeting policy on users it was not chosen on

- **Repository:** `uplift-analyzer`. **Fixed in writing first:** yes.
- **Change:** split the held-out 30% once more (seeded, half and half). Choose the targeting share on the first half and report its profit on the second, with a bootstrap interval. Report the old same-sample figure next to it.
- **Files:** `core/policy.py`, `core/validate.py`, tests, `VALIDATION.md`, README finding 4 and the limitation that calls the optimum optimistic.
- **Done when:** the README states the honest value and the size of the optimism in one sentence.

### U3. Uplift: three real-data evaluation runs with repair on

- **Repository:** `uplift-analyzer`. **Needs:** the local model server. **Gate:** if the model cannot be run in the executing environment, stop and report. This task is then the owner's.
- **Change:** run `make eval` three times on the 100,000-row real sample with the defaults, record all three, and replace "one run on synthetic data is indicative" with the three-run result and its range.

### H1. Hotel: rolling temporal backtest

- **Repository:** `hotel-booking-cancellation-ml`. **Fixed in writing first:** yes.
- **Design to fix:** four cutoffs spaced three months apart, the last being the existing 2017-01-12 if the data allows four with at least 5,000 later bookings each. The same maturity rule and exclusions as the current split. For each cutoff: average precision, ROC AUC, Brier score of the calibrated probability, and recall at the threshold set on the first 30% of later bookings. Models: the deployed logistic regression pipeline, unchanged.
- **Step 1, counts only:** count train and later bookings for each candidate cutoff. **Gate:** if fewer than three cutoffs have 5,000 later bookings, stop and report the counts.
- **Files:** new `scripts/rolling_backtest.py`, `outputs/rolling_backtest.csv`, a test with a small synthetic frame that no training row is dated after its cutoff, `VALIDATION.md`, a short README table under "Validation design".
- **Done when:** the README states the range of each metric across cutoffs and whether the single-split numbers sit inside it.

### H2. Hotel: decide what to do about `country`

- **Repository:** `hotel-booking-cancellation-ml`.
- **Step 1:** on the existing split, report the metrics of the deployed pipeline with and without `country` (the README already has ROC AUC 0.743 and average precision 0.622 without it), plus calibration error and recall at the threshold for both.
- **Gate, owner decides:** whether the deployed artifact drops `country`. Dropping it changes the model fingerprint, the API contract and the committed metrics. Stop after step 1 and report the table.
- **If the owner says drop:** retrain, update the SHA-256 fingerprint, `outputs/`, the tests that pin scores, the API example and the README. The lower scores are the result.

### H3. Hotel: replay the stated action

- **Repository:** `hotel-booking-cancellation-ml`. **Fixed in writing first:** yes.
- **Rule to fix:** when the rolling PSI is at alert for two consecutive weeks, re-set the threshold to the 80% recall point on outcomes that had matured by then. No retraining.
- **Files:** `scripts/monitoring_replay.py` (a second mode), `outputs/monitoring/`, a leakage test (the re-set uses only matured outcomes), README monitoring section.
- **Done when:** the README states the realised recall with and without the action over the same 27 weeks.

### S1. Steam: human adjudication of a sample (owner and model together)

- **Repository:** `steam-reviews-nlp-rag`.
- **Model's part:** draw a seeded random sample of 60 of the 399 judged claims, stratified by the judge's label, and write `evaluation/audit/human_sample.csv` with the claim, the five reviews and an empty verdict column. Do not include the judge's label in the file the person fills.
- **Gate:** the verdicts are the owner's (O-S1). The model must not fill them.
- **After the owner fills the file:** compute agreement with the judge (raw and Cohen's kappa with a bootstrap interval), add it to `evaluation/README.md` and replace "not yet by a person" in both READMEs.

### S2. Steam: pin the research requirements

- **Files:** `requirements.txt`. Pin every package to the version that the CI environment currently resolves. **Verify first** that the evidence workflow passes with the pins. Low risk, small.

### S3. Steam: reduce uncited sentences

- **Fixed in writing first:** yes. **Needs:** the `GROQ_API_KEY` secret, so the run itself is the owner's through the answer-evaluation workflow.
- **Change:** one prompt change that requires a citation on every factual sentence. Re-run generation and judging on the same 80 questions. Report the share without a citation and the supported rate before and after. **Gate:** if the supported rate falls by more than its interval's half width, keep the old prompt and report that.

### A1. AAPL: the same backtest on a fixed panel of tickers

- **Repository:** `aapl-forecasting-backtest`. **Fixed in writing first:** yes, including the ticker list.
- **Design to fix:** 20 tickers chosen by a stated rule that does not look at returns (for example the 20 largest S&P 500 members by index weight on a stated past date, listed by name in the write-up). Same origins, models, horizons and tests. Report, per interval method, the number of tickers whose coverage passes Kupiec at 95%, and the pooled interval score. For point forecasts report the number of tickers where any model beats the random walk under Diebold-Mariano, with Benjamini-Hochberg across tickers.
- **Gate:** if the price source cannot serve a ticker for the full period, drop it, name it, and do not replace it.
- **Files:** `scripts/` panel runner, `outputs/panel/`, tests that the leakage test runs for a second ticker, README section "Does it hold beyond AAPL".
- **Done when:** the README answers the question in one table.

### A2. AAPL: adaptive conformal intervals

- **Fixed in writing first:** yes. Add adaptive conformal inference (step size fixed in advance at 0.005) as a sixth interval method. Same scoring. The leakage test must cover it. The live method does not change unless the rule in A3 says so.

### A3. AAPL: a rule for the live method

- State in `VALIDATION.md` the rule by which a method replaces the live one (for example: better interval score at both levels on the latest refreshed backtest, checked once a year on a stated date). Apply nothing now.

### M1. Movie: pin and report the snapshot series

- Pin `requirements.txt` to the versions the CI resolves today. Add a small table to the README built from `snapshots/` that shows the median ROI of each monthly snapshot under both sampling profiles, generated by a script and checked by a test. **Verify first** how many snapshots exist. If fewer than three, add the table anyway and say how many.

### N1. M4: tests for the network

- Add tests for the basis functions of the interpretable blocks (a trend block reproduces a known polynomial, a seasonality block a known sinusoid), for the window sampler never reaching the validation horizon, and for the median ensemble. No retraining.

### X1. All repositories: lint and archive

- Widen ruff in Hotel, Steam and AAPL to the rule set and format check of `uplift-analyzer` (`pyproject.toml` there). Fix what it reports without changing behaviour. **Gate:** if a fix would change a numeric output or a pinned artifact hash, leave that line, add a per-line ignore, and list it in the commit message.
- Move archived office documents and coursework into one `archive/` directory per repository with a one-line README. Update any link. **Verify first** that no test or workflow reads those paths.

### X2. All repositories: a common header in `VALIDATION.md`

- At the top of each `VALIDATION.md`, three lines: what was rerun in the latest review and when, what is recorded history and was not rerun, and the single command that reproduces the rerun part. Take the facts from the file itself. Do not rerun anything for this task.

### Owner actions

- **O-P1.** In GitHub's interface: check each repository's description, topics and social preview, and the six pinned repositories. Suggested pins in order: missed-connection-pricing, uplift-analyzer, aapl-forecasting-backtest, hotel-booking-cancellation-ml, steam-reviews-nlp-rag, movie-analytics-data-pipeline.
- **O-P2.** Decide whether AAPL replaces Superstore in the featured table.
- **O-S1.** Judge the 60 sampled claims of S1.
- **O-H2.** Decide on `country` after the table of H2.

### Not proposed, and why

- New analysis on Superstore or World Bank. Their limits come from the data and both READMEs say so.
- The remaining M4 frequencies or a larger ensemble. Compute for little change in what the project shows.
- A new project. The flagship has two forward tests waiting on data, and finishing those reads is worth more than a tenth repository.
