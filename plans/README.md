# Project plans, written 15 November 2026

This directory lives on branch `claude/funny-pasteur-cdi67q` of `dbechrakis/dbechrakis` only. It is a working document and must not be merged into `main`, because `main` is the public profile page. The earlier review, `PORTFOLIO_PLAN.md`, lives the same way on branch `claude/tender-hypatia-z5ndyk`.

Each plan is a handoff. An executing model should be able to run it from the file alone. Every work package states the files it touches, what done means, and where to stop and report. Where a judgment call is needed, the plan says "owner decides" and the work stops there.

| Plan | Repository | New or existing | Size | What it adds to the portfolio |
|---|---|---|---|---|
| [01 Offer ranking and position bias](01-expedia-ranking.md) | `offer-ranking-position-bias` | New | 5 to 7 weeks | Choice models, ranking, off-policy evaluation |
| [02 `whatchanged` package](02-whatchanged-package.md) | `whatchanged` | New | 4 to 6 weeks | Reusable software that others install |
| [04 Overbooking limits](04-hotel-overbooking.md) | `hotel-booking-cancellation-ml` | Existing | 2 to 3 weeks | Optimization under uncertainty |
| [05 Bayesian season by hub](05-pricing-bayesian-hub-season.md) | `missed-connection-pricing` | Existing | 3 to 4 weeks | Hierarchical Bayesian modelling |
| [06 Sequential testing](06-uplift-sequential-testing.md) | `uplift-analyzer` | Existing | 2 weeks | Experiment design and monitoring |

Plan 03 (the Athens short-term rental freeze) is not written. It depends on archived Inside Airbnb data from before 2025, which must be requested first.

## Suggested order

1. **06 first.** Smallest, no owner action needed, data is reachable, and it warms up the executing model on a repository with clear rules.
2. **04 next.** Small, self-contained, feasibility already counted.
3. **01 as the main project.** Start its owner actions (Kaggle rules, new repository) on day one, because they block everything else in that plan.
4. **05 before the end of February 2027.** Its pre-registration must be pushed before the March 2027 flight data exists.
5. **02 last, or in parallel with 01.** It has no deadline and no data dependency.

## Which model for which part

This is a recommendation and not a rule.

- **A mid-tier model is enough for:** all of 06, all of 04, plan 02 work packages 0 to 3 and 6 to 8, plan 01 work packages 0 to 2 and 7 to 8.
- **Use the strongest model available for:** plan 01 work packages 3 to 6 (estimators whose errors are silent), plan 02 work package 4 (Adtributor from the paper), and all of plan 05 (a repository with frozen files and sealed months, where one wrong edit damages the record).
- **A second model should review, before any sealed read or pre-registration push:** plan 01 work package 6, plan 05 work package 4. The plans mark these points.

## Rules common to all five plans

1. Read the target repository's `CLAUDE.md`, `README.md` and `VALIDATION.md` before writing anything. Its rules win over this file when they differ.
2. One branch per plan, named in the plan. Never push to `main`. Open a pull request only when the owner asks.
3. Design before data. Where a plan says "fixed in writing first", the design section is committed and pushed before the analysis is run. The text to commit is given in the plan, so that the executing model does not have to invent thresholds.
4. Every number in a document comes from a committed results file, and a test checks it.
5. Tests need no network and no real dataset. Each plan names a synthetic generator with known answers.
6. Report a failed expectation as prominently as a passed one, in the same commit as the result. Never retune after a read. Never change a seed, a split or a threshold to improve a result.
7. Prose in the repositories uses plain sentences and full stops in place of dashes and semicolons.
8. Do not include any model identifier in commits, pull requests or code.
9. When a gate is hit, stop and report to the owner with the counts or the error. Do not try alternatives that the plan does not name.

## Owner actions, all plans

| Action | Plan | Blocks |
|---|---|---|
| Accept the rules of the Kaggle competition `expedia-personalized-sort` and make `train.csv` available to the session | 01 | Everything after work package 0 |
| Create the empty public repositories `offer-ranking-position-bias` and `whatchanged` | 01, 02 | The first push |
| Create a PyPI account, enable two-factor login and add the pending trusted publisher | 02 | The release only |
| Say yes or no to the pre-registration of plan 05 before it is pushed | 05 | Work package 4 |
| Run the forward read of plan 05 on the machine that holds the raw flight data, in 2027 | 05 | The final result only |

## What was checked while writing these plans

- The four existing repositories were cloned and read at their heads on 15 November 2026 (hotel `ea3b478`, pricing `3f598cc`, uplift `bb3f01c`, superstore `0456bef`).
- The hotel source file was downloaded and matched its recorded SHA-256. Occupancy counts for plan 04 were computed from it and are quoted there as feasibility counts.
- The Criteo file's host answered. The name `whatchanged` was free on PyPI.
- The Expedia file was not downloaded, because it needs the owner's Kaggle account. Plan 01 therefore states its facts about that file as expectations to verify in its first work package.
