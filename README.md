# Dimitris Bechrakis

### Applied Data Science · ML Engineering · Commercial Analytics

Statistics graduate · MSc Data Science in progress · Athens, Greece

I work as a Business Analyst in **Sales Ancillary at Etraveli Group**. I turn metric changes into testable business questions, build reproducible data and ML workflows, and ship them as tested services, not just notebooks.

**What changed? Which segment explains it? What should we do next?**

## Explore the work

| Commercial analytics | Applied AI |
|---|---|
| [![Superstore profitability explorer](https://raw.githubusercontent.com/dbechrakis/superstore-shipping-region-analysis/main/docs/portfolio-overview.jpg)](https://dbechrakis.github.io/superstore-shipping-region-analysis/) **[Superstore Profitability Explorer](https://github.com/dbechrakis/superstore-shipping-region-analysis)** · [Live demo](https://dbechrakis.github.io/superstore-shipping-region-analysis/) · [Method](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/docs/margin-driver-method.md) · [SQL case](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/sql/README.md) | [![Steam review explorer](https://raw.githubusercontent.com/dbechrakis/steam-reviews-nlp-rag/main/outputs/figures/live_app_gpt_oss.png)](https://dbechrakis-steam-explorer.streamlit.app/) **[Steam Review Intelligence](https://github.com/dbechrakis/steam-reviews-nlp-rag)** · [Live demo](https://dbechrakis-steam-explorer.streamlit.app/) · [Architecture](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/docs/architecture.md) · [Retrieval benchmark](https://github.com/dbechrakis/steam-reviews-nlp-rag/blob/main/evaluation/README.md) |
| Central trails West by **7.02 percentage points** of margin. An exact decomposition identifies a **−7.75 pp within-product margin** component; product mix partly offsets the gap. | Semantic retrieval, reranking and inspectable player reviews across **41,170 reviews / 241 games**, also served as a **Dockerised retrieval API**. A CI gate reruns an 80-question benchmark with the real models and blocks regressions. Sentiment modelling reached **0.887 macro F1** on a held-out-game test set. |

| Decision ML | Data engineering |
|---|---|
| [![Hotel cancellation decision app](https://raw.githubusercontent.com/dbechrakis/hotel-booking-cancellation-ml/main/docs/live-app-scoring.jpg)](https://dbechrakis-hotel-cancellation.streamlit.app/) **[Hotel Cancellation Decision System](https://github.com/dbechrakis/hotel-booking-cancellation-ml)** · [Live demo](https://dbechrakis-hotel-cancellation.streamlit.app/) · [BA case](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/docs/business-analysis-case.md) · [Validation](https://github.com/dbechrakis/hotel-booking-cancellation-ml/blob/main/VALIDATION.md) | [![Movie analytics Streamlit dashboard on a live TMDB snapshot](https://raw.githubusercontent.com/dbechrakis/movie-analytics-data-pipeline/main/docs/dashboard-preview.png)](https://github.com/dbechrakis/movie-analytics-data-pipeline#snapshot-results-2026-10-05) **[Movie Analytics Pipeline](https://github.com/dbechrakis/movie-analytics-data-pipeline)** · [Snapshot results](https://github.com/dbechrakis/movie-analytics-data-pipeline#snapshot-results-2026-10-05) · [Architecture](https://github.com/dbechrakis/movie-analytics-data-pipeline/blob/main/docs/architecture.md) · [Validation](https://github.com/dbechrakis/movie-analytics-data-pipeline/blob/main/VALIDATION.md) |
| A chronological holdout of **23,989 later bookings**, served as a **FastAPI + Docker** scoring service with an MLflow model registry and **drift monitoring**. A replay of later bookings shows the monitor would have flagged the shift behind a recall shortfall from week three, months before outcomes confirmed it. **Isotonic calibration** brings probability calibration error to 0.013. | TMDB API → PostgreSQL → tested dbt models → filter-aware Streamlit dashboard. A live run on 2026-10-05 loaded **100 top-grossing movies** and passed **11 of 11 dbt tests**. The image shows the dashboard from that run. The sample is limited to blockbusters. |

| Forecasting & uncertainty | Deep learning |
|---|---|
| [![AAPL interval coverage over time](https://raw.githubusercontent.com/dbechrakis/aapl-forecasting-backtest/main/outputs/backtest/rolling_coverage.png)](https://github.com/dbechrakis/aapl-forecasting-backtest) **[AAPL Forecasting Backtest](https://github.com/dbechrakis/aapl-forecasting-backtest)** · [Results](https://github.com/dbechrakis/aapl-forecasting-backtest#results-aapl-2705-daily-origins-jan-2016--oct-2026) · [Live forecasting design](https://github.com/dbechrakis/aapl-forecasting-backtest#live-forecasting-in-production) | [![N-BEATS hourly forecasts on M4](https://raw.githubusercontent.com/dbechrakis/m4-nbeats-forecasting/main/outputs/hourly/example_forecasts.png)](https://github.com/dbechrakis/m4-nbeats-forecasting) **[N-BEATS vs the M4 Benchmarks](https://github.com/dbechrakis/m4-nbeats-forecasting)** · [Results](https://github.com/dbechrakis/m4-nbeats-forecasting#results-on-the-official-m4-test-horizon) · [Model code](https://github.com/dbechrakis/m4-nbeats-forecasting/blob/main/src/deepforecast/nbeats.py) |
| A rolling-origin backtest over **2,705 daily origins** with leakage tests and Diebold–Mariano significance tests. No model significantly beats the random walk, and gradient boosting is significantly worse. **GARCH-conformal** intervals have the best interval score and hold 21-day coverage in calm and volatile markets alike. A scheduled job issues and scores forecasts after every US close. | N-BEATS implemented from scratch in **PyTorch**, trained as one global model per frequency on **5,000 M4 series** and scored with the official OWA. It beats the best statistical benchmark by **30% on Hourly** and **20% on Weekly** (bootstrap intervals exclude zero), and finds **no gain on Daily**, where series are close to a random walk. |

## What runs in production

| Project | Serving pattern | Automated checks |
|---|---|---|
| Hotel cancellation | Real-time REST API (FastAPI, Docker), MLflow registry, calibrated probabilities, score logging and PSI drift monitoring | Every push: image built, container queried, API score must equal artifact score |
| Steam reviews | Retrieval API in Docker with the models baked in and offline at runtime | On retrieval changes and weekly: 80-question benchmark regression gate, container queried end to end |
| AAPL forecasting | Scheduled batch forecast (GitHub Actions) with a public track record | Every push: look-ahead tests (changing future prices cannot change any past forecast) |
| Movie analytics | API → PostgreSQL → dbt → dashboard pipeline | 11 dbt data tests on each pipeline run; unit tests on every push |

These projects started from different problems. Superstore and Steam originated in academic teams; the repositories record contribution and validation limits. Historical model scores and scenarios are not claims of commercial impact.

## How I build

- **Analyse:** SQL, Python/pandas, Excel, Power BI/DAX, Qlik Sense, Looker.
- **Engineer:** PostgreSQL, dbt, APIs, Git, reproducible tests and CI.
- **Model:** scikit-learn, PyTorch (N-BEATS), gradient boosting, NLP, embeddings, retrieval, time-series backtesting, GARCH volatility, probability calibration, conformal prediction.
- **Ship:** FastAPI, Docker, MLflow, GitHub Actions (CI gates and scheduled jobs), Streamlit.

I care about the population behind each KPI, reproducible calculations, honest validation (including the results where the fancy model loses), and the limits of observational evidence.

[Connect on LinkedIn](https://www.linkedin.com/in/dimitrisbechrakis/)
