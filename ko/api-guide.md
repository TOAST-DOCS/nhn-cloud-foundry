<!-- pre-align:aligned sig=d3aa4d31c69a -->

<a id="foundry-api-guide"></a>
## Machine Learning > NHN Cloud Foundry > API 가이드 { #foundry-api-guide }

NHN Cloud Foundry가 제공하는 API를 설명합니다.

| API | 설명 |
| --- | --- |
| Ingest API | 이미 만든 데이터 소스에 데이터 수집. 스냅숏 파일 업로드, 이벤트 수집, 지표 수집 제공 |
| 추천 조회 API | 생성한 추천 시스템 앱에 추천 결과 요청 |
| 추천 이벤트 API | 추천 결과에 사용자가 보인 반응 이벤트 수집 |

<a id="auth-common"></a>
## 인증 및 공통 사항 { #auth-common }

<a id="auth-common-preparation"></a>
### 사전 준비 { #auth-common-preparation }

API를 사용하려면 **Appkey**와 **인증 토큰**이 필요합니다.

- Appkey는 NHN Cloud 콘솔의 **Machine Learning > NHN Cloud Foundry** 페이지 상단 **URL & Appkey** 메뉴에서 확인할 수 있습니다.
- API는 **gateway-public** 엔드포인트를 사용합니다.
- 인증 토큰(`X-NHN-Authorization` 헤더의 Bearer 토큰) 발급 방법은 [User Access Key 토큰](/nhncloud/ko/public-api/user-access-key-token/) 가이드를 참고합니다.

<a id="auth-common-request"></a>
### 요청 공통 사항 { #auth-common-request }

필수 헤더:

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
### 응답 공통 사항 { #auth-common-response }

모든 API는 HTTP 상태 코드 `200`으로 응답하며, 응답 본문은 `header`와 `body`로 구성됩니다. 요청이 거절되거나 처리에 실패한 경우에도 HTTP 상태 코드는 `200`이므로, 성공 여부는 HTTP 상태 코드가 아니라 `header.isSuccessful`로 판정합니다.

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

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| header.isSuccessful | Boolean | 요청 성공 여부. 실패하면 `false` |
| header.resultCode | Integer | 결과 코드. 성공 시 `0`, 실패 시 음수 오류 코드 |
| header.resultMessage | String | 결과 메시지. 성공 시 `SUCCESS`, 실패 시 오류 원인 |
| body | Object/Array | API별 응답 데이터. 실패 시 `null` |

실패 응답 예시:

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

모든 API에 공통인 오류 코드는 아래 [공통 오류 코드](#auth-common-error-codes)에, API별 오류 코드는 각 API 절 끝의 **오류 코드**에 있습니다.

- 인증 토큰이 없거나 만료된 경우에도 HTTP `200`에 실패 응답으로 반환됩니다.
- 네트워크 장애 등으로 요청이 서비스에 도달하지 못한 경우에는 `header` 없이 다른 HTTP 상태 코드가 반환될 수 있습니다. 이 경우도 실패로 처리합니다.

!!! danger "주의"
    HTTP 상태 코드로 성공 여부를 판정하면 실패 응답까지 성공으로 처리됩니다. 클라이언트는 반드시 `header.isSuccessful`로 성공 여부를 판정하고, 실패 원인은 `header.resultCode`로 구분합니다.

<a id="auth-common-error-codes"></a>
### 공통 오류 코드 { #auth-common-error-codes }

모든 API에서 반환될 수 있는 오류 코드입니다. API별 오류 코드는 각 API 절 끝의 **오류 코드**를 참고합니다.

오류 코드의 앞 세 자리는 HTTP 상태 코드와 같은 의미입니다. 400대 코드는 요청에 문제가 있는 경우이므로 `resultMessage`를 확인해 요청을 수정한 뒤 다시 보냅니다. 500대 코드는 서비스 쪽 문제이므로 잠시 후 같은 요청을 다시 시도하고, 계속 실패하면 고객센터에 문의합니다.

| 코드 | 메시지 | 설명 |
| --- | --- | --- |
| -4010000 | Unauthorized | 인증 실패. 인증 토큰이 없거나 형식이 잘못되었거나 만료되었거나, 토큰이 가리키는 Appkey가 요청 대상과 다릅니다. 토큰을 다시 발급받아 요청합니다. |
| -4040000 | Not Found | 요청 경로가 없습니다. URI를 확인합니다. |
| -4050000 | Method Not Allowed | 경로는 맞지만 HTTP 메서드가 다릅니다. |
| -4060000 | Not Acceptable | Accept 헤더로 JSON 응답을 받을 수 없습니다. |
| -4150000 | Unsupported Media Type | 요청 본문의 Content-Type을 지원하지 않습니다. `application/json`으로 보냅니다. |
| -5000000 | Internal Server Error | 서버 내부 오류입니다. 잠시 후 다시 시도하고, 계속되면 고객센터로 문의합니다. |
| -5030101 | Authentication service is not ready. Please retry. | 인증 서비스가 준비되지 않았습니다. 잠시 후 다시 시도합니다. |

<a id="ingest-api"></a>
## Ingest API { #ingest-api }

Ingest API는 콘솔에서 이미 만든 데이터 소스에 데이터를 적재하는 API입니다. 데이터 소스 타입에 따라 다음 방식을 제공합니다.

| 방식 | 대상 데이터 소스 | 설명 |
| --- | --- | --- |
| 스냅숏 업로드 | 파일 | 업로드한 파일로 데이터를 전부 교체 |
| 이벤트 수집 | 파일 | 기존 데이터를 유지한 채 변경 이벤트를 건별로 추가 |
| 지표 수집 | Prometheus API | 지표(시계열) 데이터를 실시간으로 전송 |

!!! danger "주의"
    데이터 소스를 새로 만드는 API는 제공하지 않습니다. Ingest API를 사용하려면 콘솔에서 데이터 소스를 먼저 생성해야 합니다.

<a id="ingest-snapshot"></a>
### 스냅숏 업로드(파일 업로드) { #ingest-snapshot }

업로드한 파일의 내용으로 데이터 소스의 데이터를 **전부 교체**합니다. 업로드는 3단계로 진행됩니다.

!!! danger "주의"
    스냅숏 업로드는 데이터 소스에 이미 적재된 데이터를 모두 교체합니다. 기존 데이터는 복구할 수 없습니다.

업로드 제한:

- 최대 업로드 크기: **10GB**
- `100MB` 이하 → **단일 업로드(SINGLE)**
- `100MB` 초과 → **멀티파트 업로드(MULTIPART)**
- `formPost` 필드 값들은 응답에 포함된 값을 **그대로** 요청에 넣어 사용합니다.

<a id="ingest-snapshot-init"></a>
#### 1. 업로드 초기화(init) { #ingest-snapshot-init }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init |

대용량 파일을 스토리지에 직접 업로드하기 위한 서명된 임시 URL을 발급합니다. 파일 크기에 따라 단일 URL(SINGLE) 또는 멀티파트 URL(MULTIPART)을 반환합니다.

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| fileName | String | O | 파일 이름. 허용 문자: 영문, 숫자, 점(.), 밑줄(_), 하이픈(-) |
| fileSize | Long | O | 파일 크기(bytes). 최소 1, 최대 10GB |
| contentType | String | X | Content-Type(기본값: application/octet-stream) |

응답 예시(SINGLE):

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

응답 예시(MULTIPART):

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

!!! tip "알아두기"
    MULTIPART 응답에도 `formPost`가 포함됩니다. 단, 멀티파트 업로드는 `parts[].uploadUrl`의 쿼리 파라미터(`signature`/`expires`/`max_file_size`/`max_file_count`)로 파트를 전송하므로, `formPost`는 참고용이며 파트 업로드 자체에는 사용하지 않습니다.

| 필드 | 설명 |
| --- | --- |
| body.jobId | 작업 ID. 이후 complete/상태 조회 요청에 사용 |
| body.uploadType | 업로드 타입. SINGLE(100MB 이하) 또는 MULTIPART(100MB 초과) |
| body.uploadUrl | 업로드 URL(단일 업로드 시) |
| body.uploadId | 멀티파트 업로드 ID(멀티파트 업로드 시) |
| body.partSize | 파트 크기(bytes, 멀티파트 업로드 시) |
| body.parts[].partNumber | 파트 번호(1부터 시작) |
| body.parts[].uploadUrl | 파트 업로드 URL |
| body.parts[].headUrl | ETag 조회용 URL(업로드 완료 후 HEAD 요청) |
| body.expiresAt | URL 만료 시간 |
| body.formPost.objectPrefix | 오브젝트 prefix(파일 이름 앞에 붙는 경로) |
| body.formPost.signature | HMAC-SHA1 서명 |
| body.formPost.expires | 만료 시간(UNIX timestamp) |
| body.formPost.maxFileSize | 최대 파일 크기(bytes) |
| body.formPost.maxFileCount | 최대 파일 개수 |

<a id="ingest-snapshot-upload-single"></a>
#### 2-A. 단일 파일 업로드(100MB 이하) { #ingest-snapshot-upload-single }

init 응답의 `uploadUrl`로 multipart/form-data POST를 보냅니다.
이 요청은 Object Storage에 직접 보내므로 별도 인증이 필요 없습니다(`signature`가 인증 역할).

curl 예시:

```bash
curl -X POST "{uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={formPost.maxFileSize}" \
  -F "max_file_count={formPost.maxFileCount}" \
  -F "expires={formPost.expires}" \
  -F "signature={formPost.signature}" \
  -F "file=@./data.csv;filename=data.csv"
```

!!! danger "주의"
    `file` 필드는 반드시 폼 데이터의 **마지막**에 추가해야 합니다. 성공 시 HTTP `201 Created` 응답을 받습니다.

<a id="ingest-snapshot-upload-multipart"></a>
#### 2-B. 대용량 파일 업로드(100MB 초과, MULTIPART) { #ingest-snapshot-upload-multipart }

응답의 `parts[]` 배열을 받아서 파트별로 업로드합니다.
각 파트는 **(1) 업로드 → (2) HEAD로 ETag 조회 → (3) `partETags[]`에 `partNumber` 오름차순으로 수집** 순서로 처리합니다.

1. 파일을 `partSize`(기본 100MB) 단위로 분할합니다.
2. 파트마다 `parts[i].uploadUrl`의 쿼리 파라미터(`signature`, `expires`, `max_file_size`, `max_file_count`)를 파싱하여 multipart/form-data로 전송합니다(필드 이름 `file`, 파일 이름 고정 `part`).
3. 업로드 성공 후 `parts[i].headUrl`로 `HEAD` 요청을 보내 응답 헤더의 `ETag` 값을 수집합니다.
4. 모든 파트가 완료되면 `partETags` 배열을 `partNumber` 오름차순으로 구성하여 업로드 완료(complete) 요청에 담아 보냅니다.

파트 업로드 curl 예시:

```bash
# 1) 업로드
curl -X POST "{parts[i].uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={max_file_size-from-query}" \
  -F "max_file_count={max_file_count-from-query}" \
  -F "expires={expires-from-query}" \
  -F "signature={signature-from-query}" \
  -F "file=@./part_i.bin;filename=part"

# 2) ETag 조회
curl -I "{parts[i].headUrl}" | grep -i '^etag:'
```

<a id="ingest-snapshot-complete"></a>
#### 3. 업로드 완료(complete) { #ingest-snapshot-complete }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curl 예시(단일 업로드):

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

curl 예시(멀티파트 업로드):

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| jobId | String | O | 작업 ID(init 응답의 jobId) |
| fileName | String | O | 파일 이름 |
| uploadId | String | X | 멀티파트 업로드 ID(멀티파트 업로드 시에만 필요) |
| partETags | Array | X | 파트별 ETag 목록(멀티파트 업로드 시에만 필요, partNumber순) |

응답 예시:

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

| 필드 | 설명 |
| --- | --- |
| body.jobId | 작업 ID. [작업 상태 조회](#ingest-snapshot-job-status)에 사용 |

<a id="ingest-snapshot-cancel"></a>
#### 업로드 취소 { #ingest-snapshot-cancel }

| 메서드 | URI |
| --- | --- |
| DELETE | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl 예시(단일 업로드):

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl 예시(멀티파트 업로드) - 쿼리 파라미터로 `uploadId`를 함께 전달합니다:

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest-snapshot-job-status"></a>
#### 작업 상태 조회 { #ingest-snapshot-job-status }

| 메서드 | URI |
| --- | --- |
| GET | /api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curl 예시:

```bash
curl "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

응답 예시:

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

| 필드 | 설명 |
| --- | --- |
| body.jobId | 작업 ID |
| body.dataSourceId | 대상 데이터 소스 ID |
| body.jobType | 작업 타입. SNAPSHOT(스냅숏 적재) 또는 EVENT(변경 이벤트) |
| body.status | 작업 상태. 아래 상태 값 참고 |
| body.obsFilePath | OBS 파일 경로 |
| body.statistics.totalRecords | 총 레코드 수 |
| body.statistics.failedRecords | 실패 레코드 수 |
| body.statistics.successfulRecords | 성공 레코드 수 |
| body.statistics.successRate | 성공률(0.0~1.0) |
| body.errorMessage | 오류 메시지(실패 시) |
| body.createdDatetime | 작업 생성 시각 |
| body.startedDatetime | 작업 시작 시각 |
| body.completedDatetime | 작업 완료 시각 |
| body.modifiedDatetime | 최종 수정 시각 |

작업 상태(`status`)는 다음 값을 가집니다.

| 값 | 설명 |
| --- | --- |
| UPLOADING | 파일 업로드 중 |
| QUEUED | 업로드 완료, 적재 대기 중 |
| STAGED | 처리 준비 완료 |
| RUNNING | 데이터 적재 중 |
| COMPLETED | 작업 정상 완료 |
| FAILED | 작업 실패 |

<a id="event-ingest-api"></a>
### 이벤트 수집 { #event-ingest-api }

기존 데이터를 유지한 채 변경 이벤트를 전송합니다. 타입이 파일인 데이터 소스에서 사용하며, **Event API**를 먼저 활성화해야 합니다. 활성화는 콘솔의 이벤트 설정 탭 또는 아래 활성화 API로 합니다.

!!! danger "주의"
    Event API를 활성화하면 스냅숏 업로드가 차단됩니다. 또한 스키마 변경이 제한되므로, 스키마를 변경하려면 Event API를 먼저 비활성화해야 합니다. 활성화·비활성화 방법은 [콘솔 유저 가이드](./console-user-guide/#datasource-detail-event)의 '이벤트 설정'을 참고합니다.

<a id="event-ingest-api-enable"></a>
#### Event API 활성화·비활성화 { #event-ingest-api-enable }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/enable |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/disable |

curl 예시:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/events/enable" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

응답 예시:

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

| 필드 | 설명 |
| --- | --- |
| body.enabled | 이벤트 수집 가능 여부 |
| body.status | 활성화 상태. DISABLED, ENABLING, ENABLED, ENABLE_FAILED |

- 활성화는 비동기로 진행됩니다. 요청 직후 응답은 `enabled`가 false, `status`가 ENABLING이며, ENABLED가 된 뒤부터 이벤트를 수집합니다.
- 이미 활성인 데이터 소스에 다시 활성화를 요청하면 상태를 그대로 반환합니다. 비활성 상태에 비활성화를 요청할 때도 같습니다. 여러 번 호출해도 결과가 달라지지 않습니다.
- 다만 활성화가 진행 중(`ENABLING`)일 때 다시 활성화를 요청하면 거절됩니다. 진행 상황은 콘솔의 이벤트 설정 탭에서 확인할 수 있습니다.
- 없는 데이터 소스로 호출하면 데이터 소스를 찾을 수 없다는 오류가 반환됩니다.

<a id="event-ingest-api-send"></a>
#### 이벤트 단건 전송 { #event-ingest-api-send }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events |

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| operation | String | O | 작업 타입. INSERT, UPDATE, DELETE 중 하나 |
| data | Object | O | 이벤트 데이터. 데이터 소스 스키마의 필드 이름을 키로 사용 |
| eventTimestamp | String | X | 이벤트 발생 시각. 생략 시 서버 수신 시각 사용 |

- `operation`은 대소문자를 구분하지 않습니다. 허용하지 않는 값을 보내면 요청이 거절됩니다.
- 데이터 소스에 기본 키 필드를 지정하지 않았으면 `INSERT`만 보낼 수 있습니다. `UPDATE`와 `DELETE`는 거절됩니다. 기본 키를 지정한 데이터 소스는 세 작업을 모두 사용할 수 있습니다.

응답 예시:

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

| 필드 | 설명 |
| --- | --- |
| body.eventId | 이벤트 ID |
| body.success | 처리 성공 여부 |
| body.errorMessage | 실패 시 오류 메시지 |

<a id="event-ingest-api-batch"></a>
#### 이벤트 다건 전송 { #event-ingest-api-batch }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/batch |

여러 건의 변경 이벤트를 한 번에 전송합니다. 1회 요청당 최대 5,000건까지 전송할 수 있습니다.

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| events | Array | O | 이벤트 목록. 1회 요청당 최대 5,000건이며 각 항목의 필드는 단건 전송과 동일 |

응답의 `body`는 이벤트별 처리 결과 배열입니다.

<a id="metrics-ingest-api"></a>
### 지표 수집 { #metrics-ingest-api }

타입이 Prometheus API인 데이터 소스로 지표 데이터를 전송합니다. 전송한 지표는 분석 메뉴에서 조회할 수 있고, 단변량 시계열 이상탐지 앱의 입력으로도 사용할 수 있습니다.

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/metrics |

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| metrics | Array | O | 지표 목록. 비어 있을 수 없으며 1회 요청당 최대 5,000건 |
| metrics[].timestamp | Long | O | 메트릭 시각. 밀리초 epoch |
| metrics[].value | Double | O | 측정값 |
| metrics[].labels | Array | O | 라벨 목록. 라벨 조합이 시계열을, 그룹 라벨이 그룹을 결정 |
| metrics[].labels[].name | String | O | 라벨 이름. 영문자 또는 _로 시작하고 영문자, 숫자, _만 사용 |
| metrics[].labels[].value | String | O | 라벨 값. 쉼표와 등호는 사용 불가 |
| metrics[].metadata | Object | X | 부가 정보. 해석하지 않고 그대로 저장·전달. identityKey 키는 시스템이 사용하므로 사용 불가 |

성공하면 `header.isSuccessful`이 `true`로 반환되며 `body`는 없습니다. 요청이 거절되면 `header.isSuccessful`이 `false`로 반환되므로 `header`로 성공 여부를 판정합니다.

수집 규칙은 다음과 같습니다.

- 필수 필드 누락, 라벨 이름·값 규칙 위반, 1회 요청 5,000건 초과, 필수 헤더 누락은 요청 전체가 거절되며 어떤 항목도 저장되지 않습니다.
- 한 요청에 여러 시계열의 지표를 함께 담을 수 있습니다. 시계열은 라벨 조합으로 구분되므로 시계열마다 요청을 나눌 필요가 없습니다.
- 같은 시계열은 1분에 한 번만 보냅니다. 더 짧은 주기로 수집한다면 1분 평균으로 합쳐 보냅니다. 같은 분에 값이 여러 개 오면 먼저 도착한 값만 분석에 쓰이고 나머지는 버려집니다.
- `timestamp`는 밀리초 단위 epoch입니다. 초 단위로 보내면 잘못된 시각으로 저장됩니다.
- 데이터 소스에 그룹 라벨을 지정했다면 항상 그 라벨을 포함해 전송합니다. 라벨이 빠지면 의도한 그룹에 속하지 않습니다.
- `value`가 NaN 또는 Infinity인 항목은 저장하지 않고 건너뜁니다. 같은 요청의 나머지 항목은 정상 처리됩니다.
- 성공 응답은 수신 완료를 뜻합니다. 저장은 잠시 뒤 반영되며, 같은 요청을 다시 보내면 같은 데이터가 중복 저장될 수 있습니다.
- 전송이 지연된 데이터는 저장되지만 실시간 추론 대상에서 제외될 수 있습니다.

!!! tip "알아두기"
    적재는 전송 주기와 무관합니다. 다만 이 데이터 소스를 단변량 시계열 이상탐지 앱에 연결했다면 같은 시계열을 1분에 하나씩 끊김 없이 보내야 합니다. 앱이 지표를 1분 단위로 묶어 판정하므로, 그보다 긴 간격으로 보내면 빈 구간이 생겨 정확 모드에서 준비가 끝나지 않을 수 있습니다.
    학습에도 조건이 있습니다. 데이터 소스에 시계열이 하나뿐이면 학습이 실패하므로 시계열을 둘 이상 두어야 하고, 시계열마다 약 4시간 이상 끊김 없이 쌓여야 정상적으로 학습합니다.

<a id="ingest-error-codes"></a>
### 오류 코드 { #ingest-error-codes }

[공통 오류 코드](#auth-common-error-codes) 외에 Ingest API 전체에서 반환될 수 있는 오류 코드입니다.

| 코드 | 메시지 | 설명 |
| --- | --- | --- |
| -4000001 | Invalid request. | 요청 형식 오류. 필수 필드 누락, 값의 범위·형식 위반, 본문 JSON 파싱 실패, `X-NC-APP-KEY` 헤더 누락 |
| -4041101 | DataSource not found. | `dataSourceId`에 해당하는 데이터 소스가 없거나 다른 Appkey의 데이터 소스입니다. |

<a id="ingest-error-codes-snapshot"></a>
#### 스냅숏 업로드 { #ingest-error-codes-snapshot }

| 코드 | 메시지 | 설명 | 대상 API |
| --- | --- | --- | --- |
| -4000202 | Invalid file name. | `fileName`이 비어 있거나 영문자, 숫자, `.`, `_`, `-` 이외의 문자를 포함합니다. | init, complete |
| -4001107 | File size exceeds maximum limit. | `fileSize`가 10GB를 초과합니다. | init |
| -4001101 | Invalid data source type. | 파일 타입 데이터 소스가 아닙니다. | init |
| -4001103 | DataSource is busy. | 데이터 소스가 적재 중이거나 Event API가 활성화되어 있어 스냅숏을 업로드할 수 없습니다. | init, complete |
| -4001104 | Ingest job is already running. | 같은 데이터 소스에 진행 중인 스냅숏 업로드 작업이 있습니다. 완료되거나 취소된 뒤 다시 시도합니다. | init, complete |
| -4000201 | DataSource is not ready for ingest. | 데이터 소스가 적재를 시작할 수 있는 상태가 아닙니다. | complete |
| -4041102 | IngestJob not found. | `jobId`에 해당하는 작업이 없거나 다른 Appkey의 작업입니다. | complete, 업로드 취소, 작업 상태 조회 |
| -4001105 | IngestJob is in invalid status. | 작업이 업로드 중 상태가 아닙니다. 이미 완료되었거나 취소되었거나, 업로드 제한 시간이 지나 실패 처리된 작업입니다. init부터 다시 시작합니다. | complete |
| -4000203 | File name does not match. | init 때 보낸 `fileName`과 다릅니다. | complete |
| -4291101 | Too many requests. Please try again later. | 처리 대기 중인 작업이 많아 요청을 받을 수 없습니다. 잠시 후 다시 시도합니다. | init, complete |

<a id="ingest-error-codes-event"></a>
#### 이벤트 수집 { #ingest-error-codes-event }

| 코드 | 메시지 | 설명 | 대상 API |
| --- | --- | --- | --- |
| -4001101 | Invalid data source type. | 파일 타입 데이터 소스가 아닙니다. | 활성화, 비활성화 |
| -4000201 | DataSource is not ready for ingest. | 데이터 소스가 이벤트를 받을 수 있는 상태가 아닙니다. 스냅숏 적재가 진행 중인 경우 등입니다. | 활성화, 단건 전송, 다건 전송 |
| -4001104 | Ingest job is already running. | 스냅숏 업로드 작업이 진행 중이어서 활성화할 수 없습니다. | 활성화 |
| -4091103 | Stream API activation is in progress. Please wait. | 활성화가 진행 중입니다. 완료될 때까지 기다립니다. | 활성화 |
| -4091601 | Operation is in progress. Please wait for the current operation to complete. | 같은 데이터 소스에 다른 작업이 진행 중입니다. 완료된 뒤 다시 시도합니다. | 활성화 |
| -4000002 | Invalid strategy type. | 이벤트 수집을 지원하지 않는 데이터 소스 타입입니다. | 단건 전송, 다건 전송 |
| -4001109 | Stream API is not enabled. | Event API가 활성화되어 있지 않습니다. 활성화 API를 먼저 호출합니다. | 단건 전송, 다건 전송 |
| -4000204 | Invalid operation. | `operation`이 INSERT, UPDATE, DELETE가 아니거나, 기본 키가 없는 데이터 소스에 INSERT 이외의 작업을 보냈습니다. | 단건 전송 |
| -5004001 | Failed to serialize Kafka message. | 이벤트를 저장 형식으로 변환하지 못했습니다. | 단건 전송 |
| -5004002 | Kafka send timeout. | 이벤트 저장이 제한 시간 안에 끝나지 않았습니다. 잠시 후 다시 시도합니다. | 단건 전송 |
| -5004003 | Kafka send failed. | 이벤트 저장에 실패했습니다. 잠시 후 다시 시도합니다. | 단건 전송 |
| -5004004 | Kafka send interrupted. | 이벤트 저장이 중단되었습니다. 잠시 후 다시 시도합니다. | 단건 전송 |

다건 전송에서 항목별 오류는 `header`가 아니라 `body[].success`와 `body[].errorMessage`로 반환됩니다. 데이터 소스 검증에 실패하면 요청 전체가 위 코드로 거절됩니다.

<a id="ingest-error-codes-metrics"></a>
#### 지표 수집 { #ingest-error-codes-metrics }

| 코드 | 메시지 | 설명 |
| --- | --- | --- |
| -4000001 | Invalid request. | `metrics`가 비어 있거나 5,000건을 초과하거나, `timestamp`·`value`·`labels` 누락, 라벨 이름·값 규칙 위반, `X-NC-APP-KEY` 헤더 누락. 요청 전체가 거절됩니다. |
| -4000002 | Invalid strategy type. | 지표 수집을 지원하지 않는 데이터 소스 타입입니다. |
| -4000201 | DataSource is not ready for ingest. | 데이터 소스가 지표를 받을 수 있는 상태가 아닙니다. |
| -5004001 | Failed to serialize Kafka message. | 지표를 저장 형식으로 변환하지 못했습니다. |
| -5004004 | Kafka send interrupted. | 지표 저장이 중단되었습니다. 잠시 후 다시 시도합니다. |
| -5004005 | Kafka batch send partially failed. | 일부 지표의 저장이 실패했거나 제한 시간을 넘겼습니다. 요청을 다시 보내면 이미 저장된 지표가 중복될 수 있습니다. |

<a id="univariate-api"></a>
## 단변량 시계열 이상탐지 API { #univariate-api }

<a id="univariate-group-api"></a>
### 그룹 사용 시작·중지·삭제 { #univariate-group-api }

단변량 시계열 이상탐지 앱의 그룹을 사용 시작, 중지, 삭제합니다. 세 API의 요청 형식은 같고 경로만 다릅니다.

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId`는 콘솔 앱 상세에 표시되는 앱 ID입니다.

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| groupKey | Array | 조건부 | 대상 그룹을 지정하는 라벨 목록. 데이터 소스에 그룹 라벨을 지정한 경우에만 사용 |
| groupKey[].name | String | O | 그룹 라벨 이름. 데이터 소스에 지정한 그룹 라벨과 이름이 정확히 일치해야 함 |
| groupKey[].value | Array | O | 그 라벨의 값 목록. 값 하나가 그룹 하나에 대응 |

성공하면 `header.isSuccessful`이 `true`로 반환되며 `body`는 없습니다.

요청 규칙은 다음과 같습니다.

- 데이터 소스에 그룹 라벨을 지정하지 않았으면 `groupKey`를 보내지 않습니다. 데이터 소스 전체가 하나의 그룹이므로 그 그룹이 대상이 됩니다. `groupKey`를 함께 보내면 요청이 거절됩니다.
- 데이터 소스에 그룹 라벨을 지정했으면 `groupKey`는 필수이며, 보낸 라벨 이름의 집합이 데이터 소스의 그룹 라벨과 정확히 같아야 합니다. 같은 라벨 이름을 두 번 보내면 거절됩니다.
- 그룹 라벨이 여러 개면 값 목록을 같은 순서끼리 묶어 그룹을 만듭니다. 예를 들어 `rule_id`에 `["a", "b"]`, `instance_id`에 `["q", "w"]`를 보내면 `(a, q)`와 `(b, w)` 두 그룹이 대상입니다. 모든 라벨의 값 개수가 같아야 하며 다르면 거절됩니다.
- 값 목록이 비어 있으면 거절됩니다. 같은 그룹이 여러 번 지정되면 한 번만 처리됩니다.
- 등록되지 않은 그룹을 중지하거나 삭제하면 오류가 반환되며, 요청에 포함된 다른 그룹도 처리되지 않습니다.

!!! tip "알아두기"
    데이터 소스에 그룹 라벨을 지정하지 않았으면 앱 생성이 끝날 때 데이터 소스 전체가 그룹 하나로 등록되므로 이 API를 쓰지 않아도 동작합니다. 그룹 라벨을 지정했으면 그룹이 저절로 등록되지 않으므로, 사용 시작 API로 대상 그룹을 등록해야 탐지 결과를 받을 수 있습니다. 이 조작은 API로만 제공합니다. 등록된 그룹과 상태는 콘솔 앱 상세의 **그룹 목록** 탭에서 확인합니다.

!!! danger "주의"
    그룹을 중지해도 탐지 결과 전송이 멈추지는 않습니다. 그룹 목록에 표시되는 상태만 비활성화로 바뀝니다.
    삭제한 그룹은 상태 기록과 함께 사라지며 복구할 수 없습니다.

<a id="univariate-error-codes"></a>
### 오류 코드 { #univariate-error-codes }

[공통 오류 코드](#auth-common-error-codes) 외에 그룹 사용 시작·중지·삭제 API에서 반환될 수 있는 오류 코드입니다.

| 코드 | 메시지 | 설명 | 대상 API |
| --- | --- | --- | --- |
| -4000001 | Invalid request. | `groupKey` 규칙 위반. 그룹 라벨이 없는 데이터 소스에 `groupKey`를 보냈거나, 필요한 `groupKey`가 없거나, 라벨 이름 집합이 데이터 소스의 그룹 라벨과 다르거나, 라벨 이름 중복, 값 목록이 비어 있거나 개수가 다른 경우. `X-NC-APP-KEY` 헤더 누락 | 시작, 중지, 삭제 |
| -4041301 | ServingPipeline not found. | `servingPipelineId`에 해당하는 앱이 없거나 다른 Appkey의 앱입니다. | 시작, 중지, 삭제 |
| -4041101 | DataSource not found. | 앱에 연결된 지표 데이터 소스를 찾을 수 없습니다. | 시작, 중지, 삭제 |
| -4000201 | DataSource is not ready for ingest. | 지표 데이터 소스가 사용할 수 있는 상태가 아닙니다. | 시작, 중지, 삭제 |
| -4001302 | Serving pipeline is not active. | 앱이 활성 상태가 아닙니다. 앱 상태가 활성이 된 뒤 다시 시도합니다. | 시작 |
| -4041306 | Group entry not found or has no trainingPipelineId. | 요청한 그룹 중 등록되지 않은 그룹이 있습니다. 요청에 포함된 다른 그룹도 처리되지 않습니다. | 중지, 삭제 |

<a id="recommendation-api"></a>
## 추천 조회 API { #recommendation-api }

생성한 추천 시스템 앱에 추천 결과를 요청합니다. 사용자 이력이 충분하면 모델 기반(Sequential), 부족하면 속성 기반(Cold Start)으로 추론합니다.

<a id="recommendation-api-recommend"></a>
### 추천 요청 { #recommendation-api-recommend }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/recommend |

curl 예시:

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| userId | String | O | 추천 대상 사용자 ID. 익명 사용자에게 추천을 요청하려면 빈 문자열("") 지정 |
| context.currentItemKey | String | X | 현재 보고 있는 아이템 키 |
| context.recentlyViewed | Array | X | 최근 조회한 아이템 키 목록 |
| context.availableItems | Array | X | 추천 대상 아이템 키 목록. 지정하면 이 목록에 포함된 아이템 중에서만 추천 |
| context.pageType | String | X | 현재 페이지 유형(자유 형식. 예: home, item_detail) |
| context.sessionId | String | X | 세션 ID |
| context.impressions | Array | X | 사용자에게 추천 결과로 노출된 아이템 목록 |
| context.interactions | Array | X | 사용자가 아이템에 대해 수행한 행동 정보 |
| context.feedback | Array | X | 사용자가 아이템에 남긴 평가 |
| userAttributes | Object | X | 사용자 속성 정보(Cold Start 추론에 사용) |
| options.maxRecommendations | Integer | X | 최대 추천 수(1~100). 100을 초과하는 값은 오류 없이 100으로 조정, 미지정 시 100 적용. 추천 가능한 아이템이 이 값보다 적으면 실제 아이템 수만큼만 반환 |
| options.mode | String | X | 추론 방식 지정. sequential(이력 기반), cold_start(속성 기반), popular(인기 기반) 중 하나. 미지정 시 서버가 자동 결정 |
| options.longtail | Boolean | X | 인기가 낮은 항목까지 포함해 추천 다양성 향상. sequential일 때만 적용 |
| options.excludeItemKeys | Array | X | 추천에서 제외할 아이템 키 목록. 제외한 아이템은 최대 추천 수에 미포함 |

- `options.mode`를 지정하지 않으면 서버가 추론 방식을 정합니다. 이때 정해진 방식의 모델이 앱에 없으면 앱에 연동된 다른 방식으로 대신 추천합니다. 실제로 사용한 방식은 응답의 `body.metadata.inferenceType`에서 확인합니다.
- 요청한 방식의 모델이 앱에 없고 대신할 방식도 없으면 HTTP `503`과 결과 코드 `5030001`을 반환합니다. 앱에 어떤 모델이 만들어져 있는지와 학습이 끝났는지 확인한 뒤 다시 호출합니다. `options.mode`로 방식을 지정한 요청은 대체하지 않으므로 이 응답을 받을 수 있습니다.

<a id="recommendation-api-signal"></a>
#### 행동 신호 { #recommendation-api-signal }

`context.impressions`은 사용자에게 노출된 추천 정보를 바탕으로 추천 결과를 재정렬하는 데 사용됩니다.
`context.interactions`, `context.feedback`은 사용자가 추천 결과에 보인 행위를 전달하는 필드로, 사용자 행위 기반 데이터를 모델 추론에 반영합니다.

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

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| requestId | String | O | 해당 행동이 일어난 추천 응답의 body.metadata.requestId |
| itemKeys | Array | O | 노출한 아이템 키 목록. 노출 순서대로 입력하며 impressions에서 사용 |
| itemKey | String | O | 대상 아이템 키. interactions, feedback에서 사용 |
| type | String | O | interactions는 CLICK, CONVERSION. feedback은 POSITIVE, NEGATIVE |
| occurredAt | String | O | 행동이 일어난 시각 |

- `occurredAt`은 시간대 오프셋을 포함한 ISO 8601 형식으로 보냅니다. 오프셋이 없으면 오류로 처리됩니다.
- 세 필드 모두 `requestId`, `occurredAt`, 아이템 키가 모두 있어야 신호로 사용됩니다.
- 각 필드는 오래된 것부터 최신 순서로 전달합니다.
- `impressions`는 최대 10건이고 1건당 `itemKeys`는 최대 100개입니다. `interactions`와 `feedback`은 `type`별로 최대 10건입니다. 상한을 초과하면 요청이 거절됩니다.
- 행동 신호는 이번 추천 요청의 추론 입력으로만 사용하고 저장하지 않습니다. 같은 아이템의 `feedback`이 바뀌면 가장 최근 값만 반영되므로, 효과를 유지하려면 매 요청 다시 전송합니다.
- 반응 이벤트를 저장해 분석에 활용하려면 [추천 이벤트 API](#recommendation-event-api)를 함께 사용합니다.

!!! tip "알아두기"
    `userAttributes` 스키마는 향후 선호도 유도(Preference Elicitation) 구현 방향에 따라 수집 방식이나 필드 종류가 변경될 수 있습니다.

응답 예시:

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

| 필드 | 설명 |
| --- | --- |
| body.userId | 요청한 사용자 ID |
| body.recommendations[].itemKey | 추천 아이템 키 |
| body.recommendations[].score | 추천 점수(0.0~1.0) |
| body.recommendations[].position | 추천 순위 |
| body.metadata.modelVersion | 사용된 모델 버전 |
| body.metadata.requestId | 요청 추적 ID. 추천 이벤트 API 전송 시 이 값 사용 |
| body.metadata.inferenceType | 추론 유형. sequential(이력 기반), cold_start(속성 기반), popular(인기 기반) |
| body.metadata.abTestGroup | A/B 테스트 그룹(현재는 빈 값 반환) |

<a id="recommendation-error-codes"></a>
### 오류 코드 { #recommendation-error-codes }

[공통 오류 코드](#auth-common-error-codes) 외에 추천 요청에서 반환될 수 있는 오류 코드입니다.

| 코드 | 메시지 | 설명 |
| --- | --- | --- |
| -4004201 | Invalid request. 또는 거절 사유 | 요청 형식 오류. `userId` 누락, `maxRecommendations`가 1 미만, 카테고리 최소 추천 수 규칙 위반, 카탈로그에 없는 카테고리, `context`의 행동 신호 형식·개수 위반, `X-NC-APP-KEY` 헤더 누락 등. `resultMessage`에 거절 사유가 포함됩니다. |
| -4044201 | Recommendation app not found. | `appId`에 해당하는 추천 앱이 없거나 `X-NC-APP-KEY`와 앱이 일치하지 않습니다. |
| -4001301 | Invalid model type. | `appId`가 추천 시스템 앱이 아닙니다. |
| -4001302 | Serving pipeline is not active. | 앱이 활성 상태가 아닙니다. 앱 상태가 활성이 된 뒤 다시 시도합니다. |
| -5034201 | Recommendation model is not ready. | 요청한 추천 모드의 모델이 아직 앱에 연동되지 않았습니다. 첫 학습과 배포가 끝난 뒤 다시 시도합니다. |

<a id="recommendation-event-api"></a>
## 추천 이벤트 API { #recommendation-event-api }

추천 결과에 사용자가 보인 반응(클릭 등) 이벤트를 수집합니다. 적재된 이벤트 데이터로 추천 성공률을 분석할 수 있습니다.

<a id="recommendation-event-api-send"></a>
### 추천 이벤트 전송 { #recommendation-event-api-send }

| 메서드 | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/events |

curl 예시:

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

`requestId`, `itemKey`, `userId`는 추천 조회 API 응답에서 받은 값을 그대로 전달합니다.

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| eventType | O | 이벤트 유형. 자유롭게 정의 가능(예: CLICK, PURCHASE, IMPRESSION). 영문·숫자·밑줄(_)만 사용(^[A-Za-z0-9_]+$), 최대 64자. 대소문자 구분 없이 대문자로 정규화되어 저장. REQUEST, RESPONSE는 예약어로 사용 불가 |
| requestId | O | 추천 API 응답의 body.metadata.requestId 값(opaque string, 최대 128자) |
| itemKey | O | 사용자가 반응한 추천 아이템의 itemKey |
| userId | X | 추천 API 응답의 body.userId 값 |
| context | X | 이벤트 부가 정보(자유 형식 키-값. 예: 노출 위치 position, 지면 placement) |
| userAttributes | X | 사용자 속성 정보(자유 형식 키-값) |
| options | X | 부가 옵션(자유 형식 키-값) |

성공 응답(200)은 `header`만 반환합니다.

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  }
}
```

!!! tip "알아두기"
    - 성공 응답(200)은 수집 파이프라인이 이벤트를 수신했다는 의미이며, 분석 테이블 적재 완료를 보장하지 않습니다.
    - 이벤트 API 요청 후 데이터셋에 적재까지 최대 10분이 걸릴 수 있습니다.
    - 타임아웃 후 재시도하면 같은 이벤트가 중복 적재될 수 있습니다. 분석 시 중복 제거를 고려하세요.

<a id="recommendation-event-error-codes"></a>
### 오류 코드 { #recommendation-event-error-codes }

[공통 오류 코드](#auth-common-error-codes) 외에 추천 이벤트 전송에서 반환될 수 있는 오류 코드입니다.

| 코드 | 메시지 | 설명 |
| --- | --- | --- |
| -4004202 | Invalid event request. | 요청 형식 오류. `eventType`이 비어 있거나 64자를 초과하거나 영문자·숫자·`_` 이외의 문자를 포함하거나 예약어(REQUEST, RESPONSE)인 경우, `requestId`가 비어 있거나 128자를 초과하는 경우, `itemKey` 누락, 필수 헤더 누락 |
| -4044201 | Recommendation app not found. | `appId`에 해당하는 추천 앱이 없거나 `X-NC-APP-KEY`와 앱이 일치하지 않습니다. |
| -4001301 | Invalid model type. | `appId`가 추천 시스템 앱이 아닙니다. |
| -4001302 | Serving pipeline is not active. | 앱이 활성 상태가 아닙니다. |
| -4094201 | Recommendation event dataset is not configured for this app. | 앱에 추천 이벤트를 저장할 데이터 소스가 설정되어 있지 않습니다. |
| -5034202 | Event publish failed. | 이벤트 저장에 실패했습니다. 잠시 후 다시 시도합니다. |
| -5044201 | Event publish timed out. | 이벤트 저장이 제한 시간 안에 끝나지 않았습니다. 잠시 후 다시 시도합니다. |
