<!-- machine_translated: true -->

<!-- pre-align:aligned sig=403401ac9ba3 -->

<a id="foundry.api.guide"></a>
## Machine Learning > NHN Cloud Foundry > API Guide { #foundry.api.guide }

Describes the APIs provided by NHN Cloud Foundry.

| API | Description |
| --- | --- |
| Ingest API | Collects data into an already created data source. Provides snapshot file upload, event collection, and indicator collection. |
| Recommendation Query API | Requests recommendation results from a recommendation system app. |
| Recommendation Event API | Collects user reaction events to recommendation results. |

<a id="auth.common"></a>
## Authentication and Common Information { #auth.common }

<a id="auth.common.preparation"></a>
### Prerequisites { #auth.common.preparation }

To use the APIs, you need an **Appkey** and an **authentication token**.

- You can find the Appkey in the **URL & Appkey** menu at the top of the **Machine Learning > NHN Cloud Foundry** page in the NHN Cloud console.
- The APIs use the **gateway-public** endpoint.
- For information on issuing an authentication token (Bearer token in the `X-NHN-Authorization` header), see the [User Access Key Token](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/user-access-key-token/) guide.

<a id="auth.common.request"></a>
### Common Request Information { #auth.common.request }

Required headers:

```plaintext
X-NC-APP-KEY: {appKey}
X-NHN-Authorization: Bearer {ACCESS_TOKEN}
Content-Type: application/json
```

Base URL:

```plaintext
https://{gateway-public-host}/api/v1.0
```

<a id="auth.common.response"></a>
### Common Response Information { #auth.common.response }

All API responses consist of a `header` and a `body`.

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {}
}
```

| Field | Type | Description |
| --- | --- | --- |
| header.isSuccessful | Boolean | Whether the request succeeded |
| header.resultCode | Integer | Result code. 0 for success; error code for failure. |
| header.resultMessage | String | Result message. SUCCESS for success; error details for failure. |
| body | Object/Array | Response data for each API |

<a id="ingest.api"></a>
## Ingest API { #ingest.api }

Ingest API is an API for loading data into a data source that you have already created in the console. The following methods are provided depending on the data source type.

| Method | Target Data Source | Description |
| --- | --- | --- |
| Upload Snapshot | File | Replace all data with the uploaded file |
| Event Collection | File | Add change events one by one while retaining existing data |
| Metric Collection | Prometheus API | Sends metric (time-series) data in real time |

!!! danger "Caution"
    An API for creating new data sources is not provided. To use the Ingest API, you must first create a data source in the console.

<a id="ingest.snapshot"></a>
### Snapshot Upload (File Upload) { #ingest.snapshot }

**Replaces all** data in the data source with the contents of the uploaded file. The upload process consists of three steps.

!!! danger "Caution"
    Snapshot upload replaces all data already loaded in the data source. Existing data cannot be recovered.

Upload limits:

- Maximum upload size: **10 GB**
- `100MB` or less → **Single Upload (SINGLE)**
- More than `100MB` → **Multipart Upload (MULTIPART)**
- Use the `formPost` field values **exactly as returned** in the response.

<a id="ingest.snapshot.init"></a>
#### 1. Upload Initialization (init) { #ingest.snapshot.init }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init |

Issues a signed temporary URL for uploading large files directly to storage. Returns a single URL (SINGLE) or multipart URLs (MULTIPART) depending on the file size.

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "fileName": "data.csv",
    "fileSize": 52428800,
    "contentType": "text/csv"
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| fileName | String | O | File name. Allowed characters: letters, numbers, period (.), underscore (_), hyphen (-) |
| fileSize | Long | O | File size (bytes). Minimum 1, maximum 10 GB |
| contentType | String | X | Content-Type (default: application/octet-stream) |

Response example (SINGLE):

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "jobId": "550e8400-e29b-41d4-a716-446655440000",
    "uploadType": "SINGLE",
    "uploadUrl": "{upload-url}",
    "uploadId": null,
    "partSize": null,
    "parts": null,
    "expiresAt": "2025-01-20T11:00:00Z",
    "formPost": {
      "objectPrefix": "{appKey}/{dataSourceId}/snapshot/{jobId}/",
      "signature": "{SIGNATURE}",
      "expires": 1737370800,
      "maxFileSize": 10737418240,
      "maxFileCount": 1
    }
  }
}
```

Response example (MULTIPART):

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "jobId": "550e8400-e29b-41d4-a716-446655440000",
    "uploadType": "MULTIPART",
    "uploadUrl": null,
    "uploadId": "{appKey}/{dataSourceId}/snapshot/{jobId}/data.csv_segments/",
    "partSize": 104857600,
    "parts": [
      {
        "partNumber": 1,
        "uploadUrl": "{part-upload-url}?signature={SIG}&expires={TS}&max_file_size=104857600&max_file_count=1",
        "headUrl": "{part-head-url}?temp_url_sig={SIG}&temp_url_expires={TS}"
      }
    ],
    "expiresAt": "2025-01-20T11:00:00Z",
    "formPost": {
      "objectPrefix": "{appKey}/{dataSourceId}/snapshot/{jobId}/",
      "signature": "{SIGNATURE}",
      "expires": 1737370800,
      "maxFileSize": 104857600,
      "maxFileCount": 1
    }
  }
}
```

!!! tip "Note"
    The MULTIPART response also includes `formPost`. However, since multipart uploads transmit parts using the query parameters (`signature`/`expires`/`max_file_size`/`max_file_count`) from `parts[].uploadUrl`, `formPost` is for reference only and is not used for the actual part uploads.

| Field | Description |
| --- | --- |
| body.jobId | Job ID. Used in subsequent complete/status query requests. |
| body.uploadType | Upload type. SINGLE (100 MB or less) or MULTIPART (more than 100 MB) |
| body.uploadUrl | Upload URL (for single upload) |
| body.uploadId | Multipart upload ID (for multipart upload) |
| body.partSize | Part size (bytes, for multipart upload) |
| body.parts[].partNumber | Part number (starting from 1) |
| body.parts[].uploadUrl | Part upload URL |
| body.parts[].headUrl | URL for retrieving ETag (HEAD request after upload completes) |
| body.expiresAt | URL expiration time |
| body.formPost.objectPrefix | Object prefix (path prepended to the file name) |
| body.formPost.signature | HMAC-SHA1 signature |
| body.formPost.expires | Expiration time (UNIX timestamp) |
| body.formPost.maxFileSize | Maximum file size (bytes) |
| body.formPost.maxFileCount | Maximum file count |

<a id="ingest.snapshot.upload.single"></a>
#### 2-A. Single File Upload (100 MB or Less) { #ingest.snapshot.upload.single }

Send a multipart/form-data POST request to the `uploadUrl` from the init response.
This request is sent directly to Object Storage, so no separate authentication is required (the `signature` serves as authentication).

curl example:

```bash
curl -X POST "{uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={formPost.maxFileSize}" \
  -F "max_file_count={formPost.maxFileCount}" \
  -F "expires={formPost.expires}" \
  -F "signature={formPost.signature}" \
  -F "file=@./data.csv;filename=data.csv"
```

!!! danger "Caution"
    The `file` field must be added **last** in the form data. On success, you will receive an HTTP `201 Created` response.

<a id="ingest.snapshot.upload.multipart"></a>
#### 2-B. Large File Upload (More than 100 MB, MULTIPART) { #ingest.snapshot.upload.multipart }

Receive the `parts[]` array from the response and upload each part individually.
Each part is processed in the following order: **(1) upload → (2) retrieve ETag with HEAD request → (3) collect into `partETags[]` in ascending `partNumber` order**.

1. Split the file into chunks of `partSize` (default 100 MB).
2. For each part, parse the query parameters (`signature`, `expires`, `max_file_size`, `max_file_count`) from `parts[i].uploadUrl` and send as multipart/form-data (field name `file`, fixed file name `part`).
3. After a successful upload, send a `HEAD` request to `parts[i].headUrl` to collect the `ETag` value from the response header.
4. When all parts are complete, assemble the `partETags` array in ascending `partNumber` order and include it in the upload complete request.

curl example for part upload:

```bash
# 1) Upload
curl -X POST "{parts[i].uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={max_file_size-from-query}" \
  -F "max_file_count={max_file_count-from-query}" \
  -F "expires={expires-from-query}" \
  -F "signature={signature-from-query}" \
  -F "file=@./part_i.bin;filename=part"

# 2) Retrieve ETag
curl -I "{parts[i].headUrl}" | grep -i '^etag:'
```

<a id="ingest.snapshot.complete"></a>
#### 3. Upload Complete (complete) { #ingest.snapshot.complete }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curl example (single upload):

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "jobId": "550e8400-e29b-41d4-a716-446655440000",
    "fileName": "data.csv"
  }'
```

curl example (multipart upload):

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "jobId": "550e8400-e29b-41d4-a716-446655440000",
    "fileName": "data.csv",
    "uploadId": "{multipart-upload-id}",
    "partETags": ["etag-part1", "etag-part2", "etag-part3"]
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| jobId | String | O | Job ID (the jobId from the init response) |
| fileName | String | O | File name |
| uploadId | String | X | Multipart upload ID (required only for multipart uploads) |
| partETags | Array | X | List of ETags per part (required only for multipart uploads, in partNumber order) |

Response example:

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "jobId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

| Field | Description |
| --- | --- |
| body.jobId | Job ID. Used for [Query Job Status](#ingest.snapshot.job.status). |

<a id="ingest.snapshot.cancel"></a>
#### Cancel Upload { #ingest.snapshot.cancel }

| Method | URI |
| --- | --- |
| DELETE | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl example (single upload):

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl example (multipart upload) - pass `uploadId` as a query parameter:

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest.snapshot.job.status"></a>
#### Query Job Status { #ingest.snapshot.job.status }

| Method | URI |
| --- | --- |
| GET | /api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curl example:

```bash
curl "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

Response example:

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "jobId": "550e8400-e29b-41d4-a716-446655440000",
    "dataSourceId": "ds-001",
    "jobType": "SNAPSHOT",
    "status": "COMPLETED",
    "obsFilePath": "s3a://{bucket}/{path}/file.csv",
    "statistics": {
      "totalRecords": 10000,
      "failedRecords": 5,
      "successfulRecords": 9995,
      "successRate": 0.9995
    },
    "errorMessage": null,
    "createdDatetime": "2025-01-20T10:00:00Z",
    "startedDatetime": "2025-01-20T10:01:00Z",
    "completedDatetime": "2025-01-20T10:05:00Z",
    "modifiedDatetime": "2025-01-20T10:05:00Z"
  }
}
```

| Field | Description |
| --- | --- |
| body.jobId | Job ID |
| body.dataSourceId | Target data source ID |
| body.jobType | Job type. SNAPSHOT (snapshot load) or EVENT (change event) |
| body.status | Job status. See the status values below. |
| body.obsFilePath | OBS file path |
| body.statistics.totalRecords | Total record count |
| body.statistics.failedRecords | Failed record count |
| body.statistics.successfulRecords | Successful record count |
| body.statistics.successRate | Success rate (0.0 to 1.0) |
| body.errorMessage | Error message (on failure) |
| body.createdDatetime | Job creation time |
| body.startedDatetime | Job start time |
| body.completedDatetime | Job completion time |
| body.modifiedDatetime | Last modified time |

The job status (`status`) can have the following values:

| Value | Description |
| --- | --- |
| UPLOADING | Uploading file |
| QUEUED | Upload complete, waiting to load |
| STAGED | Ready to process |
| RUNNING | Loading data |
| COMPLETED | Job completed successfully |
| FAILED | Job failed |

<a id="event.ingest.api"></a>
### Ingest Events { #event.ingest.api }

Sends change events while retaining existing data. This is used with data sources of the file type, and you must first enable the **Event API**. You can enable it from the event settings tab in the console or by using the activation API below.

!!! danger "Caution"
    Enabling the Event API blocks snapshot uploads. In addition, modifying the data source schema (adding catalog fields) is restricted while the Event API is enabled, so you must disable the Event API before adding fields. For instructions on how to enable and disable the Event API, see "Event Settings" in the [Console User Guide](../console-user-guide/#datasource.detail.event).

<a id="event.ingest.api.enable"></a>
#### Enable/Disable Event API { #event.ingest.api.enable }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/enable |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/disable |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/events/enable" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

Response example:

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "enabled": false,
    "status": "ENABLING"
  }
}
```

| Field | Description |
| --- | --- |
| body.enabled | Whether event collection is available |
| body.status | Activation status. DISABLED, ENABLING, ENABLED, ENABLE_FAILED |

- Activation is processed asynchronously. Immediately after the request, the response shows `enabled` as false and `status` as ENABLING. Events are collected after the status becomes ENABLED.
- If you request activation again while activation is in progress, the request is rejected. You can check the progress on the Event Settings tab in the console.

<a id="event.ingest.api.send"></a>
#### Send a Single Event { #event.ingest.api.send }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/events" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "operation": "INSERT",
    "data": {
      "userId": "user-12345",
      "courseId": "course-java-101",
      "action": "enroll",
      "rating": 4.5
    },
    "eventTimestamp": "2026-08-25T10:30:00Z"
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| operation | String | O | Operation type. One of INSERT, UPDATE, or DELETE |
| data | Object | O | Event data. Uses the field names of the data source schema as keys |
| eventTimestamp | String | X | Timestamp of the event. If omitted, the server's received time is used |

Response example:

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "eventId": "evt-550e8400-e29b-41d4-a716-446655440000",
    "success": true,
    "errorMessage": null
  }
}
```

| Field | Description |
| --- | --- |
| body.eventId | Event ID |
| body.success | Whether the processing was successful |
| body.errorMessage | Error message on failure |

<a id="event.ingest.api.batch"></a>
#### Send Multiple Events { #event.ingest.api.batch }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/batch |

Sends multiple change events at once. You can send up to 5,000 events per request.

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/events/batch" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "events": [
      {
        "operation": "INSERT",
        "data": { "hostname": "server-01", "portName": "eth0", "trafficIn": 1024.5 },
        "eventTimestamp": "2026-08-25T10:30:00Z"
      },
      {
        "operation": "UPDATE",
        "data": { "hostname": "server-01", "portName": "eth1", "trafficIn": 2048.7 }
      }
    ]
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| events | Array | O | List of events. Up to 5,000 events per request. Each item's fields are the same as those for sending a single event. |

The `body` of the response is an array of processing results for each event.

<a id="metrics.ingest.api"></a>
### Metric Collection { #metrics.ingest.api }

Sends metric data to a data source of type Prometheus API. The transmitted metrics are used as input for the univariate anomaly detection app.

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/metrics |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/metrics" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "metrics": [
      {
        "timestamp": 1776149886528,
        "value": 4.99,
        "labels": [
          { "name": "__name__", "value": "cpu_usage" },
          { "name": "instance_id", "value": "instance-001" }
        ],
        "metadata": { "resourceType": "Instance" }
      }
    ]
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| metrics | Array | O | List of metrics. Cannot be empty; maximum of 5,000 items per request |
| metrics[].timestamp | Long | O | Metric timestamp. Millisecond epoch |
| metrics[].value | Double | O | Measured value |
| metrics[].labels | Array | O | List of labels. The combination of labels determines the time series, and group labels determine the group |
| metrics[].labels[].name | String | O | Label name. Must start with an alphabetic character or _, and can only contain alphabetic characters, numbers, and _ |
| metrics[].labels[].value | String | O | Label value. Commas and equals signs cannot be used |
| metrics[].metadata | Object | X | Additional information. Stored and delivered as-is without interpretation. The identityKey key is reserved for system use and cannot be used |

Returns HTTP `202 Accepted` on success.

The collection rules are as follows:

- A single request can contain metrics from multiple time series. Because time series are distinguished by label combinations, you do not need to split requests by time series.
- `timestamp` is a millisecond-unit epoch. If you send it in seconds, it will be stored with an incorrect timestamp.
- If you have specified group labels for the data source, always include those labels when sending data. If a label is missing, the data will not belong to the intended group.
- Items where `value` is NaN or Infinity are skipped and not stored. The remaining items in the same request are processed normally.
- A `202` response indicates that the data has been received. Storage is reflected shortly after, and resending the same request may result in duplicate data being stored.
- Data that is sent with a delay is stored, but may be excluded from real-time inference.

!!! tip "Tips"
    Loading is independent of the transmission interval. However, if this data source is connected to a univariate anomaly detection app, you must send data for the same time series continuously at intervals of one minute or less. Because the app evaluates metrics in one-minute buckets, sending data at longer intervals creates gaps and may prevent preparation from completing in precision mode.

<a id="recommendation.api"></a>
## Recommendation API { #recommendation.api }

Requests recommendation results from the recommendation system app that you created. If the user's interaction history is sufficient, the server performs model-based inference (Sequential); if the history is insufficient, it performs attribute-based inference (Cold Start).

<a id="recommendation.api.recommend"></a>
### Request Recommendations { #recommendation.api.recommend }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/recommend |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/recommendation-apps/{appId}/recommend" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user_12345",
    "context": {
      "currentItemKey": "CONT0001",
      "recentlyViewed": ["CONT0010", "CONT0023"],
      "pageType": "course_detail",
      "sessionId": "session_abc123"
    },
    "options": {
      "maxRecommendations": 10
    }
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| userId | String | O | ID of the user to receive recommendations. To request recommendations for an anonymous user, specify an empty string (""). |
| context.currentItemKey | String | X | Key of the item currently being viewed |
| context.recentlyViewed | Array | X | List of recently viewed item keys |
| context.availableItems | Array | X | List of item keys eligible for recommendation. If specified, only items in this list are recommended. |
| context.pageType | String | X | Current page type (free-form; for example: home, item_detail) |
| context.sessionId | String | X | Session ID |
| context.impressions | Array | X | List of items exposed to the user as recommendation results |
| context.interactions | Array | X | Information about actions performed by the user on an item |
| context.feedback | Array | X | Evaluation left by the user on the item |
| userAttributes | Object | X | User attribute information (used for Cold Start inference) |
| options.maxRecommendations | Integer | X | Maximum number of recommendations (from 1 to 100). Values exceeding 100 are adjusted to 100 without an error; if not specified, 100 is applied. If fewer items are available than this value, only the actual number of available items is returned. |
| options.mode | String | X | Specifies the inference mode. One of: sequential (history-based), cold_start (attribute-based), popular (popularity-based). If not specified, the server determines this automatically. |
| options.longtail | Boolean | X | Improves recommendation diversity by including less popular items. Applies only when sequential is used. |
| options.excludeItemKeys | Array | X | List of item keys to exclude from recommendations. Excluded items are not counted toward the maximum number of recommendations. |

<a id="recommendation.api.signal"></a>
#### Behavior Signals { #recommendation.api.signal }

`context.impressions` is used to reorder recommendation results based on the recommendation information exposed to the user.
`context.interactions` and `context.feedback` are fields that convey actions the user took on the recommendation results, and reflect user behavior-based data into model inference.

```json
{
  "userId": "user_12345",
  "context": {
    "impressions": [
      {
        "requestId": "req_xyz789",
        "itemKeys": ["CONT0023", "CONT0045"],
        "occurredAt": "2026-08-25T10:00:00+09:00"
      }
    ],
    "interactions": [
      {
        "requestId": "req_xyz789",
        "itemKey": "CONT0023",
        "type": "CLICK",
        "occurredAt": "2026-08-25T10:00:05+09:00"
      }
    ],
    "feedback": [
      {
        "requestId": "req_xyz789",
        "itemKey": "CONT0045",
        "type": "NEGATIVE",
        "occurredAt": "2026-08-25T10:00:10+09:00"
      }
    ]
  }
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| requestId | String | O | The body.metadata.requestId of the recommendation response in which the action occurred |
| itemKeys | Array | O | List of exposed item keys. Enter in the order of exposure; used in impressions |
| itemKey | String | O | Target item key. Used in interactions and feedback |
| type | String | O | interactions uses CLICK or CONVERSION. feedback uses POSITIVE or NEGATIVE |
| occurredAt | String | O | The time at which the action occurred |

- `occurredAt` must be sent in ISO 8601 format with a timezone offset. Requests without an offset are treated as errors.
- All three fields require `requestId`, `occurredAt`, and an item key to be used as signals.
- Each field must be sent in chronological order, from oldest to most recent.
- `impressions` allows a maximum of 10 entries, with up to 100 `itemKeys` per entry. `interactions` and `feedback` each allow a maximum of 10 entries per `type`. Requests that exceed these limits are rejected.
- Behavior signals are used only as inference input for the current recommendation request and are not stored. If the `feedback` for the same item changes, only the most recent value is reflected, so resend the data with every request to maintain the effect.
- To store reaction events for analysis, use the [Recommendation Event API](#recommendation.event.api) together.

!!! tip "Note"
    The collection method and field types of the `userAttributes` schema may change in the future depending on the implementation direction of Preference Elicitation.

Response example:

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": {
    "userId": "user_12345",
    "recommendations": [
      { "itemKey": "CONT0023", "score": 0.95, "position": 1 },
      { "itemKey": "CONT0045", "score": 0.89, "position": 2 }
    ],
    "metadata": {
      "modelVersion": "v1.2.0",
      "requestId": "req_xyz789",
      "inferenceType": "sequential",
      "abTestGroup": ""
    }
  }
}
```

| Field | Description |
| --- | --- |
| body.userId | The ID of the requesting user |
| body.recommendations[].itemKey | Recommended item key |
| body.recommendations[].score | Recommendation score (0.0–1.0) |
| body.recommendations[].position | Recommendation rank |
| body.metadata.modelVersion | Version of the model used |
| body.metadata.requestId | Request tracking ID. Use this value when sending to the Recommendation Event API |
| body.metadata.inferenceType | Inference type. sequential (history-based), cold_start (attribute-based), popular (popularity-based) |
| body.metadata.abTestGroup | A/B test group (currently returns an empty value) |

<a id="recommendation.event.api"></a>
## Recommendation Event API { #recommendation.event.api }

Collects user interaction events (such as clicks) in response to recommendation results. You can analyze the recommendation success rate using the collected event data.

<a id="recommendation.event.api.send"></a>
### Send Recommendation Event { #recommendation.event.api.send }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/events |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/recommendation-apps/{appId}/events" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "eventType": "CLICK",
    "requestId": "req_xyz789",
    "itemKey": "CONT0023",
    "userId": "user_12345",
    "context": {
      "sessionId": "sess_abc",
      "placement": "home_main"
    }
  }'
```

Pass the values received from the recommendation query API response as-is for `requestId`, `itemKey`, and `userId`.

| Field | Required | Description |
| --- | --- | --- |
| eventType | O | Event type. Can be defined freely (e.g., CLICK, PURCHASE, IMPRESSION). Only English letters, numbers, and underscores (_) are allowed (^[A-Za-z0-9_]+$), up to 64 characters. Stored in uppercase regardless of the case entered. REQUEST and RESPONSE are reserved words and cannot be used. |
| requestId | O | The body.metadata.requestId value from the recommendation API response (opaque string, up to 128 characters) |
| itemKey | O | The itemKey of the recommended item that the user interacted with |
| userId | X | The body.userId value from the recommendation API response |
| context | X | Additional event information (free-form key-value pairs; e.g., display position, placement) |
| userAttributes | X | User attribute information (free-form key-value pairs) |
| options | X | Additional options (free-form key-value pairs) |

A successful response (200) returns only the `header`.

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  }
}
```

!!! tip "Note"
    - A successful response (200) indicates that the collection pipeline has received the event, but does not guarantee that the data has been loaded into the analytics table.
    - It may take up to 10 minutes for data to be loaded into the dataset after an event API request.
    - Retrying after a timeout may result in duplicate entries of the same event. Consider deduplication when analyzing the data.