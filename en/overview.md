<!-- machine_translated: true -->

<!-- pre-align:aligned sig=0eb4ce54bb90 -->

<a id="foundry.overview"></a>
## Machine Learning > NHN Cloud Foundry > Overview { #foundry.overview }

NHN Cloud Foundry is a service that integrates customer data and uses machine learning models (such as recommendation, time-series forecasting/anomaly detection, and structured data classification/numerical prediction) to support effective decision-making.
You can load data, process it through a visual workflow, and then analyze it using queries, charts, and dashboards, or create an app connected to a recommendation model to use recommendation results in your service.

<a id="main.feature"></a>
## Main features { #main.feature }

| Feature | Description |
| --- | --- |
| **Data Source** | A unit for storing data to be analyzed. You can create a data source by defining a schema and loading data into it; for data sources that have already been created, you can add or update data using the Ingest API. You can also create a data source that receives metric (time-series) data in real time. |
| **Pipeline** | Transforms data from a data source into a dataset for analysis or model training by processing it through a node-connected workflow. Supports automatic execution based on a batch schedule. |
| **Analysis** | Query data using SQL **queries**, visualize it with **charts**, and monitor it comprehensively using **dashboards**. |
| **App** | Create and manage apps by connecting AI models to data. Two types available: recommendation system and univariate anomaly detection |

<a id="datasource"></a>
## Data source { #datasource }

A data source is a unit for storing data to be analyzed in NHN Cloud Foundry.
When you create a data source by defining a schema, the data is loaded into a table and can then be used in pipelines, analysis, and apps.

Data sources are created in the console, and you can upload data at the same time.
To add or update data in an existing data source, use the Ingest API.
Two methods are provided: snapshot upload, which replaces all data, and event method, which adds new data while retaining existing data.

When working with metric data, create a Prometheus API type data source and send data in real time via the collection API. The metrics loaded this way are used as input for the univariate anomaly detection app.

!!! danger "Caution"
    Do not enter information that contains personal data when using this service.
    This service does not provide separate security measures for personal data entered by customers, so refrain from entering or storing information that contains personal data.

<a id="pipeline"></a>
## Pipeline { #pipeline }

This feature processes data from a data source through a visual workflow with connected nodes and transforms it into an analyzable dataset.
The transformed dataset is registered as a data source and can be used as training data for analysis or for the recommendation model in an app.

- Source data connection and automatic schema detection
- Transformation operations such as row filtering, column processing, aggregation, join, and union
- Automatic execution based on batch schedule settings
- Computing resource configuration and execution history management

<a id="analysis"></a>
## Analysis { #analysis }

You can query, visualize, and monitor data stored in data sources.
Use it by checking data with queries, creating charts, and arranging them on a dashboard.

| Feature | Description |
| --- | --- |
| Query | Query data from data sources using SQL and manage execution history |
| Chart | Visualize queried data |
| Dashboard | Place multiple charts on a single screen for integrated monitoring |

<a id="app"></a>
## App { #app }

AI models can be connected to data to create and manage apps. Two app types are provided: recommendation system and univariate anomaly detection.

The **Recommendation System** lets you select a recommendation model to use and connect user, item, and history data sources — training and deployment then proceed automatically. Once the app is active, you can request the recommendation API.
You can check recommendation results by calling the API directly from the console or by sending API requests. When you collect user interactions through the recommendation event API, you can analyze the recommendation success rate using the loaded event data.
You can also change the training cycle, stop or resume automatic retraining, run training manually, and view training artifact history from the console.

**Univariate anomaly detection** learns each time series from collected metrics and detects values that fall outside the normal range.
The anomaly scores and threshold values from the detection results are sent to the specified Prometheus, and are also stored in the result data source for use in analysis.

<a id="public.api"></a>
## API { #public.api }

NHN Cloud Foundry provides APIs in addition to the console.
You can use the Ingest API to load snapshots, events, and metrics into a data source that you have already created, as well as APIs to request recommendation results from a created app and to send user reaction events.

For more information, see the [API Guide](../api-guide/).

<a id="target"></a>
## Target users { #target }

- Teams that need a data loading/processing/analysis environment without building their own data infrastructure
- Services that need to consolidate scattered data in one place, process it regularly, and track it through metrics
- Teams that want to apply recommendation results to their service without a separate model training and serving environment
- Teams that want to automatically detect abnormalities in metrics being collected