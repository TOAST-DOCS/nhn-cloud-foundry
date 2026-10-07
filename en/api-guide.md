<!-- machine_translated: true -->

<!-- pre-align:aligned sig=d3aa4d31c69a -->

<a id="foundry-api-guide"></a>
## Machine Learning > NHN Cloud Foundry > API Guide { #foundry-api-guide }

Describes the APIs provided by NHN Cloud Foundry.

| API | Description |
| --- | --- |
| Ingest API | Ingest data into an existing data source. Supports snapshot file upload, event collection, and indicator collection. |
| Recommendation Query API | Requests recommendation results from a recommendation system app. |
| Recommendation Event API | Collects user reaction events to recommendation results. |

<a id="auth-common"></a>
## Authentication and Common Information { #auth-common }

<a id="auth-common-preparation"></a>
### Prerequisites { #auth-common-preparation }

To use the APIs, you need an **Appkey** and an **authentication token**.

- You can find the Appkey in the **URL & Appkey** menu at the top of the **Machine Learning > NHN Cloud Foundry** page in the NHN Cloud console.
- The APIs use the **gateway-public** endpoint.
- For information on issuing an authentication token (Bearer token in the `X-NHN-Authorization` header), see the [User Access Key Token](/nhncloud/en/public-api/user-access-key-token/) guide.

<a id="auth-common-request"></a>
### Common Request Information { #auth-common-request }

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

<a id="auth-common-response"></a>
### Common Response Information { #auth-common-response }

All APIs respond with HTTP status code `200`, and the response body consists of a `header` and a `body`. Even when a request is rejected or processing fails, the HTTP status code is `200`, so success or failure is determined by `header.isSuccessful`, not by the HTTP status code.

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
| header.isSuccessful | Boolean | Whether the request succeeded. `false` on failure |
| header.resultCode | Integer | Result code. `0` on success; negative error code on failure |
| header.resultMessage | String | Result message. `SUCCESS` on success; error details on failure |
| body | Object/Array | Response data for each API. `null` on failure |

Failure response example:

```json
{
  "header": {
    "isSuccessful": false,
    "resultCode": -4041102,
    "resultMessage": "IngestJob not found."
  },
  "body": null
}
```

Error codes common to all APIs are in [Common Error Codes](#auth-common-error-codes) below, and error codes specific to each API are in the **Error Code** section at the end of each API section.

- Even if the authentication token is missing or expired, the response is returned as HTTP `200` with a failure response.
- If a request fails to reach the service due to a network failure or similar issue, a different HTTP status code may be returned without a `header`. This case is also treated as a failure.

!!! danger "Caution"
    If you determine success based on the HTTP status code, failed responses may be treated as successful. The client must determine success based on `header.isSuccessful`, and identify the cause of failure using `header.resultCode`.

<a id="auth-common-error-codes"></a>
### Common Error Codes { #auth-common-error-codes }

These are error codes that may be returned by all APIs. For error codes specific to each API, refer to the **Error Codes** section at the end of each API section.

The first three digits of an error code correspond to the HTTP status code. Codes in the 400 range indicate a problem with the request — check `resultMessage`, correct the request, and send it again. Codes in the 500 range indicate a problem on the service side — retry the same request after a moment, and if the failure continues, contact Customer Center.

| Code | Message | Description |
| --- | --- | --- |
| -4010000 | Unauthorized | Authentication failed. The authentication token is missing, malformed, or expired, or the appKey that the token points to does not match the request target. Reissue the token and retry the request. |
| -4040000 | Not Found | The requested path does not exist. Check the URI. |
| -4050000 | Method Not Allowed | The path is correct, but the HTTP method is different. |
| -4060000 | Not Acceptable | A JSON response cannot be received with the Accept header. |
| -4150000 | Unsupported Media Type | The Content-Type of the request body is not supported. Send the request with `application/json`. |
| -5000000 | Internal Server Error | An internal server error occurred. Try again after a moment, and if the problem persists, contact Customer Center. |
| -5030101 | Authentication service is not ready. Please retry. | The authentication service is not ready. Try again after a moment. |

<a id="ingest-api"></a>
## Ingest API { #ingest-api }

Ingest API is an API for loading data into a data source that you have already created in the console. The following methods are provided depending on the data source type.

| Method | Target Data Source | Description |
| --- | --- | --- |
| Upload Snapshot | File | Replace all data with the uploaded file |
| Event Collection | File | Add change events one by one while retaining existing data |
| Metric Collection | Prometheus API | Sends metric (time-series) data in real time |

!!! danger "Caution"
    An API for creating new data sources is not provided. To use the Ingest API, you must first create a data source in the console.

<a id="ingest-snapshot"></a>
### Upload Snapshot (File Upload) { #ingest-snapshot }

**Replaces all** data in the data source with the contents of the uploaded file. The upload process consists of three steps.

!!! danger "Caution"
    Uploading a snapshot replaces all data already loaded in the data source. Existing data cannot be recovered.

Upload limits:

- Maximum upload size: **10 GB**
- `100MB` or less → **Single Upload (SINGLE)**
- More than `100MB` → **Multipart Upload (MULTIPART)**
- Use the `formPost` field values **exactly as returned** in the response.

<a id="ingest-snapshot-init"></a>
#### 1. Upload Initialization (init) { #ingest-snapshot-init }

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

<a id="ingest-snapshot-upload-single"></a>
#### 2-A. Single File Upload (100 MB or Less) { #ingest-snapshot-upload-single }

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

<a id="ingest-snapshot-upload-multipart"></a>
#### 2-B. Large File Upload (More than 100 MB, MULTIPART) { #ingest-snapshot-upload-multipart }

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

<a id="ingest-snapshot-complete"></a>
#### 3. Upload Complete (complete) { #ingest-snapshot-complete }

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
| body.jobId | Job ID. Used for [Query Job Status](#ingest-snapshot-job-status). |

<a id="ingest-snapshot-cancel"></a>
#### Cancel Upload { #ingest-snapshot-cancel }

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

<a id="ingest-snapshot-job-status"></a>
#### Query Job Status { #ingest-snapshot-job-status }

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
| body.jobType | Job type. SNAPSHOT (snapshot ingestion) or EVENT (change event) |
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

<a id="event-ingest-api"></a>
### Ingest Events { #event-ingest-api }

Sends change events while retaining existing data. This is used with data sources of the file type, and you must first enable the **Event API**. You can enable it from the event settings tab in the console or by using the activation API below.

!!! danger "Caution"
    Enabling the Event API blocks snapshot uploads. In addition, schema changes are restricted, so you must disable the Event API before making schema changes. For how to enable and disable it, see 'Event Settings' in the [Console User Guide](./console-user-guide/#datasource-detail-event).

<a id="event-ingest-api-enable"></a>
#### Enable/Disable Event API { #event-ingest-api-enable }

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

- Activation is performed asynchronously. Immediately after the request, the response has `enabled` set to false and `status` set to ENABLING. Events are collected only after the status becomes ENABLED.
- If you request activation again on a data source that is already active, the current status is returned as-is. The same applies when you request deactivation on a data source that is already inactive. Calling the API multiple times does not change the result.
- However, if you request activation again while activation is in progress (`ENABLING`), the request is rejected. You can check the progress on the **Event Settings** tab in the console.
- If you call the API with a data source that does not exist, an error is returned indicating that the data source cannot be found.

<a id="event-ingest-api-send"></a>
#### Send a Single Event { #event-ingest-api-send }

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

- `operation` is case-insensitive. If you send a value that is not allowed, the request will be rejected.
- If no primary key field is specified for the data source, only `INSERT` can be sent. `UPDATE` and `DELETE` will be rejected. A data source with a primary key specified can use all three operations.

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

<a id="event-ingest-api-batch"></a>
#### Send Multiple Events { #event-ingest-api-batch }

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

<a id="metrics-ingest-api"></a>
### Metric Collection { #metrics-ingest-api }

Sends metric data to a data source of the Prometheus API type. The sent metrics can be viewed in the Analysis menu and can also be used as input for the univariate time-series anomaly detection app.

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

On success, `header.isSuccessful` is returned as `true` and there is no `body`. If the request is rejected, `header.isSuccessful` is returned as `false`, so use `header` to determine whether the request succeeded.

The collection rules are as follows:

- If required fields are missing, label name or value rules are violated, a single request exceeds 5,000 items, or required headers are missing, the entire request is rejected and no items are saved.
- You can include metrics from multiple time series in a single request. Because time series are distinguished by label combinations, you do not need to send a separate request for each time series.
- Send data for the same time series only once per minute. If you collect data at a shorter interval, combine it into a 1-minute average before sending. If multiple values arrive within the same minute, only the first value received is used for analysis, and the rest are discarded.
- `timestamp` is an epoch value in milliseconds. If you send it in seconds, it will be saved as an incorrect time.
- If you have specified a group label in the Data Source, always include that label when sending data. If the label is missing, the data will not belong to the intended group.
- Items where `value` is NaN or Infinity are skipped and not saved. The remaining items in the same request are processed normally.
- A successful response indicates that the data has been received. The data will be saved after a short delay. If you send the same request again, the same data may be stored in duplicate.
- Data that arrives late is saved, but may be excluded from real-time inference.

!!! tip "Tips"
    Data ingestion is independent of the transmission interval. However, if this data source is connected to a Univariate Time-Series Anomaly Detection app, you must send data points for the same time series one per minute without interruption. Because the app groups metrics in 1-minute intervals for evaluation, sending data at longer intervals can create gaps, which may prevent the accurate mode from completing its preparation.
    Training also has requirements. If the data source has only one time series, training fails, so you must have two or more time series, and each time series must accumulate data continuously for at least approximately 4 hours for training to complete successfully.

<a id="ingest-error-codes"></a>
### Error Codes { #ingest-error-codes }

These are error codes that may be returned across all Ingest APIs, in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description |
| --- | --- | --- |
| -4000001 | Invalid request. | Invalid request format. Missing required fields, value range or format violations, request body JSON data parsing failure, or missing `X-NC-APP-KEY` header. |
| -4041101 | DataSource not found. | No Data Source exists for the specified `dataSourceId`, or the Data Source belongs to a different appKey. |

<a id="ingest-error-codes-snapshot"></a>
#### Snapshot Upload { #ingest-error-codes-snapshot }

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4000202 | Invalid file name. | `fileName` is empty or contains characters other than letters, numbers, `.`, `_`, and `-`. | init, complete |
| -4001107 | File size exceeds maximum limit. | `fileSize` exceeds 10 GB. | init |
| -4001101 | Invalid data source type. | The data source is not a file type. | init |
| -4001103 | DataSource is busy. | The snapshot cannot be uploaded because the data source is loading data or the Event API is enabled. | init, complete |
| -4001104 | Ingest job is already running. | There is a snapshot upload job already in progress for the same data source. Please try again after the job is completed or canceled. | init, complete |
| -4000201 | DataSource is not ready for ingest. | The data source is not in a state where ingestion can start. | complete |
| -4041102 | IngestJob not found. | No job matching the `jobId` was found, or the job belongs to a different appKey. | complete, Cancel Upload, Query Job Status |
| -4001105 | IngestJob is in invalid status. | The job is not in an uploading state. It has already been completed, canceled, or marked as failed due to an upload timeout. Restart from init. | complete |
| -4000203 | File name does not match. | The file name does not match the `fileName` sent during init. | complete |
| -4291101 | Too many requests. Please try again later. | The request cannot be accepted because there are too many jobs waiting to be processed. Please try again later. | init, complete |

<a id="ingest-error-codes-event"></a>
#### Ingest Event { #ingest-error-codes-event }

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4001101 | Invalid data source type. | The Data Source is not a file type. | Enable, Disable |
| -4000201 | DataSource is not ready for ingest. | The Data Source is not in a state ready to receive events. For example, a snapshot upload may be in progress. | Enable, single event, Multiple Events |
| -4001104 | Ingest job is already running. | A snapshot upload job is in progress, so the Data Source cannot be enabled. | Enable |
| -4091103 | Stream API activation is in progress. Please wait. | Activation is in progress. Wait until it is completed. | Enable |
| -4091601 | Operation is in progress. Please wait for the current operation to complete. | Another job is in progress on the same Data Source. Try again after it is completed. | Enable |
| -4000002 | Invalid strategy type. | The Data Source type does not support event ingestion. | single event, Multiple Events |
| -4001109 | Stream API is not enabled. | The Event API is not enabled. Call the activation API first. | single event, Multiple Events |
| -4000204 | Invalid operation. | The `operation` value is not INSERT, UPDATE, or DELETE, or a non-INSERT operation was sent to a Data Source that has no primary key field. | single event |
| -5004001 | Failed to serialize Kafka message. | The event could not be converted to the storage format. | single event |
| -5004002 | Kafka send timeout. | The event was not saved within the timeout period. Please try again in a few minutes. | single event |
| -5004003 | Kafka send failed. | Failed to save the event. Please try again in a few minutes. | single event |
| -5004004 | Kafka send interrupted. | The event save operation was interrupted. Please try again in a few minutes. | single event |

For Multiple Events, per-item errors are returned in `body[].success` and `body[].errorMessage`, not in the `header`. If Data Source validation fails, the entire request is rejected with one of the codes above.

<a id="ingest-error-codes-metrics"></a>
#### Metric Collection { #ingest-error-codes-metrics }

| Code | Message | Description |
| --- | --- | --- |
| -4000001 | Invalid request. | `metrics` is empty or exceeds 5,000 entries, `timestamp`, `value`, or `labels` is missing, a label name or value violates naming rules, or the `X-NC-APP-KEY` header is missing. The entire request is rejected. |
| -4000002 | Invalid strategy type. | The data source type does not support metric collection. |
| -4000201 | DataSource is not ready for ingest. | The data source is not in a state where it can receive metrics. |
| -5004001 | Failed to serialize Kafka message. | Failed to convert the metric to the storage format. |
| -5004004 | Kafka send interrupted. | Metric storage was interrupted. Try again later. |
| -5004005 | Kafka batch send partially failed. | Storage of some metrics failed or timed out. Resending the request may result in duplicate entries for metrics that were already stored. |

<a id="univariate-api"></a>
## Univariate Time Series Anomaly Detection API { #univariate-api }

<a id="univariate-group-api"></a>
### Enable, Disable, and Delete Groups { #univariate-group-api }

Enables, disables, and deletes groups of a univariate time-series anomaly detection app. The three APIs share the same request format and differ only in their paths.

| Method | URI |
| --- | --- |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId` is the app ID displayed in the app details in the console.

cURL example:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/serving-pipelines/{servingPipelineId}/groups/enable" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "groupKey": [
      { "name": "region", "value": ["kr1", "jp1"] }
    ]
  }'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| groupKey | Array | Conditional | List of labels that specify the target group. Used only when group labels are specified in the data source. |
| groupKey[].name | String | O | Group label name. Must exactly match the group label name specified in the data source. |
| groupKey[].value | Array | O | List of values for that label. Each value corresponds to one group. |

On success, `header.isSuccessful` returns `true` and there is no `body`.

The request rules are as follows:

- If no group labels are specified in the data source, do not send `groupKey`. The entire data source is treated as a single group and becomes the target. If `groupKey` is included, the request is rejected.
- If group labels are specified in the data source, `groupKey` is required, and the set of label names sent must exactly match the group labels of the data source. Sending the same label name twice is rejected.
- If there are multiple group labels, values at the same position are paired to form a group. For example, if `["a", "b"]` is sent for `rule_id` and `["q", "w"]` for `instance_id`, the two target groups are `(a, q)` and `(b, w)`. All labels must have the same number of values; otherwise, the request is rejected.
- If a value list is empty, the request is rejected. If the same group is specified more than once, it is processed only once.
- Stopping or deleting an unregistered group returns an error, and other groups included in the request are not processed either.

!!! tip "Note"
    If you did not assign a group label to the data source, the entire data source is registered as a single group when app creation is complete, so this API is not required for the service to function. If you assigned a group label, groups are not registered automatically — you must register the target groups using the Start Service API to receive detection results. This operation is available via API only. You can check the registered groups and their status on the **Group List** tab in the app details view of the console.

!!! danger "Warning"
    Disabling a group does not stop the transmission of detection results. Only the status displayed in the group list changes to disabled.
    A deleted group is permanently removed along with its status history and cannot be recovered.

<a id="univariate-error-codes"></a>
### Error Codes { #univariate-error-codes }

These are error codes that can be returned from the Enable, Disable, and Delete Groups APIs, in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4000001 | Invalid request. | Violation of `groupKey` rules. This occurs when a `groupKey` is sent to a data source that has no group label, a required `groupKey` is missing, the set of label names differs from the group labels of the data source, label names are duplicated, the value list is empty, or the counts differ. The `X-NC-APP-KEY` header is missing. | Enable, Disable, Delete |
| -4041301 | ServingPipeline not found. | No app matches the given `servingPipelineId`, or the app belongs to a different appKey. | Enable, Disable, Delete |
| -4041101 | DataSource not found. | The metric data source connected to the app cannot be found. | Enable, Disable, Delete |
| -4000201 | DataSource is not ready for ingest. | The metric data source is not in a usable state. | Enable, Disable, Delete |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. Try again after the app becomes active. | Enable |
| -4041306 | Group entry not found or has no trainingPipelineId. | One or more of the requested groups are not registered. Other groups included in the request are also not processed. | Disable, Delete |

<a id="recommendation-api"></a>
## Recommendation API { #recommendation-api }

Requests recommendation results from the recommendation system app that you created. If the user's interaction history is sufficient, the server performs model-based inference (Sequential); if the history is insufficient, it performs attribute-based inference (Cold Start).

<a id="recommendation-api-recommend"></a>
### Request Recommendations { #recommendation-api-recommend }

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

- If `options.mode` is not specified, the server determines the inference type. If the app does not have a model for the determined type, the server recommends an alternative type that is integrated with the app instead. The type actually used can be checked in `body.metadata.inferenceType` of the response.
- If the app does not have a model for the requested type and there is no alternative type, HTTP `503` and result code `5030001` are returned. Check which models have been created in the app and whether training has completed, then call again. Requests that specify a type via `options.mode` are not substituted, so you may receive this response.

<a id="recommendation-api-signal"></a>
#### Behavior Signals { #recommendation-api-signal }

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
- To store reaction events for analysis, use the [Recommendation Event API](#recommendation-event-api) together.

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

<a id="recommendation-error-codes"></a>
### Error Codes { #recommendation-error-codes }

Error codes that may be returned in recommendation requests, in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description |
| --- | --- | --- |
| -4004201 | Invalid request. or reason for rejection | Request format error. Possible causes include a missing `userId`, `maxRecommendations` less than 1, violation of the minimum recommendation count rule for a category, a category not in the catalog, violation of the format or count of behavior signals in `context`, or a missing `X-NC-APP-KEY` header. The `resultMessage` contains the reason for rejection. |
| -4044201 | Recommendation app not found. | No recommendation app was found for the specified `appId`, or the app does not match `X-NC-APP-KEY`. |
| -4001301 | Invalid model type. | The `appId` does not belong to a Recommendation System app. |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. Wait until the app status becomes active, and then try again. |
| -5034201 | Recommendation model is not ready. | The model for the requested recommendation mode has not yet been integrated into the app. Wait until the first training and deployment are complete, and then try again. |

<a id="recommendation-event-api"></a>
## Recommendation Event API { #recommendation-event-api }

Collects user interaction events (such as clicks) in response to recommendation results. You can analyze the recommendation success rate using the collected event data.

<a id="recommendation-event-api-send"></a>
### Send Recommendation Event { #recommendation-event-api-send }

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

<a id="recommendation-event-error-codes"></a>
### Error Codes { #recommendation-event-error-codes }

In addition to the [common error codes](#auth-common-error-codes), the following error codes may be returned when sending recommendation events.

| Code | Message | Description |
| --- | --- | --- |
| -4004202 | Invalid event request. | Invalid request format. Returned when `eventType` is empty, exceeds 64 characters, contains characters other than letters, numbers, or `_`, or is a reserved word (REQUEST, RESPONSE); when `requestId` is empty or exceeds 128 characters; when `itemKey` is missing; or when a required header is missing. |
| -4044201 | Recommendation app not found. | No recommendation app exists for the specified `appId`, or the app does not match `X-NC-APP-KEY`. |
| -4001301 | Invalid model type. | The `appId` does not belong to a Recommendation System app. |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. |
| -4094201 | Recommendation event dataset is not configured for this app. | No data source is configured for the app to store recommendation events. |
| -5034202 | Event publish failed. | Failed to save the event. Please try again later. |
| -5044201 | Event publish timed out. | The event could not be saved within the timeout period. Please try again later. |