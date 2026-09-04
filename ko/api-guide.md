<!-- pre-align:aligned sig=876ecae2be90 -->

<a id="foundry.api.guide"></a>
## Machine Learning > NHN Cloud Foundry > API 가이드 { #foundry.api.guide }

NHN Cloud Foundry가 제공하는 API를 설명합니다.

| API | 설명 |
| --- | --- |
| Ingest API | 이미 만든 데이터 소스에 데이터 수집. 스냅샷 파일 업로드, 이벤트 수집, 지표 수집 제공 |
| 추천 조회 API | 생성한 추천 시스템 앱에 추천 결과 요청 |
| 추천 이벤트 API | 추천 결과에 사용자가 보인 반응 이벤트 수집 |

<a id="auth.common"></a>
## 인증 및 공통 사항 { #auth.common }

<a id="auth.common.preparation"></a>
### 사전 준비 { #auth.common.preparation }

API를 사용하려면 **Appkey**와 **인증 토큰**이 필요합니다.

- Appkey는 NHN Cloud 콘솔의 **Machine Learning > NHN Cloud Foundry** 페이지 상단 **URL & Appkey** 메뉴에서 확인할 수 있습니다.
- API는 **gateway-public** 엔드포인트를 사용합니다.
- 인증 토큰(`X-NHN-Authorization` 헤더의 Bearer 토큰) 발급 방법은 [User Access Key 토큰](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/user-access-key-token/) 가이드를 참고합니다.

<a id="auth.common.request"></a>
### 요청 공통 사항 { #auth.common.request }

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

<a id="auth.common.response"></a>
### 응답 공통 사항 { #auth.common.response }

모든 API 응답은 `header`와 `body`로 구성됩니다.

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
| header.isSuccessful | Boolean | 요청 성공 여부 |
| header.resultCode | Integer | 결과 코드. 성공 시 0, 실패 시 오류 코드 |
| header.resultMessage | String | 결과 메시지. 성공 시 SUCCESS, 실패 시 오류 상세 |
| body | Object/Array | API별 응답 데이터 |

<a id="ingest.api"></a>
## Ingest API { #ingest.api }

Ingest API는 콘솔에서 이미 만든 데이터 소스에 데이터를 적재하는 API입니다. 데이터 소스 타입에 따라 다음 방식을 제공합니다.

| 방식 | 대상 데이터 소스 | 설명 |
| --- | --- | --- |
| 스냅샷 업로드 | 파일 | 업로드한 파일로 데이터를 전부 교체 |
| 이벤트 수집 | 파일 | 기존 데이터를 유지한 채 변경 이벤트를 건별로 추가 |
| 지표 수집 | Prometheus API | 지표(시계열) 데이터를 실시간으로 전송 |

!!! danger "주의"
    데이터 소스를 새로 만드는 API는 제공하지 않습니다. Ingest API를 사용하려면 콘솔에서 데이터 소스를 먼저 생성해야 합니다.

<a id="ingest.snapshot"></a>
### 스냅샷 업로드(파일 업로드) { #ingest.snapshot }

업로드한 파일의 내용으로 데이터 소스의 데이터를 **전부 교체**합니다. 업로드는 3단계로 진행됩니다.

!!! danger "주의"
    스냅샷 업로드는 데이터 소스에 이미 적재된 데이터를 모두 교체합니다. 기존 데이터는 복구할 수 없습니다.

업로드 제한:

- 최대 업로드 크기: **10GB**
- `100MB` 이하 → **단일 업로드(SINGLE)**
- `100MB` 초과 → **멀티파트 업로드(MULTIPART)**
- `formPost` 필드 값들은 응답에 포함된 값을 **그대로** 요청에 넣어 사용합니다.

<a id="ingest.snapshot.init"></a>
#### 1. 업로드 초기화(init) { #ingest.snapshot.init }

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

<a id="ingest.snapshot.upload.single"></a>
#### 2-A. 단일 파일 업로드(100MB 이하) { #ingest.snapshot.upload.single }

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

<a id="ingest.snapshot.upload.multipart"></a>
#### 2-B. 대용량 파일 업로드(100MB 초과, MULTIPART) { #ingest.snapshot.upload.multipart }

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

<a id="ingest.snapshot.complete"></a>
#### 3. 업로드 완료(complete) { #ingest.snapshot.complete }

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
| body.jobId | 작업 ID. [작업 상태 조회](#ingest.snapshot.job.status)에 사용 |

<a id="ingest.snapshot.cancel"></a>
#### 업로드 취소 { #ingest.snapshot.cancel }

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

<a id="ingest.snapshot.job.status"></a>
#### 작업 상태 조회 { #ingest.snapshot.job.status }

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
| body.jobType | 작업 타입. SNAPSHOT(스냅샷 적재) 또는 EVENT(변경 이벤트) |
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

<a id="event.ingest.api"></a>
### 이벤트 수집 { #event.ingest.api }

기존 데이터를 유지한 채 변경 이벤트를 전송합니다. 타입이 파일인 데이터 소스에서 사용하며, 콘솔에서 **Event API**를 먼저 활성화해야 합니다.

!!! danger "주의"
    Event API를 활성화하면 스냅샷 업로드가 차단됩니다. 또한 활성화 상태에서는 데이터 소스 스키마 변경(카탈로그 필드 추가)이 제한되므로, 필드를 추가하려면 Event API를 먼저 비활성화해야 합니다. 활성화·비활성화 방법은 [콘솔 유저 가이드](../console-user-guide/#datasource.detail.event)의 '이벤트 설정'을 참고합니다.

<a id="event.ingest.api.send"></a>
#### 이벤트 단건 전송 { #event.ingest.api.send }

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

<a id="event.ingest.api.batch"></a>
#### 이벤트 다건 전송 { #event.ingest.api.batch }

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

<a id="metrics.ingest.api"></a>
### 지표 수집 { #metrics.ingest.api }

타입이 Prometheus API인 데이터 소스로 지표 데이터를 전송합니다. 전송한 지표는 단변량 이상 탐지 앱의 입력으로 사용합니다.

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
| metrics[].metadata | Object | X | 부가 정보. 해석하지 않고 그대로 저장·전달. `identityKey` 키는 시스템이 사용하므로 사용 불가 |

성공하면 HTTP `202 Accepted`를 반환합니다.

수집 규칙은 다음과 같습니다.

- `timestamp`는 밀리초 단위 epoch입니다. 초 단위로 보내면 잘못된 시각으로 저장됩니다.
- 데이터 소스에 그룹 라벨을 지정했다면 항상 그 라벨을 포함해 전송합니다. 라벨이 빠지면 의도한 그룹에 속하지 않습니다.
- `value`가 NaN 또는 Infinity인 항목은 저장하지 않고 건너뜁니다. 같은 요청의 나머지 항목은 정상 처리됩니다.
- `202` 응답은 수신 완료를 뜻합니다. 저장은 잠시 뒤 반영되며, 같은 요청을 다시 보내면 같은 데이터가 중복 저장될 수 있습니다.
- 전송이 지연된 데이터는 저장되지만 실시간 추론 대상에서 제외될 수 있습니다.

<a id="recommendation.api"></a>
## 추천 조회 API { #recommendation.api }

생성한 추천 시스템 앱에 추천 결과를 요청합니다. 사용자 이력이 충분하면 모델 기반(Sequential), 부족하면 속성 기반(Cold Start)으로 추론합니다.

<a id="recommendation.api.recommend"></a>
### 추천 요청 { #recommendation.api.recommend }

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
| context.impressions | Array | X | 사용자에게 노출한 아이템 목록 |
| context.interactions | Array | X | 사용자가 아이템에 보인 반응 |
| context.feedback | Array | X | 사용자가 아이템에 남긴 평가 |
| userAttributes | Object | X | 사용자 속성 정보(Cold Start 추론에 사용) |
| options.maxRecommendations | Integer | X | 최대 추천 수(1~100). 100을 초과하는 값은 오류 없이 100으로 조정, 미지정 시 100 적용. 추천 가능한 아이템이 이 값보다 적으면 실제 아이템 수만큼만 반환 |
| options.mode | String | X | 추론 방식 지정. sequential(이력 기반), cold_start(속성 기반), popular(인기 기반) 중 하나. 미지정 시 서버가 자동 결정 |
| options.longtail | Boolean | X | 인기가 낮은 항목까지 포함해 추천 다양성 향상. sequential일 때만 적용 |
| options.excludeItemKeys | Array | X | 추천에서 제외할 아이템 키 목록. 제외한 아이템은 최대 추천 수에 미포함 |

<a id="recommendation.api.signal"></a>
#### 행동 신호 { #recommendation.api.signal }

`context.impressions`, `context.interactions`, `context.feedback`은 직전 추천 결과에 사용자가 보인 행동을 전달하는 필드입니다. 전달한 신호는 추천 결과의 순서를 조정하는 데 사용됩니다.

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
| requestId | String | O | 해당 행동이 일어난 추천 응답의 `body.metadata.requestId` |
| itemKeys | Array | O | 노출한 아이템 키 목록. 노출 순서대로 입력하며 impressions에서 사용 |
| itemKey | String | O | 대상 아이템 키. interactions, feedback에서 사용 |
| type | String | O | interactions는 CLICK, CONVERSION. feedback은 POSITIVE, NEGATIVE |
| occurredAt | String | O | 행동이 일어난 시각 |

- `occurredAt`은 시간대 오프셋을 포함한 ISO 8601 형식으로 보냅니다. 오프셋이 없으면 오류로 처리됩니다.
- 세 필드 모두 `requestId`, `occurredAt`, 아이템 키가 모두 있어야 신호로 사용됩니다.
- 각 필드는 오래된 것부터 최신 순서로 전달합니다.
- `impressions`는 최대 10건이고 1건당 `itemKeys`는 최대 100개입니다. `interactions`와 `feedback`은 `type`별로 최대 10건입니다. 상한을 초과하면 요청이 거절됩니다.
- 행동 신호는 이번 추천 요청의 추론 입력으로만 사용하고 저장하지 않습니다. 같은 아이템의 `feedback`이 바뀌면 가장 최근 값만 반영되므로, 효과를 유지하려면 매 요청 다시 전송합니다.
- 반응 이벤트를 저장해 분석에 활용하려면 [추천 이벤트 API](#recommendation.event.api)를 함께 사용합니다.

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

<a id="recommendation.event.api"></a>
## 추천 이벤트 API { #recommendation.event.api }

추천 결과에 사용자가 보인 반응(클릭 등) 이벤트를 수집합니다. 적재된 이벤트 데이터로 추천 성공률을 분석할 수 있습니다.

<a id="recommendation.event.api.send"></a>
### 추천 이벤트 전송 { #recommendation.event.api.send }

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
