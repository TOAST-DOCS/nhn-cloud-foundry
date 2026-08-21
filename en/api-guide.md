<!-- machine_translated: true -->

<!-- pre-align:aligned sig=876ecae2be90 -->

<a id="foundry.api.guide"></a>

## Machine Learning > NHN Cloud Foundry > API Guide { #foundry.api.guide }

Describes the APIs provided by NHN Cloud Foundry.

| API | Description |
| --- | --- |
| Ingest API | Ingests data into an existing data source. Provides snapshot file upload. |
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

The Ingest API loads data into a data source that you have already created in the console. It provides a snapshot upload method that replaces all data in the data source with the uploaded file.

!!! danger "Caution"
    An API for creating new data sources is not provided. To use the Ingest API, you must first create a data source in the console, and only FILE type data sources can be used.

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

<a id="recommendation.api"></a>

## Recommendation API { #recommendation.api }

Requests recommendation results from the recommendation system app that you created. If the user's interaction history is sufficient, the server performs model-based inference (Normal Flow); if the history is insufficient, it performs attribute-based inference (Cold Start).

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
| userAttributes | Object | X | User attribute information (used for Cold Start inference) |
| options.maxRecommendations | Integer | X | Maximum number of recommendations (from 1 to 100). Values exceeding 100 are adjusted to 100 without an error; if not specified, 100 is applied. If fewer items are available than this value, only the actual number of available items is returned. |
| options.mode | String | X | Specifies the inference mode. One of: sequential (history-based), cold_start (attribute-based), popular (popularity-based). If not specified, the server determines this automatically. |
| options.longtail | Boolean | X | Improves recommendation diversity by including less popular items. Applies only when sequential is used. |
| options.excludeItemKeys | Array | X | List of item keys to exclude from recommendations. Excluded items are not counted toward the maximum number of recommendations. |

!!! tip "Note"
    The `userAttributes` schema may change in terms of collection method or field types depending on the future implementation direction of Preference Elicitation.

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
| body.userId | ID of the requesting user |
| body.recommendations[].itemKey | Recommended item key |
| body.recommendations[].score | Recommendation score (from 0.0 to 1.0) |
| body.recommendations[].position | Recommendation rank |
| body.metadata.modelVersion | Model version used |
| body.metadata.requestId | Request tracking ID. Use this value when sending the recommendation event API. |
| body.metadata.inferenceType | Inference type. One of: sequential (history-based), cold_start (attribute-based), popular (popularity-based) |
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