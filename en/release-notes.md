<!-- machine_translated: true -->

<!-- pre-align:aligned sig=bbdb9424b3ef -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > Release Notes { #foundry }

<a id="foundry-release-notes-2026-10-27"></a>
### October 27, 2026 { #foundry-release-notes-2026-10-27 }

<a id="foundry-release-notes-2026-10-27-query"></a>
#### Analysis / Query { #foundry-release-notes-2026-10-27-query }

- Added support for running queries that include set operations such as `UNION` and StarRocks-specific syntax.

<a id="foundry-release-notes-2026-10-27-app"></a>
#### App { #foundry-release-notes-2026-10-27-app }

- Added the ability to check, step by step, whether an app is receiving data and producing results in the **Operation Status** column of the App List and on the **App Status** tab in App Details.

<a id="foundry-release-notes-2026-10-27-recommendation"></a>
#### Recommendation App { #foundry-release-notes-2026-10-27-recommendation }

- Added the ability to select and deploy a training version by model on the **Serving Management** tab, and to view the deployment history.
- Added the ability to cancel ongoing training for the entire app or by individual model.
- When Automatic Retraining is resumed, missed cycles during the suspension period are not run immediately — retraining starts from the next scheduled cycle.

<a id="foundry-release-notes-2026-10-27-api"></a>
#### API { #foundry-release-notes-2026-10-27-api }

API v2.0 is now available. The version in the request path is `/api/v2.0`. Compared to v1.0, the HTTP status codes and error codes in responses have changed as follows:

- All APIs respond with HTTP status code `200` even when a request fails. Use `header.isSuccessful` to determine whether the request succeeded.
- The error code (`header.resultCode`) is now a negative number. A value of `0` indicates success, and errors are represented as a 7-digit negative number where the three digits after `-` correspond to the HTTP status code.
- A [common error codes](./api-guide/#auth-common-error-codes) table and per-API error code tables have been added to the API guide.
- Support for API v1.0 will be discontinued.

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