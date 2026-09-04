<!-- machine_translated: true -->

<!-- pre-align:aligned sig=0dde8a7f55cd -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > Release Notes { #foundry }

<a id="foundry.release.notes.2026.09.18"></a>
### September 18, 2026 { #foundry.release.notes.2026.09.18 }

<a id="foundry.release.notes.2026.09.18.feature"></a>
#### Added Features { #foundry.release.notes.2026.09.18.feature }

- Added the **Univariate Anomaly Detection** type to apps. It calculates anomaly scores and threshold values from collected metrics, sends them to a specified Prometheus, and also stores them in the result data source.
- Added a **Prometheus API** type data source that receives metrics in real time.
- Added an **Event Settings** tab to the data source details view. When you enable the Event API, you can collect change events while retaining existing data.
- Added a **Training Management** tab to the recommendation system app details. It supports changing the training cycle, stopping and resuming automatic retraining, running training, and viewing artifact history.
- Added **Resource Check** to the app creation screen. You can check whether an app can be created before creating it.
- Added behavioral signals (`impressions`, `interactions`, `feedback`) to the `context` of the recommendation query API.

<a id="foundry.release.notes.2026.09.18.improvement"></a>
#### Feature Updates { #foundry.release.notes.2026.09.18.improvement }

- If a chart query fails, the error message returned by the query engine is now displayed on the screen.
- The data source in the chart list is now displayed by name instead of ID.
- The data source settings on the chart editing screen are now displayed in a locked state.
- The date and time displayed in the console are now unified based on the time zone of the browser used to access it.
- The request constraints for the Metric Collection API have been revised.

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