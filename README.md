# Dimitrios Bechrakis

### Business Analyst | Commercial Analytics · Data Products · Applied Data Science

Statistics graduate · MSc Data Science (in progress) · Athens, Greece

I work in **Sales Ancillary at Etraveli Group**, analysing commercial performance and turning metric changes into questions a team can act on. My background is in statistics and banking; my portfolio extends that decision-focused work into data pipelines, predictive modelling and NLP applications.

**What changed? Which segment explains it? What should we do next?**

## Start with these projects

| Business question | End-to-end evidence |
|---|---|
| **[Why is regional profit margin lower?](https://github.com/dbechrakis/superstore-shipping-region-analysis)** · Commercial analytics | Central trails West by **7.02 pp**. A reconciled decomposition shows that category/product mix slightly offsets the gap; margins *within* the same products account for **−7.75 pp**. [Explore the live dashboard](https://dbechrakis-superstore.bechrakisd.chatgpt.site) · [Inspect the method](https://github.com/dbechrakis/superstore-shipping-region-analysis/blob/main/docs/margin-driver-method.md). |
| **[How do raw API records become trustworthy metrics?](https://github.com/dbechrakis/movie-analytics-data-pipeline)** · Data engineering | TMDB API → PostgreSQL → tested dbt staging/mart → filter-aware Streamlit dashboard. Includes data contracts and regression tests; the live external-data pipeline was **not** rerun in the portfolio review. |
| **[Which hotel bookings merit retention review?](https://github.com/dbechrakis/hotel-booking-cancellation-ml)** · Decision ML | A leakage-aware chronological holdout of **23,989 later bookings**, a reproducible Logistic Regression artifact and a Streamlit threshold simulator. Intervention value is an **assumption-based scenario**, not measured uplift. |
| **[What are players saying across thousands of reviews?](https://github.com/dbechrakis/steam-reviews-nlp-rag)** · Applied AI | Sentiment modelling reached **0.887 macro F1** on **24,342 held-out-game reviews**. A [live review explorer](https://dbechrakis-steam-explorer.streamlit.app/) retrieves and reranks evidence from **41,170 reviews across 241 games**, with inspectable citations and optional generated answers. |

Together, these projects show how I move from a business question to a defined metric, a tested data workflow, and a usable decision surface. The Superstore and Steam projects originated in academic teams; the linked repositories describe my contribution and the validation limits. Model scores and historical comparisons are not claims of commercial impact.

## Try the live explorers

| Regional profitability | Steam review intelligence |
|---|---|
| [![Superstore profitability explorer](https://raw.githubusercontent.com/dbechrakis/superstore-shipping-region-analysis/main/docs/portfolio-overview.jpg)](https://dbechrakis-superstore.bechrakisd.chatgpt.site) | [![Steam review explorer with cited evidence](https://raw.githubusercontent.com/dbechrakis/steam-reviews-nlp-rag/main/outputs/figures/live_app_gpt_oss.png)](https://dbechrakis-steam-explorer.streamlit.app/) |

## Tools and approach

- **Analysis:** SQL · Python/pandas · Excel · Power BI/DAX · Qlik Sense · Looker
- **Build:** PostgreSQL · dbt · APIs · Git · Streamlit
- **Model:** scikit-learn · NLP · embeddings · retrieval

I care about the population behind a KPI, reproducible calculations, honest holdout design and the limits of what observational data can tell us.

[Connect on LinkedIn](https://www.linkedin.com/in/dimitrisbechrakis/)
