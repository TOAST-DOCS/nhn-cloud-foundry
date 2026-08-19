<a id="foundry.api.guide"></a>

## Machine Learning > NHN Cloud Foundry > API 가이드 { #foundry.api.guide }

NHN Cloud Foundry가 제공하는 API를 설명합니다.

| API | 설명 |
| --- | --- |
| Ingest API | 이미 만든 데이터 소스에 데이터를 수집합니다. 스냅샷 파일 업로드를 제공합니다. |
| 추천 조회 API | 생성한 추천 시스템 앱에 추천 결과를 요청합니다. |
| 추천 이벤트 API | 추천 결과에 사용자가 보인 반응 이벤트를 수집합니다. |

<a id="auth.common"></a>

## 인증 및 공통 사항 { #auth.common }

<a id="auth.common.preparation"></a>

### 사전 준비 { #auth.common.preparation }

API를 사용하려면 **Appkey**와 **인증 토큰**이 필요합니다.

- Appkey는 NHN Cloud 콘솔의 **Machine Learning > NHN Cloud Foundry** 페이지 상단 **URL & Appkey** 메뉴에서 확인할 수 있습니다.
- API는 **gateway-public** 엔드포인트를 사용합니다.
- 인증 토큰(`X-NHN-Authorization` 헤더의 Bearer 토큰) 발급 방법은 [User Access Key 토큰](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/user-access-key-token/) 가이드를 참고하세요.

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

Ingest API는 콘솔에서 이미 만든 데이터 소스에 데이터를 적재하는 API입니다.
업로드한 파일로 데이터 소스의 데이터를 전부 교체하는 스냅샷 업로드 방식을 제공합니다.

!!! danger "주의"
    데이터 소스를 새로 만드는 API는 제공하지 않습니다. Ingest API를 사용하려면 콘솔에서 데이터 소스를 먼저 생성해야 하며, FILE 타입 데이터 소스만 사용할 수 있습니다.

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
| fileName | String | O | 파일 이름. 허용 문자: 영문, 숫자, 점(.), 언더스코어(_), 하이픈(-) |
| fileSize | Long | O | 파일 크기(bytes). 최소 1, 최대 10GB |
| contentType | String | X | Content-Type(기본값: application/octet-stream) |

Response(SINGLE):

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

Response(MULTIPART):

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
| body.jobId | 작업 ID. 이후 complete / 상태 조회 요청에 사용합니다 |
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
| partETags | Array | X | 파트별 ETag 목록(멀티파트 업로드 시에만 필요, partNumber 순) |

Response:

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
| body.jobId | 작업 ID. [작업 상태 조회](#ingest.snapshot.job.status)에 사용합니다 |

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

Response:

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
| UPLOADING | 파일을 업로드하고 있습니다. |
| QUEUED | 업로드가 완료되어 적재 대기 중입니다. |
| STAGED | 처리 준비가 완료되었습니다. |
| RUNNING | 데이터를 적재하고 있습니다. |
| COMPLETED | 작업이 정상적으로 완료되었습니다. |
| FAILED | 작업이 실패했습니다. |

<a id="recommendation.api"></a>

## 추천 조회 API { #recommendation.api }

생성한 추천 시스템 앱에 추천 결과를 요청합니다. 사용자 이력이 충분하면 모델 기반(Normal Flow), 부족하면 속성 기반(Cold Start)으로 추론합니다.

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
| userId | String | O | 추천 대상 사용자 ID |
| context | Object | X | 요청 맥락 정보(자유 형식 키-값). currentItemKey와 recentlyViewed는 추천 추론에 사용됩니다 |
| context.currentItemKey | String | X | 현재 보고 있는 아이템 키 |
| context.recentlyViewed | Array | X | 최근 조회한 아이템 키 목록 |
| context.pageType | String | X | 현재 페이지 유형(자유 형식. 예: home, item_detail) |
| context.sessionId | String | X | 세션 ID |
| userAttributes | Object | X | 사용자 속성 정보(자유 형식 키-값. 예: 직업, 관심 분야, 나이). Cold Start 추론에 사용됩니다 |
| options.maxRecommendations | Integer | X | 최대 추천 수(1~100, 기본값 10) |
| options.mode | String | X | 추론 방식 지정. sequential(이력 기반), cold_start(속성 기반), popular(인기 기반) 중 하나. 지정하지 않으면 서버가 자동으로 결정합니다 |
| options.longtail | Boolean | X | 인기가 낮은 항목까지 포함해 추천 다양성을 높입니다. sequential일 때만 적용됩니다 |

!!! tip "알아두기"
    `userAttributes` 스키마는 향후 선호도 유도(Preference Elicitation) 구현 방향에 따라 수집 방식이나 필드 종류가 변경될 수 있습니다.

Response:

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
| body.metadata.requestId | 요청 추적 ID. 추천 이벤트 API 전송 시 이 값을 사용합니다 |
| body.metadata.inferenceType | 추론 유형. sequential(이력 기반), cold_start(속성 기반), popular(인기 기반) |
| body.metadata.abTestGroup | A/B 테스트 그룹(현재는 빈 값으로 반환됩니다) |

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
| eventType | O | 이벤트 유형. 자유롭게 정의할 수 있습니다(예: CLICK, PURCHASE, IMPRESSION). 영문·숫자·언더스코어만 사용(^[A-Za-z0-9_]+$), 최대 64자. 대소문자는 구분하지 않으며 대문자로 정규화되어 저장됩니다. REQUEST, RESPONSE는 예약어로 사용할 수 없습니다 |
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
