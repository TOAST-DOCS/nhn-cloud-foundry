<!-- machine_translated: true -->

<!-- pre-align:aligned sig=0dde8a7f55cd -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > Release Notes { #foundry }

<a id="foundry.release.notes.2026.09.18"></a>
### September 18, 2026 { #foundry.release.notes.2026.09.18 }

<a id="foundry.release.notes.2026.09.18.chart"></a>
#### Analytics / Chart { #foundry.release.notes.2026.09.18.chart }

- If a chart is misconfigured, the reason is displayed on the screen, and a retrieval failure for one chart does not affect other charts.

<a id="foundry.release.notes.2026.09.18.recommendation"></a>
#### Recommendation App { #foundry.release.notes.2026.09.18.recommendation }

- If you pass impressions, interactions, and feedback information in a recommendation API request, the information is reflected in the recommendation results.

<a id="foundry.release.notes.2026.09.18.univariate"></a>
#### Univariate Time Series Anomaly Detection App { #foundry.release.notes.2026.09.18.univariate }

- Added a univariate time series anomaly detection app.

<a id="foundry.release.notes.2026.08.25"></a>
### August 25, 2026 { #foundry.release.notes.2026.08.25 }

<a id="foundry.release.notes.2026.08.25.new.service"></a>
#### New Service Launch { #foundry.release.notes.2026.08.25.new.service }

- NHN Cloud Foundry is now available.
- The following features are available:
    - Data source: Create a data source by defining a schema, and load data via file upload or the Ingest API (snapshot upload) for use in recommendations and analysis.
    - Analysis: Query loaded data and analyze it by visualizing it with charts and dashboards.
    - Pipeline: Transform data from data sources into analyzable datasets using filtering, aggregation, joining, and more, with support for automatic execution on a batch schedule. The transformed datasets can be used for analysis or recommendation model training.
    - App: Create a recommendation system app trained on user, item, and interaction data, and use the recommendation results in your service via the recommendation API.