<!-- machine_translated: true -->

<!-- pre-align:aligned sig=2d6933bd5d2d -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > Release Notes { #foundry }

<!-- TODO: After the release date is confirmed, update the date in the title and anchor below, then delete this comment. -->

<a id="foundry.release.notes.2026.09.xx"></a>
### September XX, 2026 { #foundry.release.notes.2026.09.xx }

<a id="foundry.release.notes.2026.09.xx.feature"></a>
#### Added Features { #foundry.release.notes.2026.09.xx.feature }

- Added the **Univariate Anomaly Detection** type to apps. It trains on collected metrics for each time series, calculates anomaly scores and threshold values, sends them to the specified Prometheus, and stores the results in the Data Source.
- Added a **Prometheus API** type Data Source that receives metrics (time series) data in real time. You can check the transfer method on the **Collection Method** tab in the Data Source details.
- Added an **Event Settings** tab to the Data Source details. When you enable the Event API, you can collect change events via API while retaining existing data.
- Added a **Training Management** tab to the recommendation system app details. You can change the training interval, stop or resume automatic retraining, run training manually, and view the training output history.
- Added **Resource Check** to the app creation screen. You can verify whether an app can be created before creating it.
- Added behavioral signals (`impressions`, `interactions`, `feedback`) to the `context` of the recommendation retrieval API. The signals provided are used to adjust the order of recommendation results.

<a id="foundry.release.notes.2026.09.xx.improvement"></a>
#### Feature Updates { #foundry.release.notes.2026.09.xx.improvement }

- Improved the chart view so that when a chart query fails, the error message returned by the query engine is displayed on the screen.
- The data source column in the chart list now displays the name instead of the ID.
- The data source settings in the chart edit screen are displayed in a locked state.
- Unified the dates and times displayed in the console to the time zone of the browser used to access it.
- Revised the request constraints (maximum number of items, required fields, and label rules) and error responses for the Metric Collection API.

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