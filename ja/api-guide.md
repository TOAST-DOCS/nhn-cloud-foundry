<!-- machine_translated: true -->

<!-- pre-align:aligned sig=876ecae2be90 -->

<a id="foundry.api.guide"></a>

## Machine Learning > NHN Cloud Foundry > API ガイド { #foundry.api.guide }

NHN Cloud Foundry が提供する API について説明します。

| API | 説明 |
| --- | --- |
| Ingest API | 作成済みのデータソースへのデータ取り込み。スナップショットファイルのアップロードを提供 |
| レコメンデーション照会 API | 作成したレコメンデーションシステムアプリへの推薦結果のリクエスト |
| レコメンデーションイベント API | 推薦結果に対するユーザーの反応イベントの収集 |

<a id="auth.common"></a>

## 認証および共通事項 { #auth.common }

<a id="auth.common.preparation"></a>
### 事前準備 { #auth.common.preparation }

API を使用するには、**Appkey** と**認証トークン**が必要です。

- Appkey は、NHN Cloud コンソールの **[Machine Learning > NHN Cloud Foundry]** ページ上部の **[URL & Appkey]** メニューで確認できます。
- API は **gateway-public** エンドポイントを使用します。
- 認証トークン（`X-NHN-Authorization` ヘッダーの Bearer トークン）の発行方法については、[User Access Key トークン](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/user-access-key-token/) ガイドを参照してください。

<a id="auth.common.request"></a>
### リクエスト共通事項 { #auth.common.request }

必須ヘッダー:

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
### レスポンス共通事項 { #auth.common.response }

すべての API レスポンスは `header` と `body` で構成されます。

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

| フィールド | タイプ | 説明 |
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエストの成否 |
| header.resultCode | Integer | 結果コード。成功時は 0、失敗時はエラーコード |
| header.resultMessage | String | 結果メッセージ。成功時は SUCCESS、失敗時はエラー詳細 |
| body | Object/Array | API ごとのレスポンスデータ |

<a id="ingest.api"></a>

## Ingest API { #ingest.api }

Ingest API は、コンソールで作成済みのデータソースにデータを取り込むための API です。
アップロードしたファイルでデータソースのデータをすべて置き換えるスナップショットアップロード方式を提供します。

!!! danger "注意"
    データソースを新規に作成する API は提供していません。Ingest API を使用するには、コンソールであらかじめデータソースを作成する必要があります。また、FILE タイプのデータソースのみ使用できます。

<a id="ingest.snapshot"></a>
### スナップショットアップロード（ファイルアップロード） { #ingest.snapshot }

アップロードしたファイルの内容でデータソースのデータを**すべて置き換え**ます。アップロードは 3 段階で進みます。

!!! danger "注意"
    スナップショットアップロードは、データソースにすでに取り込まれているデータをすべて置き換えます。既存のデータは復元できません。

アップロード制限:

- 最大アップロードサイズ: **10GB**
- `100MB` 以下 → **単一アップロード（SINGLE）**
- `100MB` 超 → **マルチパートアップロード（MULTIPART）**
- `formPost` フィールドの値は、レスポンスに含まれる値を**そのまま**リクエストに使用します。

<a id="ingest.snapshot.init"></a>
#### 1. アップロード初期化（init） { #ingest.snapshot.init }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init |

大容量ファイルをストレージに直接アップロードするための署名済み一時 URL を発行します。ファイルサイズに応じて、単一 URL（SINGLE）またはマルチパート URL（MULTIPART）を返します。

curl の例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| fileName | String | O | ファイル名。使用可能文字: 英字、数字、ピリオド（.）、アンダースコア（_）、ハイフン（-） |
| fileSize | Long | O | ファイルサイズ（bytes）。最小 1、最大 10GB |
| contentType | String | X | Content-Type（デフォルト値: application/octet-stream） |

レスポンス例（SINGLE）:

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

レスポンス例（MULTIPART）:

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

!!! tip "ヒント"
    MULTIPART レスポンスにも `formPost` が含まれます。ただし、マルチパートアップロードでは `parts[].uploadUrl` のクエリパラメータ（`signature`/`expires`/`max_file_size`/`max_file_count`）でパートを送信するため、`formPost` は参考用であり、パートアップロード自体には使用しません。

| フィールド | 説明 |
| --- | --- |
| body.jobId | ジョブ ID。以降の complete/ステータス確認リクエストに使用 |
| body.uploadType | アップロードタイプ。SINGLE（100MB 以下）または MULTIPART（100MB 超） |
| body.uploadUrl | アップロード URL（単一アップロード時） |
| body.uploadId | マルチパートアップロード ID（マルチパートアップロード時） |
| body.partSize | パートサイズ（bytes、マルチパートアップロード時） |
| body.parts[].partNumber | パート番号（1 から開始） |
| body.parts[].uploadUrl | パートアップロード URL |
| body.parts[].headUrl | ETag 取得用 URL（アップロード完了後に HEAD リクエスト） |
| body.expiresAt | URL の有効期限 |
| body.formPost.objectPrefix | オブジェクト prefix（ファイル名の前に付くパス） |
| body.formPost.signature | HMAC-SHA1 署名 |
| body.formPost.expires | 有効期限（UNIX タイムスタンプ） |
| body.formPost.maxFileSize | 最大ファイルサイズ（bytes） |
| body.formPost.maxFileCount | 最大ファイル数 |

<a id="ingest.snapshot.upload.single"></a>
#### 2-A. 単一ファイルアップロード（100MB 以下） { #ingest.snapshot.upload.single }

init レスポンスの `uploadUrl` に multipart/form-data POST を送信します。
このリクエストは Object Storage に直接送信するため、別途の認証は不要です（`signature` が認証の役割を担います）。

curl の例:

```bash
curl -X POST "{uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={formPost.maxFileSize}" \
  -F "max_file_count={formPost.maxFileCount}" \
  -F "expires={formPost.expires}" \
  -F "signature={formPost.signature}" \
  -F "file=@./data.csv;filename=data.csv"
```

!!! danger "注意"
    `file` フィールドは必ずフォームデータの**末尾**に追加する必要があります。成功時は HTTP `201 Created` レスポンスを受け取ります。

<a id="ingest.snapshot.upload.multipart"></a>
#### 2-B. 大容量ファイルアップロード（100MB 超、MULTIPART） { #ingest.snapshot.upload.multipart }

レスポンスの `parts[]` 配列を受け取り、パートごとにアップロードします。
各パートは **(1) アップロード → (2) HEAD で ETag 取得 → (3) `partETags[]` に `partNumber` 昇順で収集** の順で処理します。

1. ファイルを `partSize`（デフォルト 100MB）単位で分割します。
2. パートごとに `parts[i].uploadUrl` のクエリパラメータ（`signature`、`expires`、`max_file_size`、`max_file_count`）を解析し、multipart/form-data で送信します（フィールド名 `file`、ファイル名は固定で `part`）。
3. アップロード成功後、`parts[i].headUrl` に `HEAD` リクエストを送信し、レスポンスヘッダーの `ETag` 値を収集します。
4. すべてのパートが完了したら、`partETags` 配列を `partNumber` 昇順で構成し、アップロード完了（complete）リクエストに含めて送信します。

パートアップロードの curl 例:

```bash
# 1) アップロード
curl -X POST "{parts[i].uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={max_file_size-from-query}" \
  -F "max_file_count={max_file_count-from-query}" \
  -F "expires={expires-from-query}" \
  -F "signature={signature-from-query}" \
  -F "file=@./part_i.bin;filename=part"

# 2) ETag の照会
curl -I "{parts[i].headUrl}" | grep -i '^etag:'
```

<a id="ingest.snapshot.complete"></a>
#### 3. アップロード完了（complete） { #ingest.snapshot.complete }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curl の例（単一アップロード）:

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

curl の例（マルチパートアップロード）:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| jobId | String | O | ジョブ ID（init レスポンスの jobId） |
| fileName | String | O | ファイル名 |
| uploadId | String | X | マルチパートアップロード ID（マルチパートアップロード時のみ必要） |
| partETags | Array | X | パートごとの ETag リスト（マルチパートアップロード時のみ必要、partNumber 順） |

レスポンス例:

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

| フィールド | 説明 |
| --- | --- |
| body.jobId | ジョブ ID。[ジョブステータス確認](#ingest.snapshot.job.status)に使用 |

<a id="ingest.snapshot.cancel"></a>
#### アップロードキャンセル { #ingest.snapshot.cancel }

| メソッド | URI |
| --- | --- |
| DELETE | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl の例（単一アップロード）:

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl の例（マルチパートアップロード）- クエリパラメータで `uploadId` を併せて渡します:

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest.snapshot.job.status"></a>
#### ジョブステータス確認 { #ingest.snapshot.job.status }

| メソッド | URI |
| --- | --- |
| GET | /api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curl の例:

```bash
curl "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

レスポンス例:

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

| フィールド | 説明 |
| --- | --- |
| body.jobId | ジョブ ID |
| body.dataSourceId | 対象データソース ID |
| body.jobType | ジョブタイプ。SNAPSHOT（スナップショット取り込み）または EVENT（変更イベント） |
| body.status | ジョブステータス。下記のステータス値を参照 |
| body.obsFilePath | OBS ファイルパス |
| body.statistics.totalRecords | 総レコード数 |
| body.statistics.failedRecords | 失敗レコード数 |
| body.statistics.successfulRecords | 成功レコード数 |
| body.statistics.successRate | 成功率（0.0〜1.0） |
| body.errorMessage | エラーメッセージ（失敗時） |
| body.createdDatetime | ジョブ作成日時 |
| body.startedDatetime | ジョブ開始日時 |
| body.completedDatetime | ジョブ完了日時 |
| body.modifiedDatetime | 最終更新日時 |

ジョブステータス（`status`）は次の値を取ります。

| 値 | 説明 |
| --- | --- |
| UPLOADING | ファイルアップロード中 |
| QUEUED | アップロード完了、取り込み待ち |
| STAGED | 処理準備完了 |
| RUNNING | データ取り込み中 |
| COMPLETED | ジョブ正常完了 |
| FAILED | ジョブ失敗 |

<a id="recommendation.api"></a>

## レコメンデーション照会 API { #recommendation.api }

作成したレコメンデーションシステムアプリにレコメンデーション結果をリクエストします。ユーザーの履歴が十分な場合はモデルベース (Normal Flow)、不足している場合は属性ベース (Cold Start) で推論します。

<a id="recommendation.api.recommend"></a>
### レコメンデーションリクエスト { #recommendation.api.recommend }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/recommend |

curl 例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| userId | String | O | レコメンデーション対象のユーザーID。匿名ユーザーにレコメンデーションをリクエストする場合は、空の文字列 ("") を指定します。 |
| context.currentItemKey | String | X | 現在表示中のアイテムキー |
| context.recentlyViewed | Array | X | 最近閲覧したアイテムキーのリスト |
| context.availableItems | Array | X | レコメンデーション対象のアイテムキーのリスト。指定した場合、このリストに含まれるアイテムの中からのみレコメンデーションします。 |
| context.pageType | String | X | 現在のページタイプ (自由形式。例: home、item_detail) |
| context.sessionId | String | X | セッションID |
| userAttributes | Object | X | ユーザー属性情報 (Cold Start 推論に使用) |
| options.maxRecommendations | Integer | X | 最大レコメンデーション数 (1〜100)。100を超える値はエラーなく100に調整されます。未指定の場合は100が適用されます。レコメンデーション可能なアイテムがこの値より少ない場合は、実際のアイテム数のみ返します。 |
| options.mode | String | X | 推論方式を指定します。sequential (履歴ベース)、cold_start (属性ベース)、popular (人気ベース) のいずれか。未指定の場合はサーバーが自動で決定します。 |
| options.longtail | Boolean | X | 人気の低いアイテムも含めてレコメンデーションの多様性を向上させます。sequential の場合のみ適用されます。 |
| options.excludeItemKeys | Array | X | レコメンデーションから除外するアイテムキーのリスト。除外したアイテムは最大レコメンデーション数に含まれません。 |

!!! tip "ヒント"
    `userAttributes` スキーマは、今後の選好度誘導 (Preference Elicitation) の実装方向に応じて、収集方式やフィールドの種類が変更される可能性があります。

レスポンス例:

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

| フィールド | 説明 |
| --- | --- |
| body.userId | リクエストしたユーザーID |
| body.recommendations[].itemKey | レコメンデーションアイテムキー |
| body.recommendations[].score | レコメンデーションスコア (0.0〜1.0) |
| body.recommendations[].position | レコメンデーション順位 |
| body.metadata.modelVersion | 使用されたモデルバージョン |
| body.metadata.requestId | リクエスト追跡ID。レコメンデーションイベントAPI送信時にこの値を使用します。 |
| body.metadata.inferenceType | 推論タイプ。sequential (履歴ベース)、cold_start (属性ベース)、popular (人気ベース) |
| body.metadata.abTestGroup | A/B テストグループ (現在は空の値を返します) |

<a id="recommendation.event.api"></a>

## 推薦イベント API { #recommendation.event.api }

推薦結果に対するユーザーの反応（クリックなど）のイベントを収集します。収集されたイベントデータを使用して、推薦の成功率を分析できます。

<a id="recommendation.event.api.send"></a>
### 推薦イベント送信 { #recommendation.event.api.send }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/events |

curl の例:

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

`requestId`、`itemKey`、`userId` は、推薦照会 API のレスポンスで受け取った値をそのまま渡します。

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| eventType | O | イベントの種類。自由に定義できます（例: CLICK、PURCHASE、IMPRESSION）。英字・数字・アンダースコア（_）のみ使用可能（^[A-Za-z0-9_]+$）、最大 64 文字。大文字小文字を区別せず大文字に正規化して保存されます。REQUEST、RESPONSE は予約語のため使用不可 |
| requestId | O | 推薦 API レスポンスの body.metadata.requestId の値（opaque string、最大 128 文字） |
| itemKey | O | ユーザーが反応した推薦アイテムの itemKey |
| userId | X | 推薦 API レスポンスの body.userId の値 |
| context | X | イベントの付加情報（自由形式のキー値。例: 表示位置 position、掲載面 placement） |
| userAttributes | X | ユーザー属性情報（自由形式のキー値） |
| options | X | 付加オプション（自由形式のキー値） |

成功レスポンス（200）は `header` のみを返します。

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  }
}
```

!!! tip "ヒント"
    - 成功レスポンス（200）は、収集パイプラインがイベントを受信したことを意味し、分析テーブルへの書き込み完了を保証するものではありません。
    - イベント API リクエスト後、データセットへの書き込みまで最大 10 分かかる場合があります。
    - タイムアウト後に再試行すると、同じイベントが重複して書き込まれる場合があります。分析時には重複排除を考慮してください。