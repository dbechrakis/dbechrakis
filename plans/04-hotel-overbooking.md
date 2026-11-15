# Plan 04. From a cancellation probability to a room limit

**Repository:** `dbechrakis/hotel-booking-cancellation-ml` (existing). **Branch:** `claude/overbooking`. **Size:** two to three weeks, five sessions.

## 1. Why this exists

The repository predicts which bookings cancel and uses the score to rank bookings for a retention review. Its own README says the value of that review rests on two assumptions nobody measured (the success rate of a contact and the margin recovered).

There is a second use of the same probability that needs no such assumption about guests' reactions: **how many rooms can a hotel sell above its capacity for a given night, given the bookings it holds?** This is the classical overbooking problem. It turns a prediction into a decision under a constraint, which no project in the portfolio does today.

Two questions, in this order:

1. **Forecast.** For a night and a number of days before it, how well does each method predict how many of the bookings on hand will show up? This needs no assumption about demand and can be scored exactly.
2. **Decision.** Given that forecast, what room limit does a cost rule recommend, and what would it have gained or cost under stated demand scenarios? This part is scenario arithmetic and is labelled so.

The honest possibility is that a flat cancellation rate by days before arrival does as well as the booking-level model. If so, that is the result.

## 2. Read first

- `README.md`, `VALIDATION.md` (note how each study has a section "design fixed before it was run" followed by "Result"), `src/hotel_cancellation/data.py`, `src/hotel_cancellation/decision.py`, `scripts/rolling_backtest.py`, `scripts/action_replay.py`, `ci/verify_evidence.py`, `Makefile`.
- The repository has no `CLAUDE.md`. Its working rules, taken from its practice: tests use `unittest` and run with `make test`, lint is `make lint`, the full gate is `make check`, the model artifact and its SHA-256 are frozen, and nothing is retuned after a run.
- **Frozen, never touched by this plan:** `artifacts/*`, `outputs/holdout_predictions.csv.gz`, the API contract in `src/hotel_cancellation/contracts.py` and `api.py`, and every existing section of `VALIDATION.md`.

## 3. What the data allows, counted on 15 November 2026

The source file was downloaded and matched the recorded SHA-256 `7c2ae42a…1fc06`. These are counts taken to check feasibility. The executing model recomputes them in WP1 and reports any difference.

- 119,390 bookings. 75,166 checked out, 43,017 cancelled, 1,207 no-shows. 715 bookings have zero nights and are left out of the room ledger.
- **The file has no capacity field.** The largest number of rooms actually occupied on any night before the cutoff of 2017-01-12 is **226 for the City Hotel and 187 for the Resort Hotel**. The 95th percentile of occupied rooms is 224 and 183, so both hotels sit against a hard ceiling on many nights. The ceiling is used as the capacity. This is an inference and is stated as one.
- After the cutoff the hotels are often nearly full. Of 202 nights from 2017-02-11 to 2017-08-31, the bookings on hand seven days before reach 90% of capacity on 127 nights for the City Hotel and 87 for the Resort Hotel.
- **Most cancellations happen early.** Of the bookings on hand for a night, the share that later cancel is about 13% at 30 days before, 8% at 14 days, 3% at 7 days and 2% at 3 days for the City Hotel, and 9%, 5%, 3% and 1% for the Resort Hotel. The overall cancellation rate of 31% is therefore the wrong number for this decision, and the deployed probability, which is made at booking time, cannot be used as it is.

## 4. Design

### 4.1 Definitions

- **Booking date** = arrival date minus `lead_time`, as `data.py` computes it. **Departure** = arrival plus nights.
- **A booking covers night d** when arrival ≤ d < departure.
- **Decision date** for night d at horizon h: t = d − h days.
- **On hand at (d, h):** bookings that cover d, with booking date ≤ t, and not already cancelled by t. A cancelled booking's `reservation_status_date` is its cancellation date. A no-show counts as not showing up and is on hand until the night.
- **Shows:** `is_canceled == 0`.
- **Pickup at (d, h):** bookings that cover d with booking date after t and that show.
- **Capacity C:** per hotel, the largest number of occupied rooms on any night before 2017-01-12. Recomputed in WP1.
- **Horizons:** 30, 14, 7 and 3 days.

### 4.2 Which bookings can be scored honestly

The deployed model was trained on bookings whose stay had ended before 2017-01-12. Any booking that covers a night after that date was not in training, so the frozen artifact scores it out of sample. This includes the rows the split calls "purged" (booked before the cutoff, staying after it). Score those with the frozen artifact. Do not retrain.

### 4.3 The probability each method gives to a booking on hand

Every method answers: what is the probability that this booking, which is still alive at t, shows up?

| Method | Probability of showing |
|---|---|
| `flat` | One minus the share of on-hand bookings at horizon h that later failed to show, per hotel, from past nights only. |
| `segment` | The same by hotel, `deposit_type` and `market_segment`, each cell shrunk toward the hotel rate with weight n ÷ (n + 50). |
| `at_booking` | One minus the deployed calibrated probability, unchanged. Expected to be badly wrong, for the reason in section 3. It is included to show the size of the error. |
| `model` | An isotonic calibrator of "later failed to show" on the deployed `risk_score`, fitted per horizon on past bookings that were alive at that horizon. |
| `model_od` | `model`, with the count distribution widened by an overdispersion factor estimated from past nights (section 4.5). |

**Past only, walk forward.** A method used at decision date t may learn only from bookings whose night of arrival is before t, because only those outcomes are known. Refit on the first day of each calendar month from all eligible past bookings. For the booking-level calibrator use one row per booking, with the horizon measured to its arrival night.

**Out-of-sample pool only.** The calibrator and the baselines learn only from bookings the frozen model did not train on (arrival on or after 2017-01-12 minus their stay, in practice every booking that covers a night from the cutoff on).

### 4.4 Evaluation nights

Nights from **2017-04-01 to 2017-08-31**, both hotels, four horizons. The start leaves the methods eleven weeks of known out-of-sample outcomes before the first decision date at the 30-day horizon.

**Gate, counts only, before any outcome is scored:** at the first decision date of each horizon, the pool must hold at least 1,000 bookings alive at that horizon with at least 50 that later failed to show, per hotel. If a horizon fails, move the start of the evaluation two weeks later for all horizons, at most twice. If it still fails, stop and report the counts.

### 4.5 The count forecast

For a night and horizon, the number of on-hand bookings that show is a sum of independent yes or no outcomes with different probabilities. Its distribution (the Poisson binomial) is computed exactly by convolution.

Bookings are not independent. A group cancels together. Expect the exact distribution to be too narrow. `model_od` estimates, from past nights only, the ratio of the observed squared error to the predicted variance, and widens the distribution to match by mixing it with a discretised normal of the missing variance.

**Scores**, per method and horizon, pooled over nights and hotels:

- Mean error in rooms (bias).
- Continuous ranked probability score, in rooms.
- Coverage of the central 80% and 95% intervals.
- Intervals for the scores by a block bootstrap over calendar weeks, because adjacent nights share most of their bookings.

### 4.6 The room limit

The hotel earns r for a room filled and pays w for a guest it cannot house. Sell the k-th room above capacity while the probability that it leads to a walk stays at or below r ÷ (r + w). With S the number of guests arriving for the night:

> accept extra room k while P(S_k ≥ C + 1) ≤ r ÷ (r + w)

where S_k is the forecast of shows from the bookings on hand, plus k more bookings that each show with the method's average probability, plus the pickup still to come.

- **Pickup forecast:** the empirical distribution of pickup at this horizon for this hotel and weekday, from past nights only, convolved in.
- **Cost ratios:** w ÷ r of 1, 2 and 3. Reported for all three. The middle one is the headline.
- r is taken as 1, so every value is in units of one room night's revenue. No currency figure is claimed.

### 4.7 What the limit would have been worth, as scenarios

For each night, horizon, method and cost ratio the rule gives a number of extra rooms k. What happens then depends on demand that the data cannot show, because a hotel that does not overbook records no turned-away guests. The value is therefore computed under a stated share φ of the extra rooms that would in fact have been booked, on constrained nights only.

- **Constrained night:** bookings on hand at the horizon are at least 90% of capacity.
- **Extra bookings taken:** φ × k on constrained nights, 0 elsewhere, for φ of 0.25, 0.5 and 1.
- **Their show rate:** the realised show rate of that night's actual pickup, or the hotel's rate when the night has fewer than 10 pickup bookings.
- **Rooms free on the night:** C minus the rooms actually occupied.
- **Gain:** extra shows up to the free rooms. **Walks:** extra shows beyond the free rooms. **Net value:** gain minus w ÷ r times walks.

Reported per hotel per 100 nights, for every method, next to "no overbooking" which is zero by definition. Each table carries the line: *scenario arithmetic under an assumed demand share, not a measured outcome.*

## 5. Text to commit first

Commit and push the following as a new section of `VALIDATION.md` before any outcome of an evaluation night is scored. Section 4 is copied in full above it. Fill the date, the recomputed capacities and the gate counts. Nothing else is filled in.

> ## Overbooking study, design fixed before it was run — [date]
>
> Written after the counts of the gate and before any forecast was scored.
>
> **Expectations.**
>
> - O1. `at_booking` over-predicts the rooms lost at every horizon. Its mean error is negative (fewer shows predicted than realised) by more than 5 rooms per night at the 7-day horizon for the City Hotel.
> - O2. `model` has a lower ranked probability score than `flat` at the 30-day and 14-day horizons. No prediction at 7 and 3 days, where little is left to predict.
> - O3. `model` has a lower ranked probability score than `segment` at the 30-day horizon by more than the bootstrap standard error of the difference. If the difference is within one standard error, the reading is "the booking-level model adds nothing over three segment rates", and that sentence goes in the README.
> - O4. The exact count distribution is too narrow. The 80% interval of `model` covers fewer than 70% of nights at the 30-day horizon.
> - O5. `model_od` covers between 70% and 90% of nights with its 80% interval at every horizon.
> - O6. At a cost ratio of 2 and the 30-day horizon, the rule under `model_od` recommends at least one extra room on more than half of the constrained nights.
> - O7. No prediction on the sign of the net value. Reported for all nine combinations of cost ratio and demand share.
>
> **What counts.** Each expectation is graded by its own rule from `outputs/overbooking/summary.json`. The methods, horizons, evaluation window, capacity rule, cost ratios and demand shares are not changed after the first run.
>
> **Not claimed.** That the capacity is the hotels' true room count. That demand existed for the extra rooms. That a walk costs a fixed multiple of a room night. Any effect on guests, reviews or repeat business. Independence between nights.

## 6. Work packages

### WP0. Orientation (small)

- `make install`, `python scripts/download_data.py`, `make check`. Record the test count.
- **Gate:** if the download's SHA-256 differs from the recorded one, stop. The repository's own script already refuses.

### WP1. The room ledger, and counts only (small)

- New `src/hotel_cancellation/ledger.py`:
  - `nightly_ledger(frame)`: one row per booking and night it covers, with booking date, cancellation date where there is one, and whether it showed.
  - `on_hand(ledger, night, horizon)`, `pickup(ledger, night, horizon)`, `occupied(ledger, night)`.
  - `capacity(ledger, before)`: the largest occupancy before a date.
- `scripts/overbooking_counts.py` writes `outputs/overbooking/counts.json`: the capacities, the number of evaluation nights, the gate counts of section 4.4, and the number of constrained nights per hotel and horizon. **It must not compute any forecast score.**
- Tests in `tests/test_ledger.py`, on a hand-built frame of about ten bookings with a known answer for every function. One test must be the leakage test: changing a booking's outcome after date t cannot change `on_hand` at t.
- Compare the capacities with section 3 (226 and 187). Report any difference before going on.

### WP2. Probabilities and the count forecast (medium)

- New `src/hotel_cancellation/overbooking.py`:
  - `poisson_binomial(probabilities)`: exact distribution by convolution.
  - `widen(distribution, factor)`: the overdispersion step.
  - `horizon_calibrator(scores, failed)`: isotonic, increasing in the risk score.
  - `segment_rates(frame, prior_weight=50)`.
  - `crps_discrete(distribution, observed)`, `interval(distribution, level)`.
  - `extra_rooms(distribution_of_shows, show_probability, capacity, cost_ratio, max_extra=40)`: the rule of section 4.6.
  - `scenario_value(extra, demand_share, show_rate, free_rooms, cost_ratio)`: section 4.7.
- Tests in `tests/test_overbooking.py`:
  1. `poisson_binomial` with equal probabilities equals the binomial. With probabilities 0 and 1 it is a point mass.
  2. `crps_discrete` of a point mass at the observed value is zero and grows with distance.
  3. `extra_rooms` is zero when the hotel is certain to be full, never decreases as the cost ratio falls, and matches a hand-worked case with three bookings.
  4. `widen` keeps the mean and multiplies the variance by the factor, within rounding.
  5. On a simulation where bookings cancel in groups of five, the exact distribution under-covers and the widened one covers within 5 points of nominal.
  6. A walk-forward test on a synthetic year: a calibrator used at date t was fitted on no booking arriving on or after t.

### WP3. Commit the design, then run (medium)

1. Commit and push section 5 into `VALIDATION.md` with the counts of WP1. Record the hash.
2. `scripts/overbooking_backtest.py` scores the purged and later bookings with the frozen artifact, runs every method walk-forward, and writes:
   - `outputs/overbooking/forecast_scores.csv` (method, horizon, hotel, bias, score, coverage, intervals),
   - `outputs/overbooking/limits.csv` (night, hotel, horizon, method, cost ratio, extra rooms),
   - `outputs/overbooking/scenarios.csv`,
   - `outputs/overbooking/summary.json` with every number the documents quote and each expectation's grade.
3. Add `make overbooking`.
- **Gate:** if the frozen artifact cannot score the purged rows (a missing category, a contract error), stop and report. Do not change the artifact or the contract.

### WP4. Tell it (medium)

- `VALIDATION.md`: "Result of the overbooking study (same day)", every expectation graded, failed ones first.
- `README.md`: a new section after "Decision economics", titled "From a probability to a room limit". Three short paragraphs and two tables (forecast scores by method and horizon, scenario value at the middle cost ratio). The first sentence states that capacity is inferred and that the value is scenario arithmetic. Add a row to "Decision in 60 seconds" only if O2 or O3 passes.
- `docs/overbooking-method.md`: the rule, its derivation in ten lines, and the limits.
- `ci/verify_evidence.py`: recompute from the committed CSVs every number in `summary.json` that the README quotes, in the terse style that file already uses.
- **Optional:** a fourth tab in `app/streamlit_app.py` that reads `outputs/overbooking/limits.csv` and shows, for a chosen hotel and horizon, the recommended extra rooms over time against the rooms that were in fact free. Read-only, no new model call. Skip if the app's tests would need restructuring.

### WP5. Second series, only if the owner asks (medium)

The study above has five months of nights from one model. `scripts/rolling_backtest.py` already fits the pipeline at four cutoffs. A second series would repeat the study for the nights after each earlier cutoff, with the pipeline fitted at that cutoff, which gives about sixteen months and a full year of seasons. It needs its own design section before it is run. Do not start it without the owner's word.

## 7. Done when

- `make check` is green without the data file.
- All seven expectations are graded in `VALIDATION.md`, and `ci/verify_evidence.py` reproduces the README's numbers.
- `git diff --stat main -- artifacts src/hotel_cancellation/contracts.py src/hotel_cancellation/api.py` is empty.

## 8. Limits to state in the README

- Capacity is the highest occupancy seen, not a published room count. A sensitivity line reports the scenario value at capacity minus 5 and plus 5.
- The data records no turned-away demand. The value depends on the assumed demand share, and the table shows all three.
- About 150 nights for each of two hotels in one spring and summer. Adjacent nights share bookings.
- The frozen model was built to rank bookings at booking time. It is reused here with a horizon calibrator, and a model built for survival from a given day might do better.
- Room types are ignored. A hotel overbooks by room type.

## 9. What the owner learns

The newsvendor rule and its critical ratio, exact count distributions and why real counts are wider, walk-forward calibration at a decision horizon, and how to report a decision's value when the counterfactual is not in the data.
