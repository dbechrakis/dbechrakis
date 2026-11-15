# Plan 06. Sequential testing and variance reduction, checked by replay

**Repository:** `dbechrakis/uplift-analyzer` (existing). **Branch:** `claude/sequential-testing`. **Size:** about two weeks, four sessions.

## 1. Why this exists

The repository analyses a finished randomized test. It has a sample ratio check, a power calculation and a covariate-adjusted effect. It says nothing about what happens while a test is running: looking early, stopping early, and how much sample a covariate saves. Those are the questions an experimentation team asks every week.

The question of this plan: **if someone had watched this test every day, how often would each stopping rule have declared a winner that was not there, and what does each honest rule cost in sample?**

The answer is measured, not quoted from theory. The Criteo control group is a pool of real users who all received the same treatment. Splitting it at random into two fake arms gives an experiment in which the true effect is exactly zero. Thousands of such experiments give the false positive rate of any rule on real outcome data.

## 2. Read first

- `CLAUDE.md` of the repository. The rules that bind this plan are: no code comments, all statistics in `core/`, every result carries a `data_source`, the default sample stays 500,000 and the default seed stays 42, tests never touch the network or the real dataset, `make check` before every commit, prose without dashes and semicolons.
- `core/power.py`, `core/adjusted.py`, `core/srm.py`, `core/results.py`, `core/data.py`, `core/validate.py`, and `tests/test_demo_power_js.py`.
- The gate that matters most: `SYSTEM_PROMPT` in `app/agent.py` and the tool descriptions in `app/tools.py` determine the published language model evaluation. **This plan does not touch either.** The new results are not exposed as a tool. If any step seems to need a change there, stop and report.

## 3. Method references the executing model must consult

Do not write these from memory. Open the source. The repository forbids code comments, so the references go in a new `docs/sequential-method.md`.

- Group sequential boundaries: Jennison and Turnbull, *Group Sequential Methods with Applications to Clinical Trials* (2000), chapter 2 for the Pocock and O'Brien-Fleming constants. Lan and DeMets (1983) for alpha spending.
- Mixture sequential probability ratio test: Johari, Koomen, Pekelis and Walsh, "Peeking at A/B Tests" (KDD 2017) and the longer "Always Valid Inference" (Operations Research 2022).
- Confidence sequences: Howard, Ramdas, McAuliffe and Sekhon, "Time-uniform, nonparametric, nonasymptotic confidence sequences" (Annals of Statistics 2021), the normal mixture boundary.
- CUPED: Deng, Xu, Kohavi and Walker (WSDM 2013). With a model prediction as the covariate: the "CUPAC" note by DoorDash engineering (2020).

## 4. Design

### 4.1 Data

- **Pool.** Every control row of the full Criteo file, about 2.1 million of the 13,979,592 rows (the design share of treatment is 0.85). Loaded by a new function `load_control_pool()` in `core/data.py` that streams the file in chunks and keeps rows with `treatment == 0`. It caches to `data/cache/criteo_control_pool.parquet`. This does not change the default sample or the default seed, which stay as they are for every existing result.
- **Primary outcome:** `visit`. **Secondary:** `conversion`, which is much rarer and is reported in the same tables.
- **An experiment** is a draw of `2 × n_arm` rows from the pool without replacement, the first half labelled arm A and the second arm B. Rows arrive in the drawn order. The file has no timestamps, so arrival order is random by construction. This is stated as a limit: real traffic has day-of-week patterns and novelty effects that a random order does not have.
- **Covariate for CUPED.** The file has no pre-period outcome. The covariate is a model prediction of the outcome from the twelve features, fitted on a part of the pool that no experiment draws from (a fixed 20% of the pool, by row index modulo 5 equal to 0). The experiments draw from the other 80%. This keeps the covariate independent of the outcomes it adjusts.
- **Data source labels.** Null experiments carry `data_source="criteo"`. Experiments with an injected effect carry `data_source="constructed"`. Tests use `"synthetic"`.

### 4.2 Stopping rules compared

All at a two-sided level of 0.05, on the difference in means between the two arms.

| Rule | What it does |
|---|---|
| `fixed` | One test at the planned end. The reference. |
| `peek` | The fixed-horizon z test applied at every look, stopping at the first p below 0.05. What a dashboard invites. |
| `bonferroni` | The same test at level 0.05 divided by the number of looks. |
| `pocock` | Group sequential, constant boundary. |
| `obf` | Group sequential, O'Brien-Fleming boundary through Lan-DeMets spending. |
| `msprt` | Mixture sequential probability ratio test with a normal mixing distribution. Valid at any number of looks. |

Each rule is run without adjustment and with the CUPED-adjusted outcome.

### 4.3 The grid, fixed before any run

- `n_arm = 50,000`.
- Looks: 5 equally spaced, 20 equally spaced, and 200 equally spaced (every 250 users per arm, standing in for "continuous").
- Replications: 5,000 null experiments, with seeds 0 to 4,999 derived from the repository default seed 42 through `numpy.random.SeedSequence(42).spawn(5000)`.
- Injected effects: relative lifts of 2%, 5% and 10% on the visit rate, 2,000 replications each. The injection turns a zero into a one in arm B with the probability that produces the stated lift exactly in expectation.
- `msprt` mixing variance: set by the rule "the squared absolute effect that the fixed test at `n_arm` detects with 80% power", computed by `core.power`. It is fixed before the run and not tuned.

## 5. Text to commit first

Commit and push the following as a new section of `VALIDATION.md`, before `load_control_pool()` is run on the real file. Fill the date and the bracketed value S from the repository's existing outputs. Nothing else is filled in.

> ## Sequential testing replay, design fixed before it was run, [date]
>
> Written before any replay was run on the real file. Section 4 of the plan (pool, experiment, rules, grid, mixing variance) is copied here in full.
>
> **Expectations.**
>
> - Q1. `fixed` has a false positive rate between 4.4% and 5.6% on the null experiments. The band is 5% plus or minus two Monte Carlo standard errors at 5,000 replications.
> - Q2. `peek` has a false positive rate above 12% at 5 looks, above 20% at 20 looks and above 30% at 200 looks.
> - Q3. `pocock` and `obf` are between 4.4% and 5.6% at 5 and at 20 looks.
> - Q4. `msprt` is at or below 5.6% at every number of looks, and below 5% at 200 looks.
> - Q5. `bonferroni` is at or below 5.6% and below `obf` in power at every injected lift.
> - Q6. At the 5% lift, `obf` stops before the planned end in more than half of the experiments, and its average sample is smaller than the fixed sample.
> - Q7. Among experiments that stop early for a winner at the 5% lift, the average estimated lift is above the true lift for `peek`, `pocock` and `obf`.
> - Q8. The variance saving of the CUPED-adjusted `visit` outcome is within 3 percentage points of [S], where S is one minus the squared ratio of `std_error` to `unadjusted_std_error` of the existing adjusted effect on `visit` in `outputs/validation.json`. The covariate model is a logistic regression on the twelve features with default settings, so that it matches the linear adjustment the repository already reports. A saving outside the band is reported as a failed expectation, in either direction.
> - Q9. No prediction on whether CUPED changes any false positive rate. Reported either way.
>
> **What counts.** Each expectation is graded passed or failed by its own rule from `outputs/sequential.json`. Nothing is rerun with other settings after the first run. A failed expectation is reported in the README next to the passed ones.
>
> **Not claimed.** That arrival order resembles real traffic. That results transfer to a metric with a heavy tail. That the pool is a random sample of anything beyond the Criteo control group.

Do not soften or widen a band. If the executing model believes a band is wrong for a mathematical reason, stop and report before committing.

## 6. Work packages

### WP0. Orientation (small)

- Run `make install` and `make check`. Record the test count.
- Confirm that `https://huggingface.co/datasets/criteo/criteo-uplift` is reachable from the session. It answered on 15 November 2026.
- **Gate:** if the file cannot be downloaded, do WP1 and WP2 on synthetic data, commit the design of section 5, and stop. The real run is then the owner's.

### WP1. `core/sequential.py`, the rules (medium)

- **Functions.**
  - `z_statistics(sum_a, sum_b, sumsq_a, sumsq_b, n)`: the z statistic at each look from cumulative sums. Vectorised over looks.
  - `spending_boundaries(information_fractions, alpha, kind)`: critical values for `pocock` and `obf` by Lan-DeMets spending. Compute the boundary at look k by solving for the value whose crossing probability equals the alpha spent at that look, given no earlier crossing. Use recursive numerical integration over the joint normal distribution of the statistics, as in Jennison and Turnbull chapter 19. If that proves too long for one session, a Monte Carlo solution with a fixed seed and two million Brownian paths is acceptable, and must be stated.
  - `msprt_p_values(...)`: the always-valid p value at each look, the running minimum of the inverse likelihood ratio.
  - `confidence_sequence(...)`: the interval implied by the same mixture, at each look.
  - `stopping_look(statistics, boundaries)`: the first look at which a rule stops, or none.
  - `cuped_adjust(y, x)`: returns `y - theta * (x - mean(x))` with `theta = cov(y, x) / var(x)` estimated on the pooled arms.
- **Result dataclasses** go in `core/results.py`, frozen, with `data_source`, following the existing ones.
- **Tests** in `tests/test_sequential.py`, on synthetic data only:
  1. With 5 equally spaced looks at alpha 0.05, the Pocock constant is 2.413 and the O'Brien-Fleming final critical value is 2.040, each within 0.01. **Verify both numbers against Jennison and Turnbull tables 2.1 and 2.3 before pinning them.** If the book disagrees with this plan, the book wins and the plan's numbers are reported as wrong.
  2. On 20,000 simulated normal null experiments with 5 looks, `pocock`, `obf` and `fixed` reject between 4.4% and 5.6%, and `peek` rejects more than 12%.
  3. `msprt` p values never increase from one look to the next.
  4. The confidence sequence contains the true effect at every look in at least 95% of 5,000 simulated experiments.
  5. `cuped_adjust` leaves the mean difference unchanged in expectation and reduces the variance by the squared correlation, on a simulation with a known correlation of 0.6.
  6. With one look, every rule equals `fixed`.

### WP2. `core/replay.py`, the experiment generator (small)

- `draw_experiment(pool, n_arm, seed_sequence)`, `inject_lift(outcome_b, relative_lift, rng)`, `cumulative_sums(outcome, looks)`.
- `load_control_pool()` in `core/data.py`, with a row count check against the full file and a SHA-256 check through the existing `sha256_of`.
- **Tests:** the two arms of a draw never share a row. The holdout fifth used to fit the covariate model is never drawn. `inject_lift` produces the stated lift within Monte Carlo error on a large synthetic arm. Changing rows after a look cannot change that look's statistic (the leakage test of this plan).

### WP3. Commit the design, then run (medium)

1. Commit and push section 5 into `VALIDATION.md`. Record the commit hash.
2. Only then run `python -m core.sequential_study`, a new module that writes `outputs/sequential.json`. It holds, for each rule, look count, adjustment and injected lift: the rejection rate with a Wilson interval, the average and median stopping sample, the share stopped early, and the average estimated lift among early winners.
3. Add `make sequential` to the `Makefile`.
4. Runtime target: under 30 minutes on a laptop. Work on cumulative sums, in chunks of 500 experiments, never on per-row loops.
- **Gate:** if Q1 fails (the fixed test is outside 4.4% to 5.6% on real data), stop and report before writing anything else. A failed Q1 means the generator or the test is wrong, or the pool is not exchangeable, and every other number depends on it.

### WP4. One real replay (small)

The real test, treated against control, in a random arrival order with seed 42, on the existing 500,000 default sample. At 20 equally spaced looks, report for each rule the look at which it would have stopped and the estimate at that look, next to the final estimate the README already reports. One table. This is one path and is labelled as an illustration, not a result.

### WP5. Tell it (medium)

- `VALIDATION.md`: a result section under the design section, same day as the run, with every expectation graded.
- `README.md`: a new finding, "What peeking would have cost", with one table (false positive rate by rule and looks) and one sentence per failed expectation. Add `core/sequential.py` and `core/replay.py` to the "Core functions" table and the project layout.
- `docs/sequential-method.md`: one page on each rule, with its reference and its limit.
- A test in `tests/` that reads `outputs/sequential.json` and checks every number the README quotes, following how the repository already pins numbers.
- **Optional, only if time remains:** a "Peeking" panel in the static demo (`demo/app.js`, `demo/index.html`) that draws the false positive rate against the number of looks from a small JSON exported by `core/export_demo.py`. Follow `tests/test_demo_power_js.py` for the test. Skip without regret.

## 7. Done when

- `make check` is green and needs neither the network nor the dataset.
- `outputs/sequential.json` exists, and the README table matches it under test.
- Every one of Q1 to Q9 is graded in `VALIDATION.md`.
- `app/agent.py` and `app/tools.py` are byte-identical to the base branch. Check with `git diff --stat main -- app/agent.py app/tools.py`.

## 8. Limits to state in the README

- Arrival order is random, so the replay has no weekday or novelty pattern.
- One dataset, one metric family (rare binary outcomes).
- The covariate is a model prediction and not a pre-period outcome, so the CUPED saving here is a lower bound on what a product with user history would see.
- Null experiments drawn from one pool share users across replications. Each experiment is valid on its own. The replications are not independent draws from a larger population, and the Monte Carlo intervals are slightly too narrow for that reason.

## 9. What the owner learns

Alpha spending, always-valid inference, why early winners overstate their effect, and how to verify a testing procedure by replay before trusting it. These are the working vocabulary of an experimentation platform team.
