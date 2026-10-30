# Dimitris Bechrakis

### Applied Data Science · ML Engineering · Commercial Analytics

Statistics graduate · MSc Data Science in progress · Athens, Greece

I work as a Business Analyst in **Sales Ancillary at Etraveli Group**. I turn metric changes into testable business questions, build reproducible data and ML workflows, and ship them as tested services, not just notebooks.

**What changed? Which segment explains it? What should we do next?**

## Featured work

*Live demos run on free hosting. If an app has been idle, the first load can take a minute while it wakes up.*

| Applied AI | Decision ML |
|---|---|
| [![Steam review explorer](https://raw.githubusercontent.com/dbechrakis/steam-reviews-nlp-rag/main/outputs/figures/live_app_gpt_oss.png)](https://dbechrakis-steam-explorer.streamlit.app/) **[Steam Review Intelligence](https://github.com/dbechrakis/steam-reviews-nlp-rag)** · [Live demo](https://dbechrakis-steam-explorer.streamlit.app/) · [Architecture](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/docs/architecture.md) · [Retrieval benchmark](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/evaluation/README.md) | [![Hotel cancellation decision app](https://raw.githubusercontent.com/dbechrakis/hotel-booking-cancellation-ml/main/docs/live-app-scoring.jpg)](https://dbechrakis-hotel-cancellation.streamlit.app/) **[Hotel Cancellation Decision System](https://github.com/dbechrakis/hotel-booking-cancellation-ml)** · [Live demo](https://dbechrakis-hotel-cancellation.streamlit.app/) · [BA case](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/docs/business-analysis-case.md) · [Validation](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/VALIDATION.md) |
| Semantic retrieval and reranking over **41,170 reviews / 241 games**, served as a **Dockerised retrieval API**. A CI gate reruns an 80-question benchmark with the real models and blocks regressions. A claim-level check judged **88.5%** of 381 answer sentences supported, found **41%** carry no citation, and is [audited and limited](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/evaluation/README.md#answer-evaluation-results-2026-10-30). | A chronological holdout of **23,989 later bookings**, served as a **FastAPI + Docker** scoring service with an MLflow registry and **drift monitoring**. **Isotonic calibration** brings probability calibration error to 0.013. |

| Experimentation & causal inference | Commercial analytics |
|---|---|
| [![Uplift Analyzer: effect estimates, randomization checks and targeting value](https://raw.githubusercontent.com/dbechrakis/uplift-analyzer/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/uplift-analyzer/) **[Uplift Analyzer](https://github.com/dbechrakis/uplift-analyzer)** · [Live explorer](https://dbechrakis.github.io/uplift-analyzer/) · [Case study](https://github.com/dbechrakis/uplift-analyzer/blob/main/docs/case-study-imperfect-randomization.md) · [Validation record](https://github.com/dbechrakis/uplift-analyzer/blob/main/VALIDATION.md) | [![Superstore profitability explorer](https://raw.githubusercontent.com/dbechrakis/superstore-shipping-region-analysis/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/superstore-shipping-region-analysis/) **[Superstore Profitability Explorer](https://github.com/dbechrakis/superstore-shipping-region-analysis)** · [Live demo](https://dbechrakis.github.io/superstore-shipping-region-analysis/) · [Method](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/docs/margin-driver-method.md) · [SQL case](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/sql/README.md) |
| Incrementality on the **13.98M-user Criteo randomized-test file**. The raw effect on visits is **+1.03 pp** and **+0.77 pp** after covariate adjustment, because assignment is slightly predictable from the features. A local LLM answers only through validated tools and never computes a number. | Central trails West by **7.02 percentage points** of margin. An exact decomposition identifies a **−7.75 pp within-product margin** component. Product mix partly offsets the gap. |

### More work

- **[AAPL Forecasting Backtest](https://github.com/dbechrakis/aapl-forecasting-backtest)** · [Results](https://github.com/dbechrakis/aapl-forecasting-backtest#results-aapl-2705-daily-origins-jan-2016-to-oct-2026). A rolling-origin backtest over **2,705 daily origins** with leakage tests and Diebold-Mariano tests. No model significantly beats the random walk. **GARCH-conformal** intervals have the best interval score.
- **[N-BEATS on M4 Hourly, Weekly and Daily](https://github.com/dbechrakis/m4-nbeats-forecasting)** · [Results](https://github.com/dbechrakis/m4-nbeats-forecasting#results-on-the-official-m4-test-horizon). N-BEATS in **PyTorch**, scored with the official OWA. It beats the best statistical benchmark by **30% on Hourly** and **20% on Weekly**, and finds **no gain on Daily**.
- **[Movie Analytics Pipeline](https://github.com/dbechrakis/movie-analytics-data-pipeline)** · [Snapshot results](https://github.com/dbechrakis/movie-analytics-data-pipeline#snapshot-results-2026-10-05). TMDB API → PostgreSQL → tested dbt models → Streamlit. A live run loaded **100 top-grossing movies** and passed **11 of 11 dbt tests**. The sample is limited to blockbusters.
- **[World Bank Indicators](https://github.com/dbechrakis/world-bank-indicators-analysis)** · [Key findings](https://github.com/dbechrakis/world-bank-indicators-analysis#key-findings). Only 32 of 203 countries produce nuclear power, and economy size predicts which ones (AUC 0.91). A test of whether NMF and PARAFAC find real structure says mostly not: they barely beat shuffled columns, and a country's own average predicts held-out cells better.

## How each project is served and checked

| Project | Serving pattern | Automated checks |
|---|---|---|
| Hotel cancellation | Real-time REST API (FastAPI, Docker), MLflow registry, calibrated probabilities, score logging and PSI drift monitoring | Every push: image built, container queried, API score must equal artifact score |
| Steam reviews | Retrieval API in Docker with the models baked in and offline at runtime | On retrieval changes and weekly: 80-question benchmark regression gate, container queried end to end |
| AAPL forecasting | Scheduled batch forecast (GitHub Actions) with a public track record | Every push: look-ahead tests (changing future prices cannot change any past forecast) |
| Movie analytics | API → PostgreSQL → dbt → dashboard pipeline | 11 dbt data tests on each pipeline run. Unit tests on every push |
| Uplift Analyzer | Static explorer on GitHub Pages plus a hardened Docker stack (nginx, Streamlit, local llama.cpp) | Every push: tests, pip-audit, Trivy image scan, compose smoke test. Weekly: 61-question LLM regression gate |

These projects started from different problems. Superstore and Steam originated in academic teams. The repositories record contribution and validation limits. Historical model scores and scenarios are not claims of commercial impact.

## How I build

- **Analyse:** SQL, Python/pandas, Excel, Power BI/DAX, Qlik Sense, Looker.
- **Engineer:** PostgreSQL, dbt, APIs, Git, reproducible tests and CI.
- **Model:** scikit-learn, PyTorch (N-BEATS), gradient boosting, NLP, embeddings, retrieval, time-series backtesting, GARCH volatility, probability calibration, conformal prediction, uplift modeling, instrumental variables (CACE), covariate adjustment, power analysis.
- **Ship:** FastAPI, Docker (hardened, Trivy, pip-audit), MLflow, GitHub Actions (CI gates and scheduled jobs), Streamlit, local LLMs with tool calling (llama.cpp).

I care about the population behind each KPI, reproducible calculations, honest validation (including the results where the fancy model loses), and the limits of observational evidence.

[Connect on LinkedIn](https://www.linkedin.com/in/dimitrisbechrakis/)
