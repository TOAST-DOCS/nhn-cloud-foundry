<!-- machine_translated: true -->

<!-- pre-align:aligned sig=0eb4ce54bb90 -->

<a id="foundry.overview"></a>
## Machine Learning > NHN Cloud Foundry > Overview { #foundry.overview }

NHN Cloud Foundry is a service that integrates customer data and uses machine learning models (such as recommendation, time-series forecasting/anomaly detection, and structured data classification/numerical prediction) to support effective decision-making.
You can load data, process it through a visual workflow, and then analyze it using queries, charts, and dashboards, or create an app connected to a recommendation model to use recommendation results in your service.

!!! danger "Caution"
    Do not enter any information that contains personal data when using this service.
    This service does not provide separate security measures for personal data entered by customers. Refrain from entering or storing any information that contains personal data.

<a id="main.feature"></a>
## Main features { #main.feature }

| Feature | Description |
| --- | --- |
| **Data source** | A unit for storing data to be analyzed. You can define a schema to create a data source and load data. For an existing data source, you can add or update data using the Ingest API. |
| **Pipeline** | Transforms data from a data source into a dataset for analysis or model training by processing it through a node-connected workflow. Supports automatic execution based on a batch schedule. |
| **Analysis** | Query data using SQL **queries**, visualize it with **charts**, and monitor it comprehensively using **dashboards**. |
| **App** | Create and manage recommendation system apps by connecting a recommendation model to data. Recommendation results can be checked through the console and API. |

<a id="datasource"></a>
## Data source { #datasource }

A data source is a unit for storing data to be analyzed in NHN Cloud Foundry.
When you create a data source by defining a schema, the data is loaded into a table and can then be used in pipelines, analysis, and apps.

Data sources are created in the console, and you can upload data at the same time.
To add or update data in an existing data source, use the Ingest API.
Two methods are provided: snapshot upload, which replaces all data, and event method, which adds new data while retaining existing data.

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

This feature allows you to create and manage recommendation system apps by connecting a recommendation model to data.
When you select a recommendation model and connect user, item, and history data sources, training and deployment proceed automatically. Once the app becomes active, you can use the recommendation results.

Recommendation results can be retrieved directly from the console or requested via API. By collecting user responses through the recommendation event API, you can analyze the recommendation success rate using the accumulated event data.

<a id="public.api"></a>
## API { #public.api }

NHN Cloud Foundry provides APIs in addition to the console.
You can use the Ingest API to add or update data in an existing data source, and APIs to request recommendation results from a created app and send user response events.

For more information, see the [API Guide](../api-guide/).

<a id="target"></a>
## Target users { #target }

- Teams that need a data loading, processing, and analysis environment without building their own data infrastructure
- Services that want to consolidate scattered data in one place, process it regularly, and monitor it through metrics
- Teams that want to apply recommendation results to their service without a separate model training and serving environment