<!-- machine_translated: true -->

<!-- pre-align:aligned sig=fef6db797893 -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > Release Notes { #foundry }

<a id="foundry-release-notes-2026-10-27"></a>
### October 27, 2026 { #foundry-release-notes-2026-10-27 }

<a id="foundry-release-notes-2026-10-27-query"></a>
#### Analysis / Query { #foundry-release-notes-2026-10-27-query }

- You can now run queries that include set operations such as `UNION` and StarRocks-specific syntax.

<a id="foundry-release-notes-2026-10-27-app"></a>
#### App { #foundry-release-notes-2026-10-27-app }

- You can now check, step by step, whether an app is receiving data and producing results from the **Operation Status** column in the App List and the **App Status** tab in the app details.

<a id="foundry-release-notes-2026-10-27-recommendation"></a>
#### Recommendation App { #foundry-release-notes-2026-10-27-recommendation }

- On the **Serving Management** tab, you can select a training version for each model to deploy and view the deployment history.
- You can cancel an ongoing training run for the entire app or on a per-model basis.
- When you resume automatic retraining, any cycles missed during the suspension period are not run immediately — retraining resumes from the next scheduled cycle.

<a id="foundry-release-notes-2026-09-18"></a>
### September 18, 2026 { #foundry-release-notes-2026-09-18 }

<a id="foundry-release-notes-2026-09-18-chart"></a>
#### Analytics / Chart { #foundry-release-notes-2026-09-18-chart }

- If a chart is misconfigured, the reason is displayed on the screen, and a retrieval failure for one chart does not affect other charts.

<a id="foundry-release-notes-2026-09-18-recommendation"></a>
#### Recommendation App { #foundry-release-notes-2026-09-18-recommendation }

- If you pass impressions, interactions, and feedback information in a recommendation API request, the information is reflected in the recommendation results.

<a id="foundry-release-notes-2026-09-18-univariate"></a>
#### Univariate Time Series Anomaly Detection App { #foundry-release-notes-2026-09-18-univariate }

- Added a univariate time series anomaly detection app.

<a id="foundry-release-notes-2026-08-25"></a>
### August 25, 2026 { #foundry-release-notes-2026-08-25 }

<a id="foundry-release-notes-2026-08-25-new-service"></a>
#### New Service Launch { #foundry-release-notes-2026-08-25-new-service }

- NHN Cloud Foundry has been released.
- The following features are available:
    - Data Source: Define a schema to create a data source, load data via file upload or the Ingest API (snapshot upload), and use it for recommendations and analysis.
    - Analysis: Query the loaded data and visualize it with charts and dashboards for analysis.
    - Pipeline: Transform data from a data source through filtering, aggregation, joining, and other operations to produce an analyzable dataset, with support for automatic execution on a batch schedule. The transformed dataset can be used for analysis or recommendation model training.
    - App: Create a Recommendation System app by training a recommendation model on user, item, and interaction data, and use the recommendation results in your service via the recommendation API.