<!-- machine_translated: true -->

<!-- pre-align:aligned sig=a293d4083b4d -->

<a id="foundry-api-guide"></a>

## Machine Learning > NHN Cloud Foundry > API Guide { #foundry-api-guide }

This guide describes the APIs provided by NHN Cloud Foundry.

| API | Description |
| --- | --- |
| Ingest API | Collects data into an existing Data Source. Provides snapshot file upload, event collection, and metric collection. |
| Recommendation API | Requests recommendation results from a created Recommendation System App. |
| Recommendation Event API | Collects user reaction events on recommendation results. |

<a id="api-version"></a>

## API Version { #api-version }

The API provides two versions: v2.0 and v1.0. Both versions share the same request format (headers and request body) and the same `body` in success responses. They differ in HTTP status codes and result codes on failure.

| Version | Request Path | Status | Response Specification |
| --- | --- | --- | --- |
| [v2.0](#api-v2-guide) | `/api/v2.0/...` | Current version | HTTP status code `200` for both success and failure. Returns a 7-digit negative result code on failure. |
| [v1.0](#api-v1-guide) | `/api/v1.0/...` | Scheduled for discontinuation | HTTP status code varies depending on the API and error. Returns a positive result code on failure. |

It is recommended to use v2.0. The end-of-support schedule for v1.0 will be announced separately.

<a id="api-v2-guide"></a>

## API v2.0 Guide { #api-v2-guide }

v2.0 responds with HTTP status code `200` for both success and failure. The cause of failure is indicated by a 7-digit negative result code. The request format is the same as v1.0.

<a id="auth-common"></a>

### Authentication and Common Information { #auth-common }

<a id="auth-common-preparation"></a>
#### Prerequisites { #auth-common-preparation }

To use the API, you need an Appkey and an authentication token.

- You can find the Appkey in the **URL & Appkey** menu at the top of the **Machine Learning > NHN Cloud Foundry** page in the NHN Cloud Console.
- The API uses the **gateway-public** endpoint.
- For information on issuing an authentication token (Bearer token in the `X-NHN-Authorization` header), see the [User Access Key Token](/nhncloud/en/public-api/user-access-key-token/) guide.

<a id="auth-common-request"></a>
#### Common Request Information { #auth-common-request }

Required headers:

```plaintext
X-NC-APP-KEY: {appKey}
X-NHN-Authorization: Bearer {ACCESS_TOKEN}
Content-Type: application/json
```

Base URL:

```plaintext
https://{gateway-public-host}/api/v2.0
```

<a id="auth-common-response"></a>
#### Common Response Information { #auth-common-response }

All APIs respond with HTTP status code `200`, and the response body consists of `header` and `body`. Even when a request is rejected or processing fails, the HTTP status code is `200`. Therefore, success or failure must be determined by `header.isSuccessful`, not by the HTTP status code.

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
| header.isSuccessful | Boolean | Whether the request succeeded. Returns `false` on failure. |
| header.resultCode | Integer | Result code. `0` on success, a negative error code on failure. |
| header.resultMessage | String | Result message. `SUCCESS` on success, the error cause on failure. |
| body | Object/Array | Response data per API. Returns `null` on failure. |

Example of a failure response:

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

Error codes common to all APIs are listed in [Common Error Codes](#auth-common-error-codes) below. API-specific error codes are listed in the **Error Codes** section at the end of each API section.

- Even when the authentication token is missing or expired, the response is returned as a failure response with HTTP status code `200`.
- If a request is rejected before the API processes it, the HTTP status code may not be `200` and the response body may differ from the format above. This case is also treated as a failure.

!!! danger "Caution"
    If you determine success or failure based on the HTTP status code, failure responses may be treated as success. Clients must always use `header.isSuccessful` to determine success or failure, and use `header.resultCode` to identify the cause of failure.

<a id="auth-common-error-codes"></a>
#### Common Error Codes { #auth-common-error-codes }

These are error codes that can be returned by all APIs. For API-specific error codes, see the **Error Codes** section at the end of each API section.

The first three digits of an error code correspond to the HTTP status code. 400-series codes indicate a problem with the request — check the `resultMessage`, correct the request, and send it again. 500-series codes indicate a service-side issue — retry the same request after a moment, and if it continues to fail, contact the customer support center.

| Code | Message | Description |
| --- | --- | --- |
| -4010000 | Unauthorized | Authentication failed. The authentication token is missing, malformed, or expired, or the Appkey indicated by the token does not match the request target. Reissue the token and retry the request. |
| -4040000 | Not Found | The request path does not exist. Check the URI. |
| -4050000 | Method Not Allowed | The path is correct, but the HTTP method is incorrect. |
| -4150000 | Unsupported Media Type | The Content-Type of the request body is not supported. Send the request with `application/json`. |
| -5000000 | Internal Server Error | An internal server error occurred. Retry after a moment, and if the issue persists, contact the customer support center. |
| -5030101 | Authentication service is not ready. Please retry. | The authentication service is not ready. Retry after a moment. |

<a id="ingest-api"></a>

### Ingest API { #ingest-api }

The Ingest API loads data into a Data Source that has already been created in the Console. The following methods are available for each Data Source type:

| Method | Target Data Source | Description |
| --- | --- | --- |
| Snapshot Upload | File | Replaces all data with the uploaded file. |
| Event Collection | File | Adds change events one by one while retaining existing data. |
| Metric Collection | Prometheus API | Sends metric (time-series) data in real time. |

!!! danger "Caution"
    No API is provided for creating a new Data Source. To use the Ingest API, you must first create a Data Source in the Console.

<a id="ingest-snapshot"></a>

#### Snapshot Upload (File Upload) { #ingest-snapshot }

**Replaces all data** in the Data Source with the contents of the uploaded file. The upload process consists of three steps.

!!! danger "Caution"
    Snapshot upload replaces all data currently stored in the Data Source. Existing data cannot be recovered.

Upload limits:

- Maximum upload size: **10 GB**
- `100 MB` or less → **Single Upload (SINGLE)**
- Over `100 MB` → **Multipart Upload (MULTIPART)**
- Use the values included in the response for the `formPost` fields **exactly as-is** in the request.

<a id="ingest-snapshot-init"></a>
##### 1. Initialize Upload (init) { #ingest-snapshot-init }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/init |

Issues a signed temporary URL for uploading large files directly to storage. Returns either a single URL (SINGLE) or multipart URLs (MULTIPART) depending on the file size.

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/init" \
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
    The `formPost` field is also included in a MULTIPART response. However, for multipart uploads, parts are sent using the query parameters (`signature`/`expires`/`max_file_size`/`max_file_count`) from `parts[].uploadUrl`, so `formPost` is for reference only and is not used for the part upload itself.

| Field | Description |
| --- | --- |
| body.jobId | Job ID. Used in subsequent complete/status query requests |
| body.uploadType | Upload type. SINGLE (100 MB or less) or MULTIPART (over 100 MB) |
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
##### 2-A. Single File Upload (100 MB or Less) { #ingest-snapshot-upload-single }

Send a multipart/form-data POST request to the `uploadUrl` from the init response.
This request is sent directly to Object Storage, so no additional authentication is required (the `signature` serves as authentication).

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
    The `file` field must be added last in the form data. On success, an HTTP `201 Created` response is returned.

<a id="ingest-snapshot-upload-multipart"></a>
##### 2-B. Large File Upload (Over 100 MB, MULTIPART) { #ingest-snapshot-upload-multipart }

Use the `parts[]` array from the response to upload each part individually.
Each part is processed in the following order: (1) upload → (2) retrieve ETag via HEAD → (3) collect into `partETags[]` in ascending `partNumber` order.

1. Split the file into chunks of `partSize` (default: 100 MB).
2. For each part, parse the query parameters (`signature`, `expires`, `max_file_size`, `max_file_count`) from `parts[i].uploadUrl` and send the part as multipart/form-data (field name: `file`, fixed file name: `part`).
3. After a successful upload, send a `HEAD` request to `parts[i].headUrl` to collect the `ETag` value from the response header.
4. Once all parts are complete, construct the `partETags` array in ascending `partNumber` order and include it in the complete upload request.

Part upload curl example:

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
##### 3. Complete Upload (complete) { #ingest-snapshot-complete }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curl example (single upload):

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/complete" \
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
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/complete" \
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
| uploadId | String | X | Multipart upload ID (required only for multipart upload) |
| partETags | Array | X | List of ETags per part (required only for multipart upload, in partNumber order) |

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
| body.jobId | Job ID. Used for [View Job Status](#ingest-snapshot-job-status) |

<a id="ingest-snapshot-cancel"></a>
##### Cancel Upload { #ingest-snapshot-cancel }

| Method | URI |
| --- | --- |
| DELETE | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl example (single upload):

```bash
curl -X DELETE "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl example (multipart upload):

Pass `uploadId` as a query parameter.

```bash
curl -X DELETE "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest-snapshot-job-status"></a>
##### View Job Status { #ingest-snapshot-job-status }

| Method | URI |
| --- | --- |
| GET | /api/v2.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curl example:

```bash
curl "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
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
| body.dataSourceId | Target Data Source ID |
| body.jobType | Job type. SNAPSHOT (snapshot ingestion) or EVENT (change event) |
| body.status | Job status. See status values below. |
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
| UPLOADING | File upload in progress |
| QUEUED | Upload complete, pending ingestion |
| STAGED | Ready for processing |
| RUNNING | Data ingestion in progress |
| COMPLETED | Job completed successfully |
| FAILED | Job failed |

<a id="event-ingest-api"></a>

#### Event Ingestion { #event-ingest-api }

Sends change events while retaining existing data. This is used with file-type data sources, and the **Event API** must be enabled first. You can enable it from the Event Settings tab in the console or by using the Enable API below.

!!! danger "Caution"
    Enabling the Event API blocks snapshot uploads. Additionally, schema changes are restricted, so you must disable the Event API before modifying the schema. For instructions on enabling and disabling, refer to 'Event Settings' in the [Console User Guide](./console-user-guide/#datasource-detail-event).

<a id="event-ingest-api-enable"></a>
##### Enable/Disable Event API { #event-ingest-api-enable }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/enable |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/disable |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/events/enable" \
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
| body.enabled | Whether event ingestion is possible |
| body.status | Activation status. DISABLED, ENABLING, ENABLED, ENABLE_FAILED |

- Activation is processed asynchronously. Immediately after the request, the response will have `enabled` as `false` and `status` as `ENABLING`. Events are collected after the status becomes `ENABLED`.
- If you request activation on an already active data source, the current status is returned as-is. The same applies when requesting deactivation on an already inactive data source. Calling multiple times does not change the result.
- However, if you request activation while activation is in progress (`ENABLING`), the request will be rejected. You can check the progress on the Event Settings tab in the console.
- If you call with a non-existent data source, an error indicating that the data source cannot be found is returned.

<a id="event-ingest-api-send"></a>
##### Send Single Event { #event-ingest-api-send }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/events" \
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
| operation | String | O | Operation type. One of INSERT, UPDATE, DELETE |
| data | Object | O | Event data. Uses field names from the data source schema as keys |
| eventTimestamp | String | X | Time of the event occurrence. If omitted, the server receipt time is used |

- `operation` is case-insensitive. Sending an unsupported value will result in the request being rejected.
- If no primary key field is specified for the data source, only `INSERT` can be sent. `UPDATE` and `DELETE` will be rejected. Data sources with a primary key specified can use all three operations.

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
| body.success | Whether the request succeeded |
| body.errorMessage | Error message on failure |

<a id="event-ingest-api-batch"></a>
##### Send Multiple Events { #event-ingest-api-batch }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/batch |

Sends multiple change events in a single request. Up to 5,000 events can be sent per request.

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/events/batch" \
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
| events | Array | O | List of events. Up to 5,000 per request; each item's fields are the same as for a single event |

The `body` of the response is an array of processing results for each event.

<a id="metrics-ingest-api"></a>

#### Collect Metrics { #metrics-ingest-api }

Sends metric data to a data source of type Prometheus API. The transmitted metrics can be viewed in the Analysis menu, and can also be used as input for the Univariate Time-Series Anomaly Detection App.

| Method | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/metrics |

curl example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/metrics" \
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
| metrics | Array | O | Metric list. Cannot be empty; maximum 5,000 items per request. |
| metrics[].timestamp | Long | O | Metric timestamp. Millisecond epoch. |
| metrics[].value | Double | O | Measurement value. |
| metrics[].labels | Array | O | Label list. The combination of labels determines the time series, and Group Labels determine the group. |
| metrics[].labels[].name | String | O | Label name. Must start with a letter or underscore (_), and can only contain letters, numbers, and underscores. |
| metrics[].labels[].value | String | O | Label value. Commas and equal signs cannot be used. |
| metrics[].metadata | Object | X | Additional information. Stored and forwarded as-is without interpretation. The `identityKey` key is reserved for system use and cannot be used. |

If successful, `header.isSuccessful` returns `true` and there is no `body`. If the request is rejected, `header.isSuccessful` returns `false`, so use `header` to determine whether the request succeeded.

The collection rules are as follows:

- If required fields are missing, label name or value rules are violated, the 5,000-item limit per request is exceeded, or required headers are missing, the entire request is rejected and no items are stored.
- A single request can include metrics from multiple time series. Time series are distinguished by label combinations, so you do not need to split requests per time series.
- Send data for the same time series only once per minute. If you collect at shorter intervals, aggregate the values into a 1-minute average before sending. If multiple values arrive within the same minute, only the first value received is used for analysis; the rest are discarded.
- `timestamp` is an epoch in milliseconds. If sent in seconds, it will be stored as an incorrect time.
- If a Group Label is specified in the Data Source, always include that label when sending data. If the label is omitted, the data will not belong to the intended group.
- Items where `value` is NaN or Infinity are skipped and not stored. The remaining items in the same request are processed normally.
- A successful response means the data has been received. Storage takes effect shortly after, and resending the same request may result in duplicate data being stored.
- Data that is delayed in transmission is stored, but may be excluded from real-time inference.

!!! tip "Note"
    Ingestion is independent of the transmission interval. However, if this Data Source is connected to the Univariate Time-Series Anomaly Detection App, you must send data for the same time series continuously, one per minute. Since the App groups metrics in 1-minute units to make determinations, sending at longer intervals creates gaps that may prevent completion of preparation in accurate mode.
    Training also has requirements. If the Data Source contains only one time series, training will fail, so you must have at least two time series. Each time series must accumulate data continuously for approximately 4 hours or more to train correctly.

<a id="ingest-error-codes"></a>

#### Error Codes { #ingest-error-codes }

These are error codes that can be returned across all Ingest APIs in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description |
| --- | --- | --- |
| -4000001 | Invalid request. | Request format error. Missing required fields, range or format violations for values, request body JSON parsing failure, or missing `X-NC-APP-KEY` header. |
| -4041101 | DataSource not found. | The data source for the specified `dataSourceId` does not exist, or belongs to a different Appkey. |

<a id="ingest-error-codes-snapshot"></a>
##### Snapshot Upload { #ingest-error-codes-snapshot }

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4000202 | Invalid file name. | `fileName` is empty or contains characters other than letters, digits, `.`, `_`, and `-`. | init, complete |
| -4001107 | File size exceeds maximum limit. | `fileSize` exceeds 10 GB. | init |
| -4001101 | Invalid data source type. | The data source is not of file type. | init |
| -4001103 | DataSource is busy. | Cannot upload a snapshot because the data source is loading data or the Event API is enabled. | init, complete |
| -4001104 | Ingest job is already running. | There is already a snapshot upload job in progress for the same data source. Wait for it to complete or be canceled, then try again. | init, complete |
| -4000201 | DataSource is not ready for ingest. | The data source is not in a state where it can start loading. | complete |
| -4041102 | IngestJob not found. | The job for the specified `jobId` does not exist, or belongs to a different Appkey. | complete, Cancel Upload, View Job Status |
| -4001105 | IngestJob is in invalid status. | The job is not in an uploading state. It has already completed, been canceled, or failed due to the upload time limit expiring. Start again from init. | complete |
| -4000203 | File name does not match. | The `fileName` does not match the one sent during init. | complete |
| -4291101 | Too many requests. Please try again later. | Too many jobs are waiting to be processed and the request cannot be accepted. Try again later. | init, complete |

<a id="ingest-error-codes-event"></a>
##### Event Collection { #ingest-error-codes-event }

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4001101 | Invalid data source type. | The data source is not of file type. | Enable, Disable |
| -4000201 | DataSource is not ready for ingest. | The data source is not in a state where it can receive events. This may occur when a snapshot load is in progress. | Enable, Single Send, Batch Send |
| -4001104 | Ingest job is already running. | Cannot enable because a snapshot upload job is in progress. | Enable |
| -4091103 | Stream API activation is in progress. Please wait. | Activation is in progress. Wait for it to complete. | Enable |
| -4091601 | Operation is in progress. Please wait for the current operation to complete. | Another operation is in progress for the same data source. Try again after it completes. | Enable |
| -4000002 | Invalid strategy type. | The data source type does not support event collection. | Single Send, Batch Send |
| -4001109 | Stream API is not enabled. | The Event API is not enabled. Call the Enable API first. | Single Send, Batch Send |
| -4000204 | Invalid operation. | `operation` is not INSERT, UPDATE, or DELETE, or an operation other than INSERT was sent for a data source that has no primary key. | Single Send |
| -5004001 | Failed to serialize Kafka message. | Failed to convert the event to the storage format. | Single Send |
| -5004002 | Kafka send timeout. | Event storage did not complete within the timeout period. Try again later. | Single Send |
| -5004003 | Kafka send failed. | Failed to store the event. Try again later. | Single Send |
| -5004004 | Kafka send interrupted. | Event storage was interrupted. Try again later. | Single Send |

In Batch Send, per-item errors are returned in `body[].success` and `body[].errorMessage`, not in `header`. If data source validation fails, the entire request is rejected with one of the codes above.

<a id="ingest-error-codes-metrics"></a>
##### Metric Collection { #ingest-error-codes-metrics }

| Code | Message | Description |
| --- | --- | --- |
| -4000001 | Invalid request. | `metrics` is empty, exceeds 5,000 items, or is missing `timestamp`, `value`, or `labels`; a label name or value rule is violated; or the `X-NC-APP-KEY` header is missing. The entire request is rejected. |
| -4000002 | Invalid strategy type. | The data source type does not support metric collection. |
| -4000201 | DataSource is not ready for ingest. | The data source is not in a state where it can receive metrics. |
| -5004001 | Failed to serialize Kafka message. | Failed to convert the metric to the storage format. |
| -5004004 | Kafka send interrupted. | Metric storage was interrupted. Try again later. |
| -5004005 | Kafka batch send partially failed. | Some metrics failed to be stored or exceeded the timeout period. Resending the request may result in duplicate metrics that were already stored. |

<a id="univariate-api"></a>

### Univariate Time-Series Anomaly Detection API { #univariate-api }

<a id="univariate-group-api"></a>
#### Enable, Disable, and Delete Groups { #univariate-group-api }

Enables, disables, and deletes groups in the Univariate Time-Series Anomaly Detection app. All three APIs share the same request format, but differ only in the path.

| Method | URI |
| --- | --- |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId` is the app ID displayed in the Console app details.

curl Example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/serving-pipelines/{servingPipelineId}/groups/enable" \
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
| groupKey | Array | Conditional | List of labels that specify the target group. Used only when group labels are specified in the Data Source. |
| groupKey[].name | String | O | Group Label name. Must exactly match the group label name specified in the Data Source. |
| groupKey[].value | Array | O | List of values for that label. Each value corresponds to one group. |

On success, `header.isSuccessful` returns `true`, and there is no `body`.

The request rules are as follows:

- If no group labels are specified in the Data Source, do not send `groupKey`. The entire Data Source is treated as a single group, which becomes the target. Sending `groupKey` along with the request will result in the request being rejected.
- If group labels are specified in the Data Source, `groupKey` is required. The set of label names sent must exactly match the group labels in the Data Source. Sending the same label name twice will result in rejection.
- If there are multiple group labels, groups are formed by pairing values at the same index. For example, if you send `["a", "b"]` for `rule_id` and `["q", "w"]` for `instance_id`, the two target groups are `(a, q)` and `(b, w)`. The number of values for all labels must be equal; if they differ, the request is rejected.
- If a value list is empty, the request is rejected. If the same group is specified multiple times, it is processed only once.
- If you attempt to disable or delete a group that is not registered, an error is returned, and the other groups included in the request are also not processed.

!!! tip "Note"
    If no group labels are specified in the Data Source, the entire Data Source is registered as a single group when the app is created, so the app works without using this API. If group labels are specified, groups are not registered automatically, so you must register the target groups using the enable API to receive detection results. This operation is available through the API only. You can check registered groups and their statuses on the **Group List** tab in the Console app details.

!!! danger "Caution"
    Disabling a group does not stop the transmission of detection results. Only the status displayed in the group list changes to inactive.
    A deleted group disappears along with its status history and cannot be recovered.

<a id="univariate-error-codes"></a>
#### Error Codes { #univariate-error-codes }

The following error codes may be returned by the Enable, Disable, and Delete Groups API, in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description | Target API |
| --- | --- | --- | --- |
| -4000001 | Invalid request. | Violation of `groupKey` rules: sent `groupKey` to a Data Source with no group labels, required `groupKey` is missing, the set of label names does not match the group labels in the Data Source, duplicate label names, value list is empty, or value counts differ. Missing `X-NC-APP-KEY` header. | Enable, Disable, Delete |
| -4041301 | ServingPipeline not found. | No app exists for the specified `servingPipelineId`, or the app belongs to a different Appkey. | Enable, Disable, Delete |
| -4041101 | DataSource not found. | The Metric Data Source connected to the app cannot be found. | Enable, Disable, Delete |
| -4000201 | DataSource is not ready for ingest. | The Metric Data Source is not in a usable state. | Enable, Disable, Delete |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. Try again after the app status becomes active. | Enable |
| -4041306 | Group entry not found or has no trainingPipelineId. | One or more groups in the request are not registered. Other groups included in the request are also not processed. | Disable, Delete |

<a id="recommendation-api"></a>

### Recommendation API { #recommendation-api }

Requests recommendation results from the Recommendation System App. If a user has sufficient history, the server uses model-based (Sequential) inference; otherwise, it uses attribute-based (Cold Start) inference.

<a id="recommendation-api-recommend"></a>
#### Request Recommendations { #recommendation-api-recommend }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/recommendation-apps/{appId}/recommend |

curl Example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/recommendation-apps/{appId}/recommend" \
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
| userId | String | O | User ID to receive recommendations. To request recommendations for an anonymous user, specify an empty string (""). |
| context.currentItemKey | String | X | Key of the item currently being viewed |
| context.recentlyViewed | Array | X | List of recently viewed item keys |
| context.availableItems | Array | X | List of item keys available for recommendation. If specified, only items in this list are recommended. |
| context.pageType | String | X | Current page type (free-form; e.g., home, item_detail) |
| context.sessionId | String | X | Session ID |
| context.impressions | Array | X | List of items exposed to the user as recommendation results |
| context.interactions | Array | X | User's item interaction information |
| context.feedback | Array | X | Ratings that the user left on items |
| userAttributes | Object | X | User attribute information (used for Cold Start inference) |
| options.maxRecommendations | Integer | X | Maximum number of recommendations (from 1 to 100). Values exceeding 100 are adjusted to 100 without an error; if not specified, 100 is applied. If fewer items are available than this value, only the actual number of items is returned. |
| options.mode | String | X | Specifies the inference method. One of sequential (history-based), cold_start (attribute-based), or popular (popularity-based). Other values are rejected. If not specified, the server determines the method automatically. |
| options.longtail | Boolean | X | Improves recommendation diversity by including less popular items. Applies only when sequential is used. |
| options.excludeItemKeys | Array | X | List of item keys to exclude from recommendations. Excluded items are not counted in the maximum recommendation count. |

- If `options.mode` is not specified, the server determines the inference method. If a model for the determined method is not linked to the app, recommendations are made using another method linked to the app instead. The method actually used can be checked in `body.metadata.inferenceType` in the response.
- If an invalid value is sent for `options.mode`, the request is rejected with result code `-4004201`. Even if the value is valid, if the model for the requested method is not linked to the app and no alternative method is available, result code `-5034201` is returned. Check which models are available in the app and whether training has completed, then retry. Requests that specify a method via `options.mode` are not substituted, so you may receive this response.

<a id="recommendation-api-signal"></a>
##### Behavior Signals { #recommendation-api-signal }

`context.impressions` is used to re-rank recommendation results based on the recommendation information exposed to the user.
`context.interactions` and `context.feedback` are fields that convey the actions that the user took on recommendation results, and incorporate user behavior data into model inference.

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
| requestId | String | O | body.metadata.requestId of the recommendation response in which the action occurred |
| itemKeys | Array | O | List of exposed item keys. Enter in order of exposure; used in impressions. |
| itemKey | String | O | Target item key. Used in interactions and feedback. |
| type | String | O | For interactions: CLICK or CONVERSION. For feedback: POSITIVE or NEGATIVE. |
| occurredAt | String | O | Time when the action occurred |

- Send `occurredAt` in ISO 8601 format with a timezone offset. Requests without an offset are treated as errors.
- All three fields — `requestId`, `occurredAt`, and the item key — must be present for the signal to be used.
- Each field must be sent in chronological order from oldest to newest.
- `impressions` allows up to 10 entries, with up to 100 `itemKeys` per entry. `interactions` and `feedback` allow up to 10 entries per `type`. Requests that exceed these limits are rejected.
- Behavior signals are used only as inference input for the current recommendation request and are not stored. If `feedback` for the same item changes, only the most recent value is reflected, so resend it with each request to maintain its effect.
- To store reaction events for analysis, use the [Recommendation Event API](#recommendation-event-api) together.

!!! tip "Note"
    The `userAttributes` schema's collection method and field types may change in the future based on the implementation direction of Preference Elicitation.

Response Example:

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
| body.userId | User ID of the requester |
| body.recommendations[].itemKey | Recommended item key |
| body.recommendations[].score | Recommendation score (from 0.0 to 1.0) |
| body.recommendations[].position | Recommendation ranking |
| body.metadata.modelVersion | Version of the model used |
| body.metadata.requestId | Request tracking ID. Use this value when sending to the Recommendation Event API. |
| body.metadata.inferenceType | Inference type: sequential (history-based), cold_start (attribute-based), or popular (popularity-based) |
| body.metadata.abTestGroup | A/B test group (currently returns an empty value) |

<a id="recommendation-error-codes"></a>
#### Error Codes { #recommendation-error-codes }

The following error codes may be returned from recommendation requests in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description |
| --- | --- | --- |
| -4004201 | Invalid request. or rejection reason | Request format error. Causes include: missing `userId`, `maxRecommendations` less than 1, invalid value for `options.mode`, violation of the minimum recommendation count rule for a category, a category not found in the catalog, violation of behavior signal format or count in `context`, missing `X-NC-APP-KEY` header, etc. The `resultMessage` includes the rejection reason. |
| -4044201 | Recommendation app not found. | No recommendation app corresponding to `appId` exists, or the `X-NC-APP-KEY` does not match the app. |
| -4001301 | Invalid model type. | `appId` does not correspond to a Recommendation System App. |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. Retry after the app status becomes active. |
| -5034201 | Recommendation model is not ready. | The model for the requested recommendation mode is not yet linked to the app. Retry after the initial training and deployment are complete. |

<a id="recommendation-event-api"></a>

### Recommendation Event API { #recommendation-event-api }

Collects events representing user reactions (such as clicks) to recommendation results. You can analyze recommendation success rates using the collected event data.

<a id="recommendation-event-api-send"></a>
#### Send Recommendation Event { #recommendation-event-api-send }

| Method | URI |
| --- | --- |
| POST | /api/v2.0/recommendation-apps/{appId}/events |

curl Example:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/recommendation-apps/{appId}/events" \
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

Pass the values of `requestId`, `itemKey`, and `userId` exactly as received from the recommendation API response.

| Field | Required | Description |
| --- | --- | --- |
| eventType | O | Event type. Freely definable (e.g., CLICK, PURCHASE, IMPRESSION). Only letters, numbers, and underscores (_) are allowed (^[A-Za-z0-9_]+$), with a maximum of 64 characters. Stored in uppercase regardless of the case used. REQUEST and RESPONSE are reserved words and cannot be used. |
| requestId | O | The body.metadata.requestId value from the recommendation API response (opaque string, up to 128 characters) |
| itemKey | O | The itemKey of the recommended item that the user reacted to |
| userId | X | The body.userId value from the recommendation API response |
| context | X | Additional event information (free-form key-value pairs, e.g., exposure position, placement) |
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
    - A successful response (200) indicates that the collection pipeline has received the event; it does not guarantee that the data has been fully loaded into the analysis table.
    - After an event API request, it may take up to 10 minutes for the data to be loaded into the Dataset.
    - Retrying after a Timeout may result in the same event being loaded more than once. Consider deduplication when performing analysis.

<a id="recommendation-event-error-codes"></a>
#### Error Codes { #recommendation-event-error-codes }

These are error codes that can be returned when sending recommendation events, in addition to the [common error codes](#auth-common-error-codes).

| Code | Message | Description |
| --- | --- | --- |
| -4004202 | Invalid event request. | Invalid request format. Returned when `eventType` is empty, exceeds 64 characters, contains characters other than letters, numbers, or `_`, or is a reserved word (REQUEST, RESPONSE); when `requestId` is empty or exceeds 128 characters; when `itemKey` is missing; or when required headers are missing. |
| -4044201 | Recommendation app not found. | No recommendation app matching the `appId` was found, or the app does not match the `X-NC-APP-KEY`. |
| -4001301 | Invalid model type. | The `appId` does not correspond to a Recommendation System app. |
| -4001302 | Serving pipeline is not active. | The app is not in an active state. |
| -4094201 | Recommendation event dataset is not configured for this app. | No Data Source is configured to store recommendation events for the app. |
| -5034202 | Event publish failed. | Failed to save the event. Try again later. |
| -5044201 | Event publish timed out. | The event was not saved within the time limit. Try again later. |

<a id="api-v1-guide"></a>

## API v1.0 Guide { #api-v1-guide }

!!! danger "Caution"
    API v1.0 is scheduled for end of support. The end-of-support schedule will be announced separately.
    It is recommended to use v2.0. For more information, see [API v2.0 Guide](#api-v2-guide).

<a id="auth-common-v1"></a>

### Authentication and Common Information { #auth-common-v1 }

<a id="auth-common-preparation-v1"></a>
#### Prerequisites { #auth-common-preparation-v1 }

To use the API, you need an Appkey and an authentication token.

- The Appkey can be found in the **URL & Appkey** menu at the top of the **Machine Learning > NHN Cloud Foundry** page in the NHN Cloud Console.
- The API uses the **gateway-public** endpoint.
- For information on issuing an authentication token (Bearer token in the `X-NHN-Authorization` header), refer to the [User Access Key Token](/nhncloud/en/public-api/user-access-key-token/) guide.

<a id="auth-common-request-v1"></a>
#### Common Request Information { #auth-common-request-v1 }

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

<a id="auth-common-response-v1"></a>
#### Common Response Information { #auth-common-response-v1 }

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
| header.isSuccessful | Boolean | Whether the request was successful |
| header.resultCode | Integer | Result code. 0 for success, error code for failure |
| header.resultMessage | String | Result message. SUCCESS for success, detailed error message for failure |
| body | Object/Array | Response data per API |

Even if a request is rejected, the HTTP status code may be returned as `200`. Success is determined by `header.isSuccessful` and `header.resultCode`, not the HTTP status code. If the authentication token is missing or expired, HTTP `401` is returned.

<a id="ingest-api-v1"></a>

### Ingest API { #ingest-api-v1 }

The Ingest API loads data into Data Sources that have already been created in the Console. The following methods are provided based on the Data Source type:

| Method | Target Data Source | Description |
| --- | --- | --- |
| Snapshot Upload | File | Replaces all data with the uploaded file |
| Event Collection | File | Adds change events one by one while retaining existing data |
| Metric Collection | Prometheus API | Sends metric (time-series) data in real time |

!!! danger "Caution"
    No API is provided for creating new Data Sources. To use the Ingest API, you must first create a Data Source in the Console.

<a id="ingest-snapshot-v1"></a>

#### Snapshot Upload (File Upload) { #ingest-snapshot-v1 }

Replaces **all** data in the Data Source with the contents of the uploaded file. The upload process consists of three steps.

!!! danger "Caution"
    Snapshot upload replaces all data already loaded in the Data Source. Existing data cannot be recovered.

Upload limits:

- Maximum upload size: **10 GB**
- `100MB` or less → **Single Upload (SINGLE)**
- More than `100MB` → **Multipart Upload (MULTIPART)**
- Use the `formPost` field values **exactly as returned** in the response.

<a id="ingest-snapshot-init-v1"></a>
##### 1. Upload Initialization (init) { #ingest-snapshot-init-v1 }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init |

Issues a signed temporary URL for uploading large files directly to storage. Returns a single URL (SINGLE) or a multipart URL (MULTIPART) depending on the file size.

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
    The MULTIPART response also includes `formPost`. However, since multipart upload transmits each part using the query parameters (`signature`/`expires`/`max_file_size`/`max_file_count`) of `parts[].uploadUrl`, `formPost` is for reference only and is not used for the actual part upload.

| Field | Description |
| --- | --- |
| body.jobId | Job ID. Used in subsequent complete/status query requests |
| body.uploadType | Upload type. SINGLE (100 MB or less) or MULTIPART (more than 100 MB) |
| body.uploadUrl | Upload URL (for single upload) |
| body.uploadId | Multipart upload ID (for multipart upload) |
| body.partSize | Part size (bytes, for multipart upload) |
| body.parts[].partNumber | Part number (starting from 1) |
| body.parts[].uploadUrl | Part upload URL |
| body.parts[].headUrl | URL for ETag retrieval (HEAD request after upload completion) |
| body.expiresAt | URL expiration time |
| body.formPost.objectPrefix | Object prefix (path prepended to the file name) |
| body.formPost.signature | HMAC-SHA1 signature |
| body.formPost.expires | Expiration time (UNIX timestamp) |
| body.formPost.maxFileSize | Maximum file size (bytes) |
| body.formPost.maxFileCount | Maximum file count |

<a id="ingest-snapshot-upload-single-v1"></a>
##### 2-A. Single File Upload (100 MB or Less) { #ingest-snapshot-upload-single-v1 }

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
    The `file` field must be added last in the form data. On success, you will receive an HTTP `201 Created` response.

<a id="ingest-snapshot-upload-multipart-v1"></a>
##### 2-B. Large File Upload (More Than 100 MB, MULTIPART) { #ingest-snapshot-upload-multipart-v1 }

Receive the `parts[]` array from the response and upload each part individually.
Each part is processed in the following order: (1) upload → (2) retrieve the ETag via HEAD → (3) collect in `partETags[]` in ascending `partNumber` order.

1. Split the file into chunks of `partSize` (default 100 MB).
2. For each part, parse the query parameters (`signature`, `expires`, `max_file_size`, `max_file_count`) from `parts[i].uploadUrl` and send as multipart/form-data (field name `file`, fixed file name `part`).
3. After a successful upload, send a `HEAD` request to `parts[i].headUrl` to collect the `ETag` value from the response header.
4. Once all parts are complete, construct the `partETags` array in ascending `partNumber` order and include it in the upload complete request.

Part upload curl example:

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

<a id="ingest-snapshot-complete-v1"></a>
##### 3. Upload Complete (complete) { #ingest-snapshot-complete-v1 }

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
| uploadId | String | X | Multipart upload ID (required only for multipart upload) |
| partETags | Array | X | List of ETags per part (required only for multipart upload, in partNumber order) |

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
| body.jobId | Job ID. Used for [View Job Status](#ingest-snapshot-job-status-v1) |

<a id="ingest-snapshot-cancel-v1"></a>
##### Cancel Upload { #ingest-snapshot-cancel-v1 }

| Method | URI |
| --- | --- |
| DELETE | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl example (single upload):

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl example (multipart upload):

Pass `uploadId` as a query parameter.

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest-snapshot-job-status-v1"></a>
##### View Job Status { #ingest-snapshot-job-status-v1 }

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
| body.dataSourceId | Target Data Source ID |
| body.jobType | Job type. SNAPSHOT (snapshot loading) or EVENT (change event) |
| body.status | Job status. Refer to the status values below |
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

The job status (`status`) has the following values:

| Value | Description |
| --- | --- |
| UPLOADING | File is being uploaded |
| QUEUED | Upload complete, waiting to be loaded |
| STAGED | Ready for processing |
| RUNNING | Data is being loaded |
| COMPLETED | Job completed successfully |
| FAILED | Job failed |

<a id="event-ingest-api-v1"></a>

#### Collect Events { #event-ingest-api-v1 }

Sends change events while retaining existing data. This is used with a Data Source whose type is File, and you must enable the **Event API** first. You can enable it from the event settings tab in the Console or by using the Enable API below.

!!! danger "Caution"
    Enabling the Event API blocks snapshot uploads. Schema changes are also restricted; to change the Schema, you must disable the Event API first. For information on how to enable and disable it, refer to 'Event Settings' in the [Console User Guide](./console-user-guide/#datasource-detail-event).

<a id="event-ingest-api-enable-v1"></a>
##### Enable/Disable Event API { #event-ingest-api-enable-v1 }

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
| body.enabled | Whether events can be collected |
| body.status | Activation status. DISABLED, ENABLING, ENABLED, ENABLE_FAILED |

- Activation is processed asynchronously. Immediately after the request, the response has `enabled` set to `false` and `status` set to `ENABLING`. Events are collected after the status becomes `ENABLED`.
- Sending an enable request to an already active Data Source returns the current status unchanged. The same applies when sending a disable request to an already inactive Data Source. Calling the API multiple times does not change the result.
- However, if an enable request is sent while activation is in progress (`ENABLING`), the request is rejected. You can check the progress in the event settings tab in the Console.
- If the API is called with a non-existent Data Source, an error indicating that the Data Source could not be found is returned.

<a id="event-ingest-api-send-v1"></a>
##### Send Single Event { #event-ingest-api-send-v1 }

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
| operation | String | O | Operation type. One of INSERT, UPDATE, DELETE |
| data | Object | O | Event data. Uses the field names from the Data Source Schema as keys |
| eventTimestamp | String | X | Time when the event occurred. If omitted, the Server's received time is used |

- `operation` is case-insensitive. Sending an unsupported value results in the request being rejected.
- If the Data Source does not have a primary key field specified, only `INSERT` can be sent. `UPDATE` and `DELETE` are rejected. A Data Source with a primary key field specified can use all three operations.

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
| body.success | Whether the operation was successful |
| body.errorMessage | Error message on failure |

<a id="event-ingest-api-batch-v1"></a>
##### Send Events in Batch { #event-ingest-api-batch-v1 }

| Method | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/batch |

Sends multiple change events at once. Up to 5,000 events can be sent per request.

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
| events | Array | O | Event list. Up to 5,000 events per request; the fields for each item are the same as for single event sending |

The `body` of the response is an array of processing results for each event.

<a id="metrics-ingest-api-v1"></a>

#### Collect Metrics { #metrics-ingest-api-v1 }

Sends metric data to a Data Source of the Prometheus API type. The sent metrics can be viewed in the Analysis menu and can also be used as input for the Univariate Time-Series Anomaly Detection App.

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
| metrics | Array | O | List of metrics. Cannot be empty; maximum of 5,000 items per request. |
| metrics[].timestamp | Long | O | Metric timestamp. Millisecond epoch. |
| metrics[].value | Double | O | Measured value. |
| metrics[].labels | Array | O | List of labels. The label combination determines the time series, and the Group Label determines the Group. |
| metrics[].labels[].name | String | O | Label name. Must start with a letter or underscore (_) and can only contain letters, numbers, and underscores. |
| metrics[].labels[].value | String | O | Label value. Commas and equals signs cannot be used. |
| metrics[].metadata | Object | X | Additional information. Stored and delivered as-is without interpretation. The `identityKey` key is reserved for system use and cannot be used. |

On success, HTTP `202 Accepted` is returned. If the request is rejected, HTTP `200` is returned with `header.isSuccessful` set to `false`. Check the `header` to determine whether the request succeeded.

The collection rules are as follows:

- If a required field is missing, label name or value rules are violated, the number of items per request exceeds 5,000, or a required header is missing, the entire request is rejected and no items are stored.
- A single request can contain metrics for multiple time series. Because time series are distinguished by their label combinations, there is no need to split requests by time series.
- Each time series should be sent only once per minute. If data is collected at a shorter interval, average the values into 1-minute intervals before sending. If multiple values arrive for the same minute, only the first value received is used for analysis, and the rest are discarded.
- `timestamp` is a millisecond-unit epoch. If sent in seconds, the data is stored with an incorrect timestamp.
- If a Group Label is configured for the Data Source, always include that label when sending data. If the label is omitted, the metric will not belong to the intended Group.
- Items where `value` is NaN or Infinity are skipped and not stored. The remaining items in the same request are processed normally.
- A `202` response indicates that the data has been received. Storage is reflected shortly after, and sending the same request again may result in duplicate data being stored.
- Data sent with a delay is stored but may be excluded from real-time inference.

!!! tip "Note"
    Ingestion is independent of the transmission interval. However, if this Data Source is connected to the Univariate Time-Series Anomaly Detection App, you must send data for the same time series without interruption — one record per minute. Because the App evaluates metrics in 1-minute intervals, sending data at longer intervals creates gaps that may prevent preparation from completing in precise mode.
    Training also has requirements. If the Data Source has only one time series, training fails, so there must be at least two time series. Each time series must also accumulate at least approximately 4 hours of uninterrupted data for training to complete successfully.

<a id="univariate-api-v1"></a>

### Univariate Time-Series Anomaly Detection API { #univariate-api-v1 }

<a id="univariate-group-api-v1"></a>
#### Enable, Stop, and Delete Groups { #univariate-group-api-v1 }

Enables, stops, and deletes groups in the Univariate Time-Series Anomaly Detection app. The three APIs share the same request format, differing only in the path.

| Method | URI |
| --- | --- |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId` is the app ID displayed in the app Details on the Console.

curl example:

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
| groupKey | Array | Conditional | List of labels specifying the target groups. Used only when group labels are specified in the Data Source. |
| groupKey[].name | String | O | Group Label name. Must exactly match the name of the group label specified in the Data Source. |
| groupKey[].value | Array | O | List of values for the label. Each value corresponds to one group. |

On success, `header.isSuccessful` is returned as `true` and `body` is empty.

Request rules are as follows:

- If no group labels are specified in the Data Source, do not send `groupKey`. The entire Data Source constitutes a single group, and that group becomes the target. If `groupKey` is included in the request, the request will be rejected.
- If group labels are specified in the Data Source, `groupKey` is required. The set of label names sent must exactly match the group labels in the Data Source. Sending the same label name twice will result in rejection.
- If there are multiple group labels, values are paired in order to form groups. For example, if `["a", "b"]` is sent for `rule_id` and `["q", "w"]` for `instance_id`, the two target groups are `(a, q)` and `(b, w)`. All labels must have the same number of values; if they differ, the request is rejected.
- An empty value list will result in rejection. If the same group is specified multiple times, it is processed only once.
- Stopping or deleting an unregistered group returns an error.

!!! tip "Note"
    If no group labels are specified in the Data Source, the entire Data Source is registered as a single group when app creation is complete, so this API is not required for operation. If group labels are specified, groups are not registered automatically. You must register the target groups using the Enable API to receive detection results. This operation is available through the API only. Registered groups and their statuses can be checked on the **Group List** tab in the app Details on the Console.

!!! danger "Caution"
    Stopping a group does not stop the transmission of detection results. Only the status displayed in the group list changes to Inactive.
    A deleted group disappears along with its status history and cannot be recovered.

<a id="recommendation-api-v1"></a>

### Recommendation API { #recommendation-api-v1 }

Requests recommendation results from the Recommendation System App that you created. If user history is sufficient, inference uses a model-based approach (Sequential); if insufficient, it uses an attribute-based approach (Cold Start).

<a id="recommendation-api-recommend-v1"></a>
#### Recommendation Request { #recommendation-api-recommend-v1 }

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
| userId | String | O | User ID of the user to receive recommendations. To request recommendations for an anonymous user, specify an empty string ("") |
| context.currentItemKey | String | X | Key of the item currently being viewed |
| context.recentlyViewed | Array | X | List of recently viewed item keys |
| context.availableItems | Array | X | List of item keys to recommend from. If specified, only items in this list are recommended |
| context.pageType | String | X | Current page type (free format, e.g., home, item_detail) |
| context.sessionId | String | X | Session ID |
| context.impressions | Array | X | List of items exposed to the user as recommendation results |
| context.interactions | Array | X | User's item interaction information |
| context.feedback | Array | X | User's feedback on items |
| userAttributes | Object | X | User attribute information (used for Cold Start inference) |
| options.maxRecommendations | Integer | X | Maximum number of recommendations (from 1 to 100). Values exceeding 100 are adjusted to 100 without an error; defaults to 100 if not specified. If fewer items are available than this value, only the actual number of items is returned |
| options.mode | String | X | Specifies the inference method. One of sequential (history-based), cold_start (attribute-based), or popular (popularity-based). If not specified, the server determines automatically |
| options.longtail | Boolean | X | Improves recommendation diversity by including less popular items. Applies only when sequential is used |
| options.excludeItemKeys | Array | X | List of item keys to exclude from recommendations. Excluded items are not counted toward the maximum number of recommendations |

- If `options.mode` is not specified, the server determines the inference method. If the app does not have a model for the determined method, it falls back to another method linked to the app. The method actually used can be checked in `body.metadata.inferenceType` in the response.
- If the app does not have a model for the requested method and no fallback method is available, HTTP `503` and result code `5030001` are returned. Check which models are available in the app and whether training is complete, then retry. Requests that specify a method with `options.mode` do not fall back, so this response may be returned.

<a id="recommendation-api-signal-v1"></a>
##### Behavior Signals { #recommendation-api-signal-v1 }

`context.impressions` is used to re-rank recommendation results based on the recommendation information exposed to the user.
`context.interactions` and `context.feedback` are fields that convey user actions on recommendation results, and reflect user behavior data in model inference.

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
| requestId | String | O | body.metadata.requestId of the recommendation response in which the action occurred |
| itemKeys | Array | O | List of exposed item keys. Enter in order of exposure; used in impressions |
| itemKey | String | O | Target item key. Used in interactions and feedback |
| type | String | O | For interactions: CLICK, CONVERSION. For feedback: POSITIVE, NEGATIVE |
| occurredAt | String | O | Time when the action occurred |

- `occurredAt` must be sent in ISO 8601 format with a timezone offset. Requests without an offset are treated as an error.
- All three fields — `requestId`, `occurredAt`, and the item key — must be present for the data to be used as a signal.
- Each field should be provided in chronological order, from oldest to most recent.
- `impressions` supports up to 10 entries, with up to 100 `itemKeys` per entry. `interactions` and `feedback` support up to 10 entries per `type`. Requests that exceed these limits are rejected.
- Behavior signals are used only as inference input for the current recommendation request and are not stored. If `feedback` for the same item changes, only the most recent value is reflected, so you must resend on every request to maintain the effect.
- To store reaction events for analysis, also use the [Recommendation Event API](#recommendation-event-api-v1).

!!! tip "Note"
    The `userAttributes` schema may change in its collection method or field types depending on the future direction of Preference Elicitation implementation.

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
| body.userId | User ID from the request |
| body.recommendations[].itemKey | Recommended item key |
| body.recommendations[].score | Recommendation score (from 0.0 to 1.0) |
| body.recommendations[].position | Recommendation rank |
| body.metadata.modelVersion | Model version used |
| body.metadata.requestId | Request tracking ID. Use this value when sending to the Recommendation Event API |
| body.metadata.inferenceType | Inference type. sequential (history-based), cold_start (attribute-based), popular (popularity-based) |
| body.metadata.abTestGroup | A/B test group (currently returns an empty value) |

<a id="recommendation-event-api-v1"></a>

### Recommendation Event API { #recommendation-event-api-v1 }

Collects user reaction events (such as clicks) on recommendation results. You can analyze the recommendation success rate using the collected event data.

<a id="recommendation-event-api-send-v1"></a>
#### Send Recommendation Event { #recommendation-event-api-send-v1 }

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

Pass the values of `requestId`, `itemKey`, and `userId` exactly as received from the Recommendation API response.

| Field | Required | Description |
| --- | --- | --- |
| eventType | O | Event type. Freely definable (e.g., CLICK, PURCHASE, IMPRESSION). Only letters, numbers, and underscores (_) are allowed (^[A-Za-z0-9_]+$), with a maximum of 64 characters. Stored normalized to uppercase regardless of case. REQUEST and RESPONSE are reserved words and cannot be used |
| requestId | O | The body.metadata.requestId value from the Recommendation API response (opaque string, up to 128 characters) |
| itemKey | O | The itemKey of the recommended item that the user interacted with |
| userId | X | The body.userId value from the Recommendation API response |
| context | X | Additional event information (free-format key-value pairs, e.g., exposure position, placement) |
| userAttributes | X | User attribute information (free-format key-value pairs) |
| options | X | Additional options (free-format key-value pairs) |

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
    - A success response (200) means the collection pipeline has received the event; it does not guarantee that the data has been fully loaded into the analysis table.
    - It may take up to 10 minutes for data to be loaded into the dataset after an event API request.
    - Retrying after a timeout may result in duplicate event entries. Consider deduplication when performing analysis.