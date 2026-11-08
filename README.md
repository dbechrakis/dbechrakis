# Dimitris Bechrakis

### Business Analyst · Commercial Analytics & Experimentation · Applied Data Science

Statistics graduate · MSc Data Science in progress · Athens, Greece · [LinkedIn](https://www.linkedin.com/in/dimitrisbechrakis/) · [Email](mailto:bechrakisd@gmail.com)

I work as a Business Analyst in **Sales Ancillary at Etraveli Group**. I turn metric changes into testable business questions, build reproducible data and ML workflows, and ship them as tested services, not just notebooks.

**What changed? Which segment explains it? What should we do next?**

## Flagship: Missed-Connection Protection Pricing

[![Missed-connection pricing demo: chance of a miss, claim cost, price and the offer decision for a connection](https://raw.githubusercontent.com/dbechrakis/missed-connection-pricing/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/missed-connection-pricing/)

**[Missed-Connection Protection Pricing](https://github.com/dbechrakis/missed-connection-pricing)** · [Live demo](https://dbechrakis.github.io/missed-connection-pricing/) · [Decision memo](https://github.com/dbechrakis/missed-connection-pricing/blob/main/docs/decision-memo.md) · [Validation](https://github.com/dbechrakis/missed-connection-pricing/blob/main/VALIDATION.md) · [Pre-registration](https://github.com/dbechrakis/missed-connection-pricing/blob/main/PREREGISTRATION_2026H2_V3.md)

Should an online travel agency sell protection against missed flight connections, at what price, and where should it refuse? Built from public US data only (BTS on-time and delay causes, DB1B tickets, GSA hotel rates) for five hubs.

- Prices fixed before the test ran a **60.5% loss ratio against a 65% target** on **37.1M connections** the model had never seen. One flat price would have run **117%** if risky connections buy more.
- An audit of my own assumptions found airlines already pay for much of what the cover pays. So the product was re-scoped to the gap they leave: hotel nights after weather or air-traffic disruptions, **40.6% to 72.5%** of misses.
- That gap is a correlated weather risk. Months ran **0.84 to 1.37×** expected claims and no model choice removed it, so the margin, not the model, has to carry it.
- I listed 14 limits of my own work and worked through them: **5 closed, 7 reduced to one named input only an agency can supply, 2 open** (demand needs customers, the clean test needs the data to exist). As it would be sold, the cover ran a **75% loss ratio over 26 backtest months against a 65% target**, so I report that, not just the average.
- The next test is **pre-registered and tagged before its data exists** (September to December 2026), with every earlier look at the data disclosed.

## Featured work

*Superstore and Uplift demos open instantly (GitHub Pages). The Hotel and Steam apps run on free Streamlit hosting and can take about a minute to wake up if idle.*

| Commercial analytics | Experimentation & causal inference |
|---|---|
| [![Superstore profitability explorer](https://raw.githubusercontent.com/dbechrakis/superstore-shipping-region-analysis/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/superstore-shipping-region-analysis/) **[Superstore Profitability Explorer](https://github.com/dbechrakis/superstore-shipping-region-analysis)** · [Live demo](https://dbechrakis.github.io/superstore-shipping-region-analysis/) · [Method](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/docs/margin-driver-method.md) · [SQL case](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/sql/README.md) | [![Uplift Analyzer: effect estimates, randomization checks and targeting value](https://raw.githubusercontent.com/dbechrakis/uplift-analyzer/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/uplift-analyzer/) **[Uplift Analyzer](https://github.com/dbechrakis/uplift-analyzer)** · [Live explorer](https://dbechrakis.github.io/uplift-analyzer/) · [Case study](https://github.com/dbechrakis/uplift-analyzer/blob/main/docs/case-study-imperfect-randomization.md) · [Validation record](https://github.com/dbechrakis/uplift-analyzer/blob/main/VALIDATION.md) |
| *MSc team project (3 people). I added the SQL case, Python companion, margin decomposition and explorer.*<br>Central trails West by **7.02 percentage points** of margin. An exact decomposition identifies a **−7.75 pp within-product margin** component. Product mix partly offsets the gap. | Incrementality on the **13.98M-user Criteo randomized-test file**. The raw effect on visits is **+1.03 pp** and **+0.77 pp** after covariate adjustment, because assignment is slightly predictable from the features. A local LLM answers only through validated tools and never computes a number. |

| Decision ML | Applied AI |
|---|---|
| [![Hotel cancellation decision app](https://raw.githubusercontent.com/dbechrakis/hotel-booking-cancellation-ml/main/docs/live-app-scoring.jpg)](https://dbechrakis-hotel-cancellation.streamlit.app/) **[Hotel Cancellation Decision System](https://github.com/dbechrakis/hotel-booking-cancellation-ml)** · [Live demo](https://dbechrakis-hotel-cancellation.streamlit.app/) · [BA case](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/docs/business-analysis-case.md) · [Validation](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/VALIDATION.md) | [![Steam review explorer](https://raw.githubusercontent.com/dbechrakis/steam-reviews-nlp-rag/main/outputs/figures/live_app_gpt_oss.png)](https://dbechrakis-steam-explorer.streamlit.app/) **[Steam Review Intelligence](https://github.com/dbechrakis/steam-reviews-nlp-rag)** · [Live demo](https://dbechrakis-steam-explorer.streamlit.app/) · [Architecture](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/docs/architecture.md) · [Retrieval benchmark](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/evaluation/README.md) |
| A chronological holdout of **23,989 later bookings**, served as a **FastAPI + Docker** scoring service with an MLflow registry and **drift monitoring**. **Isotonic calibration** brings probability calibration error to 0.013. | *MSc team project. I own notebooks 03 to 05 (embeddings, DistilBERT, two-stage retrieval); the API, CI gate and evaluation are my portfolio edition.*<br>Semantic retrieval and reranking over **41,170 reviews / 241 games**, served as a **Dockerised retrieval API**. A CI gate reruns an 80-question benchmark with the real models and blocks regressions. A claim-level check judged **88.5%** of 399 answer sentences supported, found **40%** carry no citation, and is [audited and limited](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/evaluation/README.md#answer-evaluation-results-2026-10-30). |

### More work

- **[AAPL Forecasting Backtest](https://github.com/dbechrakis/aapl-forecasting-backtest)** · [Results](https://github.com/dbechrakis/aapl-forecasting-backtest#results-aapl-2705-daily-origins-jan-2016-to-oct-2026). A rolling-origin backtest over **2,705 daily origins** with leakage tests and Diebold-Mariano tests. No model significantly beats the random walk. **GARCH-conformal** intervals have the best interval score.
- **[N-BEATS on M4 Hourly, Weekly and Daily](https://github.com/dbechrakis/m4-nbeats-forecasting)** · [Results](https://github.com/dbechrakis/m4-nbeats-forecasting#results-on-the-official-m4-test-horizon). N-BEATS in **PyTorch**, scored with the official OWA. It beats the best statistical benchmark by **30% on Hourly** and **20% on Weekly**, and finds **no gain on Daily**.
- **[Movie Analytics Pipeline](https://github.com/dbechrakis/movie-analytics-data-pipeline)** · [Sampling comparison](https://github.com/dbechrakis/movie-analytics-data-pipeline#first-comparison-2026-10-31). TMDB API → PostgreSQL → tested dbt models → Streamlit. Picking films by revenue inflates ROI. Switching to the most-voted films of each year cut the median ROI from **5.79 to 3.25** on a live run, with **13 of 13 dbt tests** passing. One dated run, stated limits.
- **[World Bank Indicators](https://github.com/dbechrakis/world-bank-indicators-analysis)** · [Key findings](https://github.com/dbechrakis/world-bank-indicators-analysis#key-findings). Only 32 of 203 countries produce nuclear power, and economy size predicts which ones (AUC 0.91). A test of whether NMF and PARAFAC find real structure says mostly not: they barely beat shuffled columns, and a country's own average predicts held-out cells better.

## How each project is served and checked

| Project | Serving pattern | Automated checks |
|---|---|---|
| Missed-connection pricing | One price table behind a static demo, a CLI and a FastAPI service in a read-only, non-root Docker image; monthly recalibration job | Every push: 244 tests incl. leakage guards, 450 container quotes must equal the code's, frozen model hashes, pip-audit. Weekly: watches BTS for the pre-registered months |
| Hotel cancellation | Real-time REST API (FastAPI, Docker), MLflow registry, calibrated probabilities, score logging and PSI drift monitoring | Every push: image built, container queried, API score must equal artifact score |
| Steam reviews | Retrieval API in Docker with the models baked in and offline at runtime | On retrieval changes and weekly: 80-question benchmark regression gate, container queried end to end |
| AAPL forecasting | Scheduled batch forecast (GitHub Actions) with a public track record | Every push: look-ahead tests (changing future prices cannot change any past forecast) |
| Movie analytics | API → PostgreSQL → dbt → dashboard pipeline, monthly snapshot refresh | 13 dbt data tests on each pipeline run. Unit tests and an end-to-end synthetic pipeline job on every push |
| Uplift Analyzer | Static explorer on GitHub Pages plus a hardened Docker stack (nginx, Streamlit, local llama.cpp) | Every push: tests, pip-audit, Trivy image scan, compose smoke test. Weekly: 61-question LLM regression gate |

These projects started from different problems. Superstore and Steam originated in academic teams. The repositories record contribution and validation limits. Historical model scores and scenarios are not claims of commercial impact.

## How I build

- **Analyse:** SQL, Python/pandas, Excel, Power BI/DAX, plus Qlik Sense and Looker at work.
- **Engineer:** PostgreSQL, DuckDB and Parquet, dbt, APIs, Git, reproducible tests and CI.
- **Model:** scikit-learn, PyTorch (N-BEATS), gradient boosting, NLP, embeddings, retrieval, time-series backtesting, GARCH volatility, probability calibration, conformal prediction, uplift modeling, instrumental variables (CACE), covariate adjustment, power analysis, pre-registered validation.
- **Ship:** FastAPI, Docker (hardened, Trivy, pip-audit), MLflow, GitHub Actions (CI gates and scheduled jobs), Streamlit, local LLMs with tool calling (llama.cpp).

I care about the population behind each KPI, reproducible calculations, honest validation (including the results where the fancy model loses), and the limits of observational evidence.

[Connect on LinkedIn](https://www.linkedin.com/in/dimitrisbechrakis/)
