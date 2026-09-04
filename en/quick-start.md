<!-- machine_translated: true -->

<!-- pre-align:aligned sig=c85588b241bb -->

<a id="foundry.getting.started"></a>
## Machine Learning > NHN Cloud Foundry > Getting Started { #foundry.getting.started }

This document describes the process of creating an app in NHN Cloud Foundry and using the results.
After completing the prerequisites (applying for the service and preparing data), follow the steps below based on the type of app you want to create.

**Recommended System App**

1. Create a data source
2. Create an app
3. Check the app status
4. Retrieve recommendation results
5. Collect recommendation events

**Univariate Anomaly Detection App**

1. Create a metric data source
2. Transfer metrics
3. Create an app
4. Check detection results

<a id="preparation"></a>
## Prerequisites { #preparation }

<a id="preparation.service.enable"></a>
### Request service { #preparation.service.enable }

NHN Cloud Foundry cannot be enabled directly from the console. To use the service, you must submit a request via [1:1 Inquiry](https://www.nhncloud.com/kr/support/inquiry).

1. Select the organization and project where you want to use the service in the NHN Cloud console.
2. On the **Machine Learning > NHN Cloud Foundry > Status** tab, click the **1:1 Inquiry** button, and submit a request including the resource size that you want.
3. Once the person in charge enables the service for the project, all features become available.

![Request service](../static/images/quick-start/서비스이용신청.png){ height="70%" }

<a id="preparation.data"></a>
### Prepare data { #preparation.data }

To create a recommendation system app, you need the following three CSV data files:

| Data | Required columns | Description |
| --- | --- | --- |
| User table | User ID | User information (additional attribute columns are optional) |
| Item table | Item ID | Item information (additional attribute columns are optional) |
| History table | User ID, Item ID, Timestamp | User-item interaction history (rating and category columns are optional) |

Univariate anomaly detection apps require metric (time series) data sent via the collection API instead of CSV. Refer to 'Creating a Univariate Anomaly Detection App'.

<a id="datasource.create"></a>
## 1. Create a data source { #datasource.create }

Go to the **Machine Learning > NHN Cloud Foundry > Data Source** tab.
For a detailed description of each setting, see 'Create a data source' in the [Console User Guide](../console-user-guide/#datasource.create).

1. Click the **Create data source** button.

    ![Create data source](../static/images/quick-start/데이터소스생성모달1.png){ height="70%" }

2. In Basic Settings, enter the data source name and table name.
3. In Connection Settings, confirm that the data source type is **File Upload**.
4. In Advanced Settings, select the CSV file. If the first row of the file contains column names, check **First row is header**. Enter the primary key field (e.g., `user_id`).
5. Click the **Infer Types** button to automatically populate the schema from the CSV sample. Manually correct any incorrectly inferred types.

    ![Create data source - select file and infer types](../static/images/quick-start/데이터소스생성모달2.png){ height="70%" }

6. Click **Add** to create the data source.
7. Using the same method, create the **User**, **Item**, and **History** data sources.
8. Wait until the status changes to `COMPLETED` in the list.

    ![Data source list](../static/images/quick-start/데이터소스목록.png){ height="70%" }

Prometheus API type for receiving metric (time series) data is created differently. Refer to "Create a Metric Data Source" in "Create a Univariate Anomaly Detection App."

<a id="app.create"></a>
## 2. Create an app { #app.create }

Go to the **Machine Learning > NHN Cloud Foundry > Apps** tab and click the **Create app** button.
For a detailed description of each setting, see 'Create an app' in the [Console User Guide](../console-user-guide/#app.create).

<a id="app.create.basic"></a>
### Basic settings { #app.create.basic }

Enter the app name and description, select **Recommendation system** as the app type, and click **Next**.

![Create app - basic settings](../static/images/quick-start/앱생성화면1.png){ height="70%" }

<a id="app.create.detail"></a>
### Detailed settings { #app.create.detail }

1. Click the **Add model** button to add the model to use. For a new service, we recommend the **Cold User** model; if you have sufficient user behavior history, use **Warm User (Transformer)**.

    ![Create app - model settings](../static/images/quick-start/앱생성화면2.png){ height="70%" }

2. In **Data connection settings** on the model card, select the user, item, and history data sources that you created in step 1.
   Specify the user ID and item ID columns and the timestamp column in the history. Select feature columns only when needed.

    ![Create app - data connection settings](../static/images/quick-start/앱생성화면3.png){ height="70%" }

3. If necessary, connect skill tables and other resources in **Additional settings (Skills)**. Set the Longtail mode in the basic model settings (this includes items with lower popularity in recommendations). When done, click **Next**.

    ![Create app - additional settings](../static/images/quick-start/앱생성화면4.png){ height="70%" }

<a id="app.create.review"></a>
### Final review { #app.create.review }

1. Review the basic settings, model settings, and additional settings that you entered.
2. Click the **Save** button to create the app.

![Create app - final review](../static/images/quick-start/앱생성화면5.png){ height="70%" }

<a id="app.status"></a>
## 3. Check the app status { #app.status }

After the app is created, training and deployment proceed automatically. The status changes through Initializing, Training, Deploying, and Activating before reaching Active.
Wait until the status in the app list changes to Active.

![App list](../static/images/quick-start/앱목록.png){ height="70%" }

For a detailed description of each status value, see 'App status' in the [Console User Guide](../console-user-guide/#app.list.status).

!!! tip "Note"
    Training and deployment immediately after app creation is the process of preparing the app. The first training of the recommendation model runs at the time specified in the batch schedule settings. Until then, even if the recommendation API returns a response, it does not reflect the recommendations of a trained model.

<a id="recommendation.query"></a>
## 4. Retrieve recommendation results { #recommendation.query }

When the app becomes active, you can check recommendation results on the recommendation API call screen in the console, or retrieve recommendation results by calling the recommendation query API.
For a detailed description of each item, see 'Call recommendation API' in the [Console User Guide](../console-user-guide/#app.detail.recommend).

1. In the app list, click the app that you created to go to the **Call recommendation API** tab on the details screen.
2. Enter the user ID and specify the recommendation mode and maximum number of recommendations.
3. Click the **Request recommendations** button to display the rank, item key, and score in the recommendation results. You can also check the total number of results and the response time.

    ![Call recommendation API](../static/images/quick-start/추천API호출.png){ height="70%" }

**Request preview** displays the actual API request JSON composed of the input values. You can copy it using the **Copy** button and use it for API integration development.
For instructions on directly calling the recommendation query API, see 'Recommendation query API' in the [API Guide](../api-guide/#recommendation.api).

The response includes the request identifier (`metadata.requestId`) and the list of recommended items (`recommendations[].itemKey`). These values are used when sending recommendation events in the next step.

On the **App info** tab, you can check the app ID, status, and version used for API calls.

![App info](../static/images/quick-start/앱정보.png){ height="70%" }

<a id="recommendation.event"></a>
## 5. Collect recommendation events { #recommendation.event }

When a user interacts with recommendation results, such as clicking on them, send the event data using the recommendation event API. You can analyze the recommendation success rate using the accumulated event data.
For a detailed description of each request field, see 'Recommendation event API' in the [API Guide](../api-guide/#recommendation.event.api).

```bash
curl -X POST '{URL}/api/v1.0/recommendation-apps/{APP_ID}/events' \
  -H "X-NC-APP-KEY: {APP_KEY}" \
  -H "Content-Type: application/json" \
  -H "X-NHN-Authorization: {AUTH_TOKEN}" \
  -d '{
    "eventType": "CLICK",
    "requestId": "{RecommendApiResponse.body.metadata.requestId}",
    "itemKey": "{RecommendApiResponse.body.recommendations.itemKey}",
    "userId": "{RecommendApiResponse.body.userId}",
    "context": {
      "position": 1,
      "placement": "home_main"
    }
  }'
```

!!! tip "Note"
    After an event API request, it may take up to 10 minutes for the data to be loaded into the dataset.

<a id="univariate"></a>
## Create a Univariate Anomaly Detection App { #univariate }

To automatically find values in metrics that fall outside the normal range, use a univariate anomaly detection app. This is a separate flow from the recommendation system.

<a id="univariate.datasource"></a>
### 1. Create a Metric Data Source { #univariate.datasource }

On the **Machine Learning > NHN Cloud Foundry > Data Source** tab, click the **Create Data Source** button.

1. In the basic settings, enter the data source name and table name.
2. In the connection settings, select **Prometheus API** as the data source type.
3. In the detail settings, specify the series identification label and group label.
    - The schema is fixed, so you do not enter it manually.
4. Click the **Add** button and wait until the status in the list shows `COMPLETED`.

    ![Create metric data source](../static/images/quick-start/지표데이터소스생성.png){ height="70%" }

For a detailed description of each field, refer to the 'Prometheus API Detail Settings' section in the [Console User Guide](../console-user-guide/#datasource.create.detail.prometheus).

<a id="univariate.ingest"></a>
### 2. Send Metrics { #univariate.ingest }

Go to the details view of the data source you created and open the **Collection Method** tab. Use the **Copy** button to copy the endpoint, request headers, and request body example, then send the metrics.

![Collection Method](../static/images/quick-start/수집방법.png){ height="70%" }

```bash
curl -X POST '{URL}/api/v1.0/data-sources/{DATA_SOURCE_ID}/ingest/metrics' \
  -H "X-NC-APP-KEY: {APP_KEY}" \
  -H "X-NHN-Authorization: {AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "metrics": [
      {
        "timestamp": 1776149886528,
        "value": 4.99,
        "labels": [
          { "name": "__name__", "value": "cpu_usage" },
          { "name": "instance_id", "value": "instance-001" }
        ]
      }
    ]
  }'
```

For a detailed description of the request format, see "Metric Collection" in the [API Guide](../api-guide/#metrics.ingest.api).

!!! tip "Note"
    After creating the app, send metrics of the same time series continuously at intervals of one minute or less. If you send them at longer intervals, gaps will occur and the preparation may not complete in Exact mode.

<a id="univariate.app"></a>
### 3. Create an App { #univariate.app }

On the **Machine Learning > NHN Cloud Foundry > App** tab, click the **Create App** button.

1. In the basic settings, enter the app name and description, and select **Univariate Anomaly Detection** as the app type.

    ![Create app - Basic settings](../static/images/quick-start/이상탐지앱생성1.png){ height="70%" }

2. In the detailed settings, select the metric data source you created earlier.
    - Specify the model resources, retraining interval, detection options, and result delivery.

    ![Create app - Detailed settings](../static/images/quick-start/이상탐지앱생성2.png){ height="70%" }

3. In the final review, check the entered information and click the **Save** button.

For a detailed description of each item, refer to "Univariate Anomaly Detection Detailed Settings" in the [Console User Guide](../console-user-guide/#app.create.detail.univariate).

!!! tip "Tips"
    You can create only one univariate anomaly detection app per metric data source. For the result delivery transmission mode, the default Accurate mode is recommended. If you want to receive values immediately before preparation is complete, select Instant mode.

<a id="univariate.result"></a>
### 4. Check Detection Results { #univariate.result }

Click the app you created in the app list to go to the details screen.

1. On the **App Info** tab, check the training status and group status.

    ![Univariate anomaly detection app info](../static/images/quick-start/이상탐지앱정보.png){ height="70%" }

2. On the **Group List** tab, check the group status.
    - Groups are registered after metrics are received, so the list is empty immediately after creating an app.
    - "Waiting for activation" means data is being collected for detection. Once activated, detection results are sent.

    ![Group list](../static/images/quick-start/이상탐지그룹목록.png){ height="70%" }

3. The anomaly score and threshold, which are the detection results, are sent to the specified Prometheus and also stored in the result data source.
4. View the stored results using queries or charts on the **Analysis** tab.

For detailed descriptions of each item, see "Univariate Anomaly Detection App Details" in the [Console User Guide](../console-user-guide/#app.detail.univariate).