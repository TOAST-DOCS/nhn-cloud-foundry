<!-- machine_translated: true -->

<!-- pre-align:aligned sig=0498f0e05eba -->

<a id="foundry-api-guide"></a>
## Machine Learning > NHN Cloud Foundry > APIガイド { #foundry-api-guide }

NHN Cloud Foundryが提供するAPIについて説明します。

| API | 説明 |
| --- | --- |
| Ingest API | 作成済みのデータソースへのデータ収集。スナップショットファイルアップロード、イベント収集、指標収集を提供 |
| 推薦照会 API | 作成した推薦システムアプリに推薦結果をリクエスト |
| 推薦イベント API | 推薦結果に対してユーザーが示した反応イベントを収集 |

<a id="api-version"></a>
## APIバージョン { #api-version }

APIはv2.0とv1.0の2つのバージョンを提供します。2つのバージョンは、リクエスト形式(ヘッダとリクエスト本文)と成功レスポンスの`body`が同じで、失敗時のHTTPステータスコードと結果コードが異なります。

| バージョン | リクエストパス | 状態 | レスポンス仕様 |
| --- | --- | --- | --- |
| [v2.0](#api-v2-guide) | `/api/v2.0/...` | 現在のバージョン | 成功・失敗ともにHTTPステータスコード`200`。失敗時は負の7桁の結果コード |
| [v1.0](#api-v1-guide) | `/api/v1.0/...` | サポート終了予定 | APIとエラーによってHTTPステータスコードが異なる。失敗時は正の結果コード |

v2.0の使用を推奨します。v1.0のサポート終了スケジュールは別途ご案内します。

<a id="api-v2-guide"></a>
## API v2.0ガイド { #api-v2-guide }

v2.0は成功・失敗ともにHTTPステータスコード`200`でレスポンスし、失敗の原因は負の7桁の結果コードで通知します。リクエスト形式はv1.0と同じです。

<a id="auth-common"></a>
### 認証および共通事項 { #auth-common }

<a id="auth-common-preparation"></a>
#### 事前準備 { #auth-common-preparation }

APIを使用するには、Appkeyと認証トークンが必要です。

- AppkeyはNHN Cloudコンソールの**Machine Learning > NHN Cloud Foundry**ページ上部の**URL & Appkey**メニューで確認できます。
- APIは**gateway-public**エンドポイントを使用します。
- 認証トークン(`X-NHN-Authorization`ヘッダのBearerトークン)の発行方法については、[User Access Keyトークン](/nhncloud/ja/public-api/user-access-key-token/)ガイドを参照してください。

<a id="auth-common-request"></a>
#### リクエスト共通事項 { #auth-common-request }

必須ヘッダ:

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
#### レスポンス共通事項 { #auth-common-response }

すべてのAPIはHTTPステータスコード`200`でレスポンスし、レスポンス本文は`header`と`body`で構成されます。リクエストが拒否された場合や処理に失敗した場合も、HTTPステータスコードは`200`であるため、成否はHTTPステータスコードではなく`header.isSuccessful`で判定します。

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
| header.isSuccessful | Boolean | リクエストの成否。失敗した場合は`false` |
| header.resultCode | Integer | 結果コード。成功時は`0`、失敗時は負のエラーコード |
| header.resultMessage | String | 結果メッセージ。成功時は`SUCCESS`、失敗時はエラーの原因 |
| body | Object/Array | API別レスポンスデータ。失敗時は`null` |

失敗レスポンスの例:

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

すべてのAPIに共通のエラーコードは以下の[共通エラーコード](#auth-common-error-codes)に、API別のエラーコードは各APIセクション末尾の**エラーコード**にあります。

- 認証トークンがない場合や有効期限が切れた場合も、HTTP `200`で失敗レスポンスが返されます。
- APIがリクエストを処理する前に拒否された場合、HTTPステータスコードが`200`ではなく、レスポンス本文も上記の形式と異なる場合があります。この場合も失敗として処理します。

!!! danger "注意"
    HTTPステータスコードで成否を判定すると、失敗レスポンスまで成功として処理されます。クライアントは必ず`header.isSuccessful`で成否を判定し、失敗の原因は`header.resultCode`で区別します。

<a id="auth-common-error-codes"></a>
#### 共通エラーコード { #auth-common-error-codes }

すべてのAPIで返される可能性があるエラーコードです。API別のエラーコードは各APIセクション末尾の**エラーコード**を参照してください。

エラーコードの先頭3桁はHTTPステータスコードと同じ意味です。400番台のコードはリクエストに問題がある場合であるため、`resultMessage`を確認してリクエストを修正し、再送します。500番台のコードはサービス側の問題であるため、しばらく後に同じリクエストを再試行し、それでも失敗が続く場合はサポートセンターにお問い合わせください。

| コード | メッセージ | 説明 |
| --- | --- | --- |
| -4010000 | Unauthorized | 認証失敗。認証トークンがない、形式が誤っている、有効期限が切れているか、トークンが示すAppkeyがリクエスト対象と異なります。トークンを再発行してリクエストします。 |
| -4040000 | Not Found | リクエストパスが存在しません。URIを確認します。 |
| -4050000 | Method Not Allowed | パスは正しいですが、HTTPメソッドが異なります。 |
| -4150000 | Unsupported Media Type | リクエスト本文のContent-Typeがサポートされていません。`application/json`で送信します。 |
| -5000000 | Internal Server Error | サーバー内部エラーです。しばらく後に再試行し、それでも続く場合はサポートセンターにお問い合わせください。 |
| -5030101 | Authentication service is not ready. Please retry. | 認証サービスの準備ができていません。しばらく後に再試行します。 |

<a id="ingest-api"></a>
### Ingest API { #ingest-api }

Ingest APIは、コンソールで作成済みのデータソースにデータを投入するAPIです。データソースのタイプ別に次の方式を提供します。

| 方式 | 対象データソース | 説明 |
| --- | --- | --- |
| スナップショットアップロード | ファイル | アップロードしたファイルでデータをすべて置き換え |
| イベント収集 | ファイル | 既存データを維持しながら変更イベントを1件ずつ追加 |
| 指標収集 | Prometheus API | 指標(時系列)データをリアルタイムで転送 |

!!! danger "注意"
    データソースを新規作成するAPIは提供していません。Ingest APIを使用するには、コンソールでデータソースを先に作成する必要があります。

<a id="ingest-snapshot"></a>
#### スナップショットアップロード(ファイルアップロード) { #ingest-snapshot }

アップロードしたファイルの内容でデータソースのデータを**すべて置換**します。アップロードは3段階で進行します。

!!! danger "注意"
    スナップショットアップロードは、データソースにすでに格納されているデータをすべて置換します。既存のデータは復元できません。

アップロード制限:

- 最大アップロードサイズ: **10GB**
- `100MB`以下 → **単一アップロード(SINGLE)**
- `100MB`超 → **マルチパートアップロード(MULTIPART)**
- `formPost`フィールドの値は、レスポンスに含まれる値を**そのまま**リクエストに含めて使用します。

<a id="ingest-snapshot-init"></a>
##### 1. アップロード初期化(init) { #ingest-snapshot-init }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/init |

大容量ファイルをストレージに直接アップロードするための署名済み一時URLを発行します。ファイルサイズに応じて、単一URL(SINGLE)またはマルチパートURL(MULTIPART)を返します。

curl例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| fileName | String | O | ファイル名。使用可能な文字: 英字、数字、ピリオド(.)、アンダースコア(_)、ハイフン(-) |
| fileSize | Long | O | ファイルサイズ(bytes)。最小1、最大10GB |
| contentType | String | X | Content-Type(デフォルト: application/octet-stream) |

レスポンス例(SINGLE):

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

レスポンス例(MULTIPART):

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

!!! tip "ポイント"
    MULTIPARTレスポンスにも`formPost`が含まれます。ただし、マルチパートアップロードは`parts[].uploadUrl`のクエリパラメータ(`signature`/`expires`/`max_file_size`/`max_file_count`)でパートを転送するため、`formPost`は参考用であり、パートアップロード自体には使用しません。

| フィールド | 説明 |
| --- | --- |
| body.jobId | ジョブID。以降のcomplete/ステータス照会リクエストに使用 |
| body.uploadType | アップロードタイプ。SINGLE(100MB以下)またはMULTIPART(100MB超) |
| body.uploadUrl | アップロードURL(単一アップロード時) |
| body.uploadId | マルチパートアップロードID(マルチパートアップロード時) |
| body.partSize | パートサイズ(bytes、マルチパートアップロード時) |
| body.parts[].partNumber | パート番号(1から開始) |
| body.parts[].uploadUrl | パートアップロードURL |
| body.parts[].headUrl | ETag照会用URL(アップロード完了後にHEADリクエスト) |
| body.expiresAt | URL有効期限 |
| body.formPost.objectPrefix | オブジェクトプレフィックス(ファイル名の前に付くパス) |
| body.formPost.signature | HMAC-SHA1署名 |
| body.formPost.expires | 有効期限(UNIXタイムスタンプ) |
| body.formPost.maxFileSize | 最大ファイルサイズ(bytes) |
| body.formPost.maxFileCount | 最大ファイル数 |

<a id="ingest-snapshot-upload-single"></a>
##### 2-A. 単一ファイルアップロード(100MB以下) { #ingest-snapshot-upload-single }

initレスポンスの`uploadUrl`にmultipart/form-data POSTを送信します。
このリクエストはObject Storageに直接送信するため、別途認証は不要です(`signature`が認証の役割を果たします)。

curl例:

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
    `file`フィールドは必ずフォームデータの最後に追加してください。成功時はHTTP `201 Created`レスポンスを受け取ります。

<a id="ingest-snapshot-upload-multipart"></a>
##### 2-B. 大容量ファイルアップロード(100MB超、MULTIPART) { #ingest-snapshot-upload-multipart }

レスポンスの`parts[]`配列を受け取り、パートごとにアップロードします。
各パートは(1) アップロード → (2) HEADでETag照会 → (3) `partETags[]`に`partNumber`昇順で収集の順序で処理します。

1. ファイルを`partSize`(デフォルト100MB)単位で分割します。
2. パートごとに`parts[i].uploadUrl`のクエリパラメータ(`signature`、`expires`、`max_file_size`、`max_file_count`)を解析し、multipart/form-dataで転送します(フィールド名`file`、ファイル名固定`part`)。
3. アップロード成功後、`parts[i].headUrl`に`HEAD`リクエストを送信し、レスポンスヘッダの`ETag`値を収集します。
4. すべてのパートが完了したら、`partETags`配列を`partNumber`昇順で構成し、アップロード完了(complete)リクエストに含めて送信します。

パートアップロードcurl例:

```bash
# 1) アップロード
curl -X POST "{parts[i].uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={max_file_size-from-query}" \
  -F "max_file_count={max_file_count-from-query}" \
  -F "expires={expires-from-query}" \
  -F "signature={signature-from-query}" \
  -F "file=@./part_i.bin;filename=part"

# 2) ETag照会
curl -I "{parts[i].headUrl}" | grep -i '^etag:'
```

<a id="ingest-snapshot-complete"></a>
##### 3. アップロード完了(complete) { #ingest-snapshot-complete }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curl例(単一アップロード):

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

curl例(マルチパートアップロード):

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| jobId | String | O | ジョブID(initレスポンスのjobId) |
| fileName | String | O | ファイル名 |
| uploadId | String | X | マルチパートアップロードID(マルチパートアップロード時のみ必要) |
| partETags | Array | X | パートごとのETag一覧(マルチパートアップロード時のみ必要、partNumber順) |

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
| body.jobId | ジョブID。[ジョブステータス照会](#ingest-snapshot-job-status)に使用 |

<a id="ingest-snapshot-cancel"></a>
##### アップロードキャンセル { #ingest-snapshot-cancel }

| メソッド | URI |
| --- | --- |
| DELETE | /api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curl例(単一アップロード):

```bash
curl -X DELETE "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curl例(マルチパートアップロード):

クエリパラメータで`uploadId`を合わせて渡します。

```bash
curl -X DELETE "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest-snapshot-job-status"></a>
##### ジョブステータス照会 { #ingest-snapshot-job-status }

| メソッド | URI |
| --- | --- |
| GET | /api/v2.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curl例:

```bash
curl "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
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
| body.jobId | ジョブID |
| body.dataSourceId | 対象データソースID |
| body.jobType | ジョブタイプ。SNAPSHOT(スナップショット格納)またはEVENT(変更イベント) |
| body.status | ジョブステータス。以下のステータス値を参照 |
| body.obsFilePath | OBSファイルパス |
| body.statistics.totalRecords | 総レコード数 |
| body.statistics.failedRecords | 失敗レコード数 |
| body.statistics.successfulRecords | 成功レコード数 |
| body.statistics.successRate | 成功率(0.0～1.0) |
| body.errorMessage | エラーメッセージ(失敗時) |
| body.createdDatetime | ジョブ作成日時 |
| body.startedDatetime | ジョブ開始日時 |
| body.completedDatetime | ジョブ完了日時 |
| body.modifiedDatetime | 最終更新日時 |

ジョブステータス(`status`)は次の値を取ります。

| 値 | 説明 |
| --- | --- |
| UPLOADING | ファイルアップロード中 |
| QUEUED | アップロード完了、格納待ち |
| STAGED | 処理準備完了 |
| RUNNING | データ格納中 |
| COMPLETED | ジョブ正常完了 |
| FAILED | ジョブ失敗 |

<a id="event-ingest-api"></a>
#### イベント収集 { #event-ingest-api }

既存のデータを保持したまま変更イベントを転送します。タイプがファイルのデータソースで使用し、**Event API**を先に有効化する必要があります。有効化はコンソールのイベント設定タブまたは以下の有効化APIで行います。

!!! danger "注意"
    Event APIを有効化すると、スナップショットアップロードがブロックされます。またスキーマ変更が制限されるため、スキーマを変更するにはEvent APIを先に無効化する必要があります。有効化・無効化の方法は[コンソールユーザーガイド](./console-user-guide/#datasource-detail-event)の「イベント設定」を参照してください。

<a id="event-ingest-api-enable"></a>
##### Event API 有効化・無効化 { #event-ingest-api-enable }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/enable |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/disable |

curl例:

```bash
curl -X POST "https://{gateway-public-host}/api/v2.0/data-sources/{dataSourceId}/ingest/events/enable" \
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
    "enabled": false,
    "status": "ENABLING"
  }
}
```

| フィールド | 説明 |
| --- | --- |
| body.enabled | イベント収集が可能かどうか |
| body.status | 有効化の状態。DISABLED、ENABLING、ENABLED、ENABLE_FAILED |

- 有効化は非同期で進行します。リクエスト直後のレスポンスは`enabled`が`false`、`status`が`ENABLING`であり、`ENABLED`になってからイベントを収集します。
- すでにアクティブなデータソースに再度有効化をリクエストすると、状態をそのまま返します。非アクティブ状態に無効化をリクエストする場合も同様です。複数回呼び出しても結果は変わりません。
- ただし、有効化が進行中(`ENABLING`)のときに再度有効化をリクエストすると拒否されます。進行状況はコンソールのイベント設定タブで確認できます。
- 存在しないデータソースで呼び出すと、データソースが見つからないというエラーが返されます。

<a id="event-ingest-api-send"></a>
##### イベント単件転送 { #event-ingest-api-send }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events |

curl例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| operation | String | O | 操作タイプ。INSERT、UPDATE、DELETEのいずれか |
| data | Object | O | イベントデータ。データソーススキーマのフィールド名をキーとして使用 |
| eventTimestamp | String | X | イベント発生時刻。省略時はサーバー受信時刻を使用 |

- `operation`は大文字・小文字を区別しません。許可されていない値を送信するとリクエストが拒否されます。
- データソースに主キーフィールドを指定していない場合は、`INSERT`のみ送信できます。`UPDATE`と`DELETE`は拒否されます。主キーを指定したデータソースは、3つの操作をすべて使用できます。

レスポンス例:

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

| フィールド | 説明 |
| --- | --- |
| body.eventId | イベントID |
| body.success | 成功かどうか |
| body.errorMessage | 失敗時のエラーメッセージ |

<a id="event-ingest-api-batch"></a>
##### イベント複数件転送 { #event-ingest-api-batch }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/events/batch |

複数件の変更イベントを一度に転送します。1回のリクエストにつき最大5,000件まで転送できます。

curl例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| events | Array | O | イベント一覧。1回のリクエストにつき最大5,000件で、各項目のフィールドは単件転送と同じ |

レスポンスの`body`はイベントごとの処理結果の配列です。

<a id="metrics-ingest-api"></a>
#### 指標収集 { #metrics-ingest-api }

タイプがPrometheus APIのデータソースに指標データを転送します。転送した指標は分析メニューで照会でき、単変量時系列異常検出アプリの入力としても使用できます。

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/data-sources/{dataSourceId}/ingest/metrics |

curlの例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| metrics | Array | O | 指標一覧。空にすることはできず、1回のリクエストあたり最大5,000件 |
| metrics[].timestamp | Long | O | 指標の時刻。ミリ秒epoch |
| metrics[].value | Double | O | 測定値 |
| metrics[].labels | Array | O | ラベル一覧。ラベルの組み合わせが時系列を、グループラベルがグループを決定します |
| metrics[].labels[].name | String | O | ラベル名。英字または_で始まり、英字、数字、_のみ使用可能 |
| metrics[].labels[].value | String | O | ラベル値。カンマと等号は使用不可 |
| metrics[].metadata | Object | X | 付加情報。解釈せずそのまま保存・転送します。identityKeyキーはシステムが使用するため使用不可 |

成功すると`header.isSuccessful`が`true`で返され、`body`はありません。リクエストが拒否された場合は`header.isSuccessful`が`false`で返されるため、`header`で成功かどうかを判定します。

収集ルールは次のとおりです。

- 必須フィールドの欠落、ラベル名・値のルール違反、1回のリクエスト5,000件超過、必須ヘッダの欠落がある場合は、リクエスト全体が拒否され、いかなる項目も保存されません。
- 1つのリクエストに複数の時系列の指標をまとめて含めることができます。時系列はラベルの組み合わせで区別されるため、時系列ごとにリクエストを分ける必要はありません。
- 同じ時系列は1分に1回だけ送信します。より短い周期で収集する場合は、1分間の平均にまとめて送信します。同じ分に複数の値が届いた場合、最初に到着した値のみ分析に使用され、残りは破棄されます。
- `timestamp`はミリ秒単位のepochです。秒単位で送信すると、誤った時刻として保存されます。
- データソースにグループラベルを指定した場合は、常にそのラベルを含めて転送します。ラベルが欠落すると、意図したグループに属しません。
- `value`がNaNまたはInfinityの項目は保存せずスキップします。同じリクエストの残りの項目は正常に処理されます。
- 成功レスポンスは受信完了を意味します。保存は少し後に反映され、同じリクエストを再送すると同じデータが重複して保存される可能性があります。
- 転送が遅延したデータは保存されますが、リアルタイム推論の対象から除外される可能性があります。

!!! tip "ポイント"
    蓄積は転送周期に関係ありません。ただし、このデータソースを単変量時系列異常検出アプリに連携している場合は、同じ時系列を1分に1つずつ途切れなく送信する必要があります。アプリが指標を1分単位にまとめて判定するため、それより長い間隔で送信すると空の区間が生じ、精確モードで準備が完了しない場合があります。
    学習にも条件があります。データソースに時系列が1つだけの場合は学習が失敗するため、時系列を2つ以上設ける必要があり、時系列ごとに約4時間以上途切れなく蓄積されている必要があります。

<a id="ingest-error-codes"></a>
#### エラーコード { #ingest-error-codes }

[共通エラーコード](#auth-common-error-codes)以外に、Ingest API全体で返される可能性があるエラーコードです。

| コード | メッセージ | 説明 |
| --- | --- | --- |
| -4000001 | Invalid request. | リクエスト形式エラー。必須フィールドの欠落、値の範囲・形式違反、本文JSONのパース失敗、`X-NC-APP-KEY`ヘッダの欠落 |
| -4041101 | DataSource not found. | `dataSourceId`に該当するデータソースが存在しないか、別のAppkeyのデータソースです。 |

<a id="ingest-error-codes-snapshot"></a>
##### スナップショットアップロード { #ingest-error-codes-snapshot }

| コード | メッセージ | 説明 | 対象API |
| --- | --- | --- | --- |
| -4000202 | Invalid file name. | `fileName`が空か、英字、数字、`.`、`_`、`-`以外の文字を含んでいます。 | init、complete |
| -4001107 | File size exceeds maximum limit. | `fileSize`が10GBを超えています。 | init |
| -4001101 | Invalid data source type. | ファイルタイプのデータソースではありません。 | init |
| -4001103 | DataSource is busy. | データソースがロード中か、Event APIが有効化されているため、スナップショットをアップロードできません。 | init、complete |
| -4001104 | Ingest job is already running. | 同じデータソースに進行中のスナップショットアップロードジョブがあります。完了またはキャンセルされた後に再試行します。 | init、complete |
| -4000201 | DataSource is not ready for ingest. | データソースがロードを開始できる状態ではありません。 | complete |
| -4041102 | IngestJob not found. | `jobId`に該当するジョブが存在しないか、別のAppkeyのジョブです。 | complete、アップロードキャンセル、ジョブステータス照会 |
| -4001105 | IngestJob is in invalid status. | ジョブがアップロード中の状態ではありません。すでに完了またはキャンセルされたか、アップロードの制限時間を超えて失敗処理されたジョブです。initから再度開始します。 | complete |
| -4000203 | File name does not match. | initで送信した`fileName`と異なります。 | complete |
| -4291101 | Too many requests. Please try again later. | 処理待ちのジョブが多く、リクエストを受け付けることができません。しばらくしてから再試行します。 | init、complete |

<a id="ingest-error-codes-event"></a>
##### イベント収集 { #ingest-error-codes-event }

| コード | メッセージ | 説明 | 対象API |
| --- | --- | --- | --- |
| -4001101 | Invalid data source type. | ファイルタイプのデータソースではありません。 | 有効化、無効化 |
| -4000201 | DataSource is not ready for ingest. | データソースがイベントを受け付けられる状態ではありません。スナップショットのロードが進行中の場合などです。 | 有効化、単件転送、複数件転送 |
| -4001104 | Ingest job is already running. | スナップショットアップロードジョブが進行中のため、有効化できません。 | 有効化 |
| -4091103 | Stream API activation is in progress. Please wait. | 有効化が進行中です。完了するまでお待ちください。 | 有効化 |
| -4091601 | Operation is in progress. Please wait for the current operation to complete. | 同じデータソースで別のジョブが進行中です。完了した後に再試行します。 | 有効化 |
| -4000002 | Invalid strategy type. | イベント収集をサポートしていないデータソースタイプです。 | 単件転送、複数件転送 |
| -4001109 | Stream API is not enabled. | Event APIが有効化されていません。有効化APIを先に呼び出します。 | 単件転送、複数件転送 |
| -4000204 | Invalid operation. | `operation`がINSERT、UPDATE、DELETEでないか、主キーがないデータソースにINSERT以外の操作を送信しました。 | 単件転送 |
| -5004001 | Failed to serialize Kafka message. | イベントを保存形式に変換できませんでした。 | 単件転送 |
| -5004002 | Kafka send timeout. | イベントの保存が制限時間内に完了しませんでした。しばらくしてから再試行します。 | 単件転送 |
| -5004003 | Kafka send failed. | イベントの保存に失敗しました。しばらくしてから再試行します。 | 単件転送 |
| -5004004 | Kafka send interrupted. | イベントの保存が中断されました。しばらくしてから再試行します。 | 単件転送 |

複数件転送における項目ごとのエラーは、`header`ではなく`body[].success`と`body[].errorMessage`で返されます。データソースの検証に失敗した場合、リクエスト全体が上記のコードで拒否されます。

<a id="ingest-error-codes-metrics"></a>
##### 指標収集 { #ingest-error-codes-metrics }

| コード | メッセージ | 説明 |
| --- | --- | --- |
| -4000001 | Invalid request. | `metrics`が空か5,000件を超えるか、`timestamp`・`value`・`labels`の欠落、ラベル名・値のルール違反、`X-NC-APP-KEY`ヘッダの欠落。リクエスト全体が拒否されます。 |
| -4000002 | Invalid strategy type. | 指標収集をサポートしていないデータソースタイプです。 |
| -4000201 | DataSource is not ready for ingest. | データソースが指標を受け付けられる状態ではありません。 |
| -5004001 | Failed to serialize Kafka message. | 指標を保存形式に変換できませんでした。 |
| -5004004 | Kafka send interrupted. | 指標の保存が中断されました。しばらくしてから再試行します。 |
| -5004005 | Kafka batch send partially failed. | 一部の指標の保存が失敗したか、制限時間を超えました。リクエストを再送すると、すでに保存された指標が重複する可能性があります。 |

<a id="univariate-api"></a>
### 単変量時系列異常検出 API { #univariate-api }

<a id="univariate-group-api"></a>
#### グループ使用開始・停止・削除 { #univariate-group-api }

単変量時系列異常検出アプリのグループを使用開始、停止、削除します。3つのAPIのリクエスト形式は同じで、パスのみ異なります。

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v2.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId`はコンソールアプリ詳細に表示されるアプリIDです。

curlの例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| groupKey | Array | 条件付き | 対象グループを指定するラベルのリスト。データソースにグループラベルを指定した場合にのみ使用 |
| groupKey[].name | String | O | グループラベル名。データソースに指定したグループラベルと名前が正確に一致する必要がある |
| groupKey[].value | Array | O | そのラベルの値のリスト。値1つがグループ1つに対応 |

成功すると`header.isSuccessful`が`true`で返され、`body`はありません。

リクエストルールは次のとおりです。

- データソースにグループラベルを指定していない場合は、`groupKey`を送信しません。データソース全体が1つのグループとなり、そのグループが対象になります。`groupKey`を一緒に送信するとリクエストが拒否されます。
- データソースにグループラベルを指定した場合は、`groupKey`は必須で、送信したラベル名のセットがデータソースのグループラベルと正確に一致する必要があります。同じラベル名を2回送信すると拒否されます。
- グループラベルが複数ある場合は、値のリストを同じ順序でまとめてグループを作成します。例えば、`rule_id`に`["a", "b"]`、`instance_id`に`["q", "w"]`を送信すると、`(a, q)`と`(b, w)`の2つのグループが対象になります。すべてのラベルの値の数が同じである必要があり、異なる場合は拒否されます。
- 値のリストが空の場合は拒否されます。同じグループが複数回指定された場合は、1回のみ処理されます。
- 登録されていないグループを停止または削除するとエラーが返され、リクエストに含まれる他のグループも処理されません。

!!! tip "ポイント"
    データソースにグループラベルを指定していない場合は、アプリ作成完了時にデータソース全体がグループ1つとして登録されるため、このAPIを使用しなくても動作します。グループラベルを指定した場合は、グループが自動的に登録されないため、使用開始APIで対象グループを登録することで、検出結果を受け取ることができます。この操作はAPIでのみ提供します。登録されたグループと状態はコンソールアプリ詳細の**グループ一覧**タブで確認します。

!!! danger "注意"
    グループを停止しても、検出結果の転送は停止しません。グループ一覧に表示される状態のみ非アクティブに変わります。
    削除したグループは状態の記録とともに消え、復元することはできません。

<a id="univariate-error-codes"></a>
#### エラーコード { #univariate-error-codes }

[共通エラーコード](#auth-common-error-codes)以外に、グループ使用開始・停止・削除APIで返される可能性があるエラーコードです。

| コード | メッセージ | 説明 | 対象API |
| --- | --- | --- | --- |
| -4000001 | Invalid request. | `groupKey`のルール違反。グループラベルのないデータソースに`groupKey`を送信した場合、必要な`groupKey`がない場合、ラベル名のセットがデータソースのグループラベルと異なる場合、ラベル名の重複、値のリストが空または数が異なる場合。`X-NC-APP-KEY`ヘッダが欠落 | 開始、停止、削除 |
| -4041301 | ServingPipeline not found. | `servingPipelineId`に該当するアプリが存在しないか、別のAppkeyのアプリです。 | 開始、停止、削除 |
| -4041101 | DataSource not found. | アプリに接続された指標データソースが見つかりません。 | 開始、停止、削除 |
| -4000201 | DataSource is not ready for ingest. | 指標データソースが使用可能な状態ではありません。 | 開始、停止、削除 |
| -4001302 | Serving pipeline is not active. | アプリがアクティブ状態ではありません。アプリ状態がアクティブになってから再試行します。 | 開始 |
| -4041306 | Group entry not found or has no trainingPipelineId. | リクエストしたグループの中に登録されていないグループがあります。リクエストに含まれる他のグループも処理されません。 | 停止、削除 |

<a id="recommendation-api"></a>
### 推薦照会 API { #recommendation-api }

作成した推薦システムアプリに推薦結果をリクエストします。ユーザーの履歴が十分な場合はモデルベース(Sequential)、不足している場合は属性ベース(Cold Start)で推論します。

<a id="recommendation-api-recommend"></a>
#### 推薦リクエスト { #recommendation-api-recommend }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/recommendation-apps/{appId}/recommend |

curl例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| userId | String | O | 推薦対象のユーザーID。匿名ユーザーへの推薦をリクエストする場合は空文字列("")を指定 |
| context.currentItemKey | String | X | 現在閲覧中のアイテムキー |
| context.recentlyViewed | Array | X | 最近閲覧したアイテムキーの一覧 |
| context.availableItems | Array | X | 推薦対象のアイテムキー一覧。指定した場合はこの一覧に含まれるアイテムの中からのみ推薦 |
| context.pageType | String | X | 現在のページタイプ(自由形式。例: home、item_detail) |
| context.sessionId | String | X | セッションID |
| context.impressions | Array | X | ユーザーへの推薦結果として表示されたアイテム一覧 |
| context.interactions | Array | X | ユーザーのアイテム行動情報 |
| context.feedback | Array | X | ユーザーがアイテムに残した評価 |
| userAttributes | Object | X | ユーザー属性情報(Cold Start推論に使用) |
| options.maxRecommendations | Integer | X | 最大推薦数(1～100)。100を超える値はエラーなく100に調整、未指定時は100を適用。推薦可能なアイテムがこの値より少ない場合は実際のアイテム数のみ返却 |
| options.mode | String | X | 推論方式の指定。sequential(履歴ベース)、cold_start(属性ベース)、popular(人気ベース)のいずれかで、それ以外の値は拒否。未指定時はサーバーが自動決定 |
| options.longtail | Boolean | X | 人気の低い項目まで含めて推薦の多様性を向上。sequentialのみ適用 |
| options.excludeItemKeys | Array | X | 推薦から除外するアイテムキー一覧。除外したアイテムは最大推薦数に含まれません |

- `options.mode`を指定しない場合、サーバーが推論方式を決定します。この場合、決定された方式のモデルがアプリに存在しない場合は、アプリに連携されている別の方式で代わりに推薦します。実際に使用した方式は、レスポンスの`body.metadata.inferenceType`で確認します。
- `options.mode`に許可されていない値を送信すると、結果コード`-4004201`で拒否されます。許可された値であっても、リクエストした方式のモデルがアプリになく、代替方式もない場合は結果コード`-5034201`を返します。アプリにどのモデルが作成されているか、および学習が完了しているかを確認してから再度呼び出してください。`options.mode`で方式を指定したリクエストは代替されないため、このレスポンスを受け取ることがあります。

<a id="recommendation-api-signal"></a>
##### 行動シグナル { #recommendation-api-signal }

`context.impressions`は、ユーザーに表示された推薦情報をもとに推薦結果を再ランキングするために使用されます。
`context.interactions`、`context.feedback`は、ユーザーが推薦結果に示した行動を伝えるフィールドで、ユーザー行動ベースのデータをモデル推論に反映します。

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| requestId | String | O | 該当の行動が発生した推薦レスポンスのbody.metadata.requestId |
| itemKeys | Array | O | 表示したアイテムキーの一覧。表示順に入力し、impressionsで使用 |
| itemKey | String | O | 対象アイテムキー。interactions、feedbackで使用 |
| type | String | O | interactionsはCLICK、CONVERSION。feedbackはPOSITIVE、NEGATIVE |
| occurredAt | String | O | 行動が発生した日時 |

- `occurredAt`はタイムゾーンオフセットを含むISO 8601形式で送信します。オフセットがない場合はエラーとして処理されます。
- `requestId`、`occurredAt`、アイテムキーの3フィールドすべてが揃っている場合のみシグナルとして使用されます。
- 各フィールドは古いものから新しい順に送信します。
- `impressions`は最大10件で、1件あたりの`itemKeys`は最大100個です。`interactions`と`feedback`は`type`別に最大10件です。上限を超えるとリクエストが拒否されます。
- 行動シグナルは今回の推薦リクエストの推論入力としてのみ使用し、保存はしません。同じアイテムの`feedback`が変わった場合は最新の値のみ反映されるため、効果を維持するには毎回のリクエストで再送信します。
- 反応イベントを保存して分析に活用するには、[推薦イベント API](#recommendation-event-api)を併せて使用します。

!!! tip "ポイント"
    `userAttributes`スキーマは、今後の選好度誘導(Preference Elicitation)の実装方針によって、収集方式やフィールドの種類が変更される場合があります。

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
| body.recommendations[].itemKey | 推薦アイテムキー |
| body.recommendations[].score | 推薦スコア(0.0～1.0) |
| body.recommendations[].position | 推薦順位 |
| body.metadata.modelVersion | 使用されたモデルバージョン |
| body.metadata.requestId | リクエスト追跡ID。推薦イベントAPI送信時はこの値を使用 |
| body.metadata.inferenceType | 推論タイプ。sequential(履歴ベース)、cold_start(属性ベース)、popular(人気ベース) |
| body.metadata.abTestGroup | A/Bテストグループ(現在は空値を返却) |

<a id="recommendation-error-codes"></a>
#### エラーコード { #recommendation-error-codes }

[共通エラーコード](#auth-common-error-codes)以外に、推薦リクエストで返される可能性のあるエラーコードです。

| コード | メッセージ | 説明 |
| --- | --- | --- |
| -4004201 | Invalid request. または拒否理由 | リクエスト形式エラー。`userId`の欠落、`maxRecommendations`が1未満、`options.mode`が許可されていない値、カテゴリ最小推薦数ルール違反、カタログにないカテゴリ、`context`の行動シグナルの形式・件数違反、`X-NC-APP-KEY`ヘッダの欠落など。`resultMessage`に拒否理由が含まれます。 |
| -4044201 | Recommendation app not found. | `appId`に対応する推薦アプリが存在しないか、`X-NC-APP-KEY`とアプリが一致しません。 |
| -4001301 | Invalid model type. | `appId`が推薦システムアプリではありません。 |
| -4001302 | Serving pipeline is not active. | アプリがアクティブ状態ではありません。アプリの状態がアクティブになってから再試行してください。 |
| -5034201 | Recommendation model is not ready. | リクエストした推薦モードのモデルがまだアプリに連携されていません。初回の学習とデプロイが完了してから再試行してください。 |

<a id="recommendation-event-api"></a>
### 推薦イベント API { #recommendation-event-api }

推薦結果に対するユーザーの反応(クリックなど)イベントを収集します。収集したイベントデータで推薦の成功率を分析できます。

<a id="recommendation-event-api-send"></a>
#### 推薦イベント転送 { #recommendation-event-api-send }

| メソッド | URI |
| --- | --- |
| POST | /api/v2.0/recommendation-apps/{appId}/events |

curlの例:

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

`requestId`、`itemKey`、`userId`は、推薦照会 APIのレスポンスで受け取った値をそのまま渡します。

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| eventType | O | イベントのタイプ。自由に定義可能(例: CLICK、PURCHASE、IMPRESSION)。英字・数字・アンダースコア(_)のみ使用可(^[A-Za-z0-9_]+$)、最大64文字。大文字・小文字を区別せず大文字に正規化して保存。REQUEST、RESPONSEは予約語のため使用不可 |
| requestId | O | 推薦 APIレスポンスのbody.metadata.requestId値(opaque string、最大128文字) |
| itemKey | O | ユーザーが反応した推薦アイテムのitemKey |
| userId | X | 推薦 APIレスポンスのbody.userId値 |
| context | X | イベントの付加情報(自由形式のキーと値。例: 表示位置 position、掲載面 placement) |
| userAttributes | X | ユーザー属性情報(自由形式のキーと値) |
| options | X | 付加オプション(自由形式のキーと値) |

成功レスポンス(200)は`header`のみ返します。

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  }
}
```

!!! tip "ポイント"
    - 成功レスポンス(200)は、収集パイプラインがイベントを受信したことを意味し、分析テーブルへの格納完了を保証するものではありません。
    - イベント APIリクエスト後、データセットへの格納まで最大10分かかる場合があります。
    - タイムアウト後に再試行すると、同じイベントが重複して格納される場合があります。分析時は重複排除を考慮してください。

<a id="recommendation-event-error-codes"></a>
#### エラーコード { #recommendation-event-error-codes }

[共通エラーコード](#auth-common-error-codes)以外に、推薦イベント転送で返される可能性があるエラーコードです。

| コード | メッセージ | 説明 |
| --- | --- | --- |
| -4004202 | Invalid event request. | リクエスト形式のエラー。`eventType`が空か64文字を超えるか英字・数字・`_`以外の文字を含むか予約語(REQUEST、RESPONSE)の場合、`requestId`が空か128文字を超える場合、`itemKey`の欠落、必須ヘッダの欠落 |
| -4044201 | Recommendation app not found. | `appId`に該当する推薦アプリが存在しないか、`X-NC-APP-KEY`とアプリが一致しません。 |
| -4001301 | Invalid model type. | `appId`が推薦システムアプリではありません。 |
| -4001302 | Serving pipeline is not active. | アプリがアクティブ状態ではありません。 |
| -4094201 | Recommendation event dataset is not configured for this app. | アプリに推薦イベントを保存するデータソースが設定されていません。 |
| -5034202 | Event publish failed. | イベントの保存に失敗しました。しばらくしてから再試行してください。 |
| -5044201 | Event publish timed out. | イベントの保存が制限時間内に完了しませんでした。しばらくしてから再試行してください。 |

<a id="api-v1-guide"></a>
## API v1.0ガイド { #api-v1-guide }

!!! danger "注意"
    API v1.0はサポート終了予定です。サポート終了のスケジュールは別途ご案内します。
    v2.0の使用を推奨します。詳細については、[API v2.0ガイド](#api-v2-guide)を参照してください。

<a id="auth-common-v1"></a>
### 認証および共通事項 { #auth-common-v1 }

<a id="auth-common-preparation-v1"></a>
#### 事前準備 { #auth-common-preparation-v1 }

APIを使用するには、Appkeyと認証トークンが必要です。

- AppkeyはNHN Cloudコンソールの**Machine Learning > NHN Cloud Foundry**ページ上部の**URL & Appkey**メニューで確認できます。
- APIは**gateway-public**エンドポイントを使用します。
- 認証トークン(`X-NHN-Authorization`ヘッダのBearerトークン)の発行方法は、[User Access Keyトークン](/nhncloud/ja/public-api/user-access-key-token/)ガイドを参照してください。

<a id="auth-common-request-v1"></a>
#### リクエスト共通事項 { #auth-common-request-v1 }

必須ヘッダ:

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
#### レスポンス共通事項 { #auth-common-response-v1 }

すべてのAPIレスポンスは`header`と`body`で構成されます。

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
| header.isSuccessful | Boolean | リクエストの成功可否 |
| header.resultCode | Integer | 結果コード。成功時は0、失敗時はエラーコード |
| header.resultMessage | String | 結果メッセージ。成功時はSUCCESS、失敗時はエラー詳細メッセージ |
| body | Object/Array | API別レスポンスデータ |

リクエストが拒否されてもHTTPステータスコードは`200`で返される場合があります。成功可否はHTTPステータスコードではなく、`header.isSuccessful`と`header.resultCode`で判定します。認証トークンがないか期限切れの場合は、HTTP `401`を返します。

<a id="ingest-api-v1"></a>
### Ingest API { #ingest-api-v1 }

Ingest APIは、コンソールで作成済みのデータソースにデータを格納するAPIです。データソースのタイプ別に次の方式を提供します。

| 方式 | 対象データソース | 説明 |
| --- | --- | --- |
| スナップショットアップロード | ファイル | アップロードしたファイルでデータをすべて置き換え |
| イベント収集 | ファイル | 既存データを維持したまま変更イベントを1件ずつ追加 |
| 指標収集 | Prometheus API | 指標(時系列)データをリアルタイムで転送 |

!!! danger "注意"
    データソースを新規に作成するAPIは提供していません。Ingest APIを使用するには、コンソールでデータソースを先に作成する必要があります。

<a id="ingest-snapshot-v1"></a>
#### スナップショットアップロード(ファイルアップロード) { #ingest-snapshot-v1 }

アップロードしたファイルの内容でデータソースのデータを**すべて置き換え**ます。アップロードは3段階で進めます。

!!! danger "注意"
    スナップショットアップロードはデータソースにすでに取り込まれたデータをすべて置き換えます。既存のデータは復旧できません。

アップロード制限：

- 最大アップロードサイズ：**10GB**
- `100MB`以下 → **シングルアップロード(SINGLE)**
- `100MB`超過 → **マルチパートアップロード(MULTIPART)**
- `formPost`フィールドの値は、レスポンスに含まれる値を**そのまま**リクエストに使用します。

<a id="ingest-snapshot-init-v1"></a>
##### 1. アップロード初期化(init) { #ingest-snapshot-init-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/init |

大容量ファイルをストレージに直接アップロードするための署名付き一時URLを発行します。ファイルサイズに応じてシングルURL(SINGLE)またはマルチパートURL(MULTIPART)を返します。

curlの例：

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
| fileName | String | O | ファイル名。使用可能文字：英字、数字、ドット(.)、アンダースコア(_)、ハイフン(-) |
| fileSize | Long | O | ファイルサイズ(bytes)。最小1、最大10GB |
| contentType | String | X | Content-Type(デフォルト：application/octet-stream) |

レスポンスの例(SINGLE)：

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

レスポンスの例(MULTIPART)：

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

!!! tip "ポイント"
    MULTIPARTレスポンスにも`formPost`が含まれます。ただし、マルチパートアップロードは`parts[].uploadUrl`のクエリパラメータ(`signature`/`expires`/`max_file_size`/`max_file_count`)でパートを転送するため、`formPost`は参考用であり、パートアップロード自体には使用しません。

| フィールド | 説明 |
| --- | --- |
| body.jobId | ジョブID。以降のcomplete/ステータス照会リクエストに使用 |
| body.uploadType | アップロードタイプ。SINGLE(100MB以下)またはMULTIPART(100MB超過) |
| body.uploadUrl | アップロードURL(シングルアップロード時) |
| body.uploadId | マルチパートアップロードID(マルチパートアップロード時) |
| body.partSize | パートサイズ(bytes、マルチパートアップロード時) |
| body.parts[].partNumber | パート番号(1から始まる) |
| body.parts[].uploadUrl | パートアップロードURL |
| body.parts[].headUrl | ETag照会用URL(アップロード完了後にHEADリクエスト) |
| body.expiresAt | URL有効期限 |
| body.formPost.objectPrefix | オブジェクトプレフィックス(ファイル名の前に付くパス) |
| body.formPost.signature | HMAC-SHA1署名 |
| body.formPost.expires | 有効期限(UNIXタイムスタンプ) |
| body.formPost.maxFileSize | 最大ファイルサイズ(bytes) |
| body.formPost.maxFileCount | 最大ファイル数 |

<a id="ingest-snapshot-upload-single-v1"></a>
##### 2-A. シングルファイルアップロード(100MB以下) { #ingest-snapshot-upload-single-v1 }

initレスポンスの`uploadUrl`にmultipart/form-data POSTを送信します。
このリクエストはObject Storageに直接送信するため、別途認証は不要です(`signature`が認証の役割を担います)。

curlの例：

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
    `file`フィールドは必ずフォームデータの最後に追加する必要があります。成功時にHTTP `201 Created`レスポンスを受け取ります。

<a id="ingest-snapshot-upload-multipart-v1"></a>
##### 2-B. 大容量ファイルアップロード(100MB超過、MULTIPART) { #ingest-snapshot-upload-multipart-v1 }

レスポンスの`parts[]`配列を受け取り、パートごとにアップロードします。
各パートは(1)アップロード → (2)HEADでETag照会 → (3)`partETags[]`に`partNumber`の昇順で収集する順序で処理します。

1. ファイルを`partSize`(デフォルト100MB)単位で分割します。
2. パートごとに`parts[i].uploadUrl`のクエリパラメータ(`signature`、`expires`、`max_file_size`、`max_file_count`)を解析してmultipart/form-dataで転送します(フィールド名`file`、ファイル名固定`part`)。
3. アップロード成功後、`parts[i].headUrl`に`HEAD`リクエストを送信し、レスポンスヘッダの`ETag`値を収集します。
4. すべてのパートが完了したら、`partETags`配列を`partNumber`の昇順で構成し、アップロード完了(complete)リクエストに含めて送信します。

パートアップロードのcurlの例：

```bash
# 1) アップロード
curl -X POST "{parts[i].uploadUrl}" \
  -F "redirect=" \
  -F "max_file_size={max_file_size-from-query}" \
  -F "max_file_count={max_file_count-from-query}" \
  -F "expires={expires-from-query}" \
  -F "signature={signature-from-query}" \
  -F "file=@./part_i.bin;filename=part"

# 2) ETag照会
curl -I "{parts[i].headUrl}" | grep -i '^etag:'
```

<a id="ingest-snapshot-complete-v1"></a>
##### 3. アップロード完了(complete) { #ingest-snapshot-complete-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/complete |

curlの例(シングルアップロード)：

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

curlの例(マルチパートアップロード)：

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
| jobId | String | O | ジョブID(initレスポンスのjobId) |
| fileName | String | O | ファイル名 |
| uploadId | String | X | マルチパートアップロードID(マルチパートアップロード時のみ必要) |
| partETags | Array | X | パートごとのETagリスト(マルチパートアップロード時のみ必要、partNumber順) |

レスポンスの例：

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
| body.jobId | ジョブID。[ジョブステータス照会](#ingest-snapshot-job-status-v1)に使用 |

<a id="ingest-snapshot-cancel-v1"></a>
##### アップロードキャンセル { #ingest-snapshot-cancel-v1 }

| メソッド | URI |
| --- | --- |
| DELETE | /api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId} |

curlの例(シングルアップロード)：

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

curlの例(マルチパートアップロード)：

クエリパラメータで`uploadId`を一緒に渡します。

```bash
curl -X DELETE "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/snapshots/{jobId}?uploadId={uploadId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

<a id="ingest-snapshot-job-status-v1"></a>
##### ジョブステータス照会 { #ingest-snapshot-job-status-v1 }

| メソッド | URI |
| --- | --- |
| GET | /api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId} |

curlの例：

```bash
curl "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/jobs/{jobId}" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

レスポンスの例：

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
| body.jobId | ジョブID |
| body.dataSourceId | 対象データソースID |
| body.jobType | ジョブタイプ。SNAPSHOT(スナップショット取り込み)またはEVENT(変更イベント) |
| body.status | ジョブステータス。下記のステータス値を参照 |
| body.obsFilePath | OBSファイルパス |
| body.statistics.totalRecords | レコードの総数 |
| body.statistics.failedRecords | 失敗レコード数 |
| body.statistics.successfulRecords | 成功レコード数 |
| body.statistics.successRate | 成功率(0.0～1.0) |
| body.errorMessage | エラーメッセージ(失敗時) |
| body.createdDatetime | ジョブ作成日時 |
| body.startedDatetime | ジョブ開始日時 |
| body.completedDatetime | ジョブ完了日時 |
| body.modifiedDatetime | 最終更新日時 |

ジョブステータス(`status`)は次の値を持ちます。

| 値 | 説明 |
| --- | --- |
| UPLOADING | ファイルアップロード中 |
| QUEUED | アップロード完了、取り込み待機中 |
| STAGED | 処理準備完了 |
| RUNNING | データ取り込み中 |
| COMPLETED | ジョブ正常完了 |
| FAILED | ジョブ失敗 |

<a id="event-ingest-api-v1"></a>
#### イベント収集 { #event-ingest-api-v1 }

既存のデータを保持したまま、変更イベントを転送します。タイプがファイルのデータソースで使用し、**Event API**を先に有効化する必要があります。有効化は、コンソールのイベント設定タブまたは以下の有効化APIで行います。

!!! danger "注意"
    Event APIを有効化すると、スナップショットアップロードがブロックされます。また、スキーマの変更が制限されるため、スキーマを変更するにはEvent APIを先に無効化する必要があります。有効化・無効化の方法は[コンソールユーザーガイド](./console-user-guide/#datasource-detail-event)の「イベント設定」を参照してください。

<a id="event-ingest-api-enable-v1"></a>
##### Event API有効化・無効化 { #event-ingest-api-enable-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/enable |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/disable |

curlの例:

```bash
curl -X POST "https://{gateway-public-host}/api/v1.0/data-sources/{dataSourceId}/ingest/events/enable" \
  -H "X-NC-APP-KEY: {appKey}" \
  -H "X-NHN-Authorization: Bearer {ACCESS_TOKEN}"
```

レスポンスの例:

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

| フィールド | 説明 |
| --- | --- |
| body.enabled | イベント収集の可否 |
| body.status | 有効化状態。DISABLED、ENABLING、ENABLED、ENABLE_FAILED |

- 有効化は非同期で行われます。リクエスト直後のレスポンスは`enabled`が`false`、`status`が`ENABLING`であり、`ENABLED`になってからイベントを収集します。
- すでにアクティブなデータソースに再度有効化をリクエストすると、状態をそのまま返します。非アクティブ状態に無効化をリクエストする場合も同様です。複数回呼び出しても結果は変わりません。
- ただし、有効化が進行中(`ENABLING`)のときに再度有効化をリクエストすると拒否されます。進行状況はコンソールのイベント設定タブで確認できます。
- 存在しないデータソースで呼び出すと、データソースが見つからないというエラーが返されます。

<a id="event-ingest-api-send-v1"></a>
##### イベント単件転送 { #event-ingest-api-send-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events |

curlの例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| operation | String | O | 操作タイプ。INSERT、UPDATE、DELETEのいずれか |
| data | Object | O | イベントデータ。データソーススキーマのフィールド名をキーとして使用 |
| eventTimestamp | String | X | イベント発生時刻。省略時はサーバー受信時刻を使用 |

- `operation`は大文字と小文字を区別しません。許可されない値を送信するとリクエストが拒否されます。
- データソースに主キーフィールドを指定していない場合、`INSERT`のみ送信できます。`UPDATE`と`DELETE`は拒否されます。主キーを指定したデータソースは3つの操作をすべて使用できます。

レスポンスの例:

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

| フィールド | 説明 |
| --- | --- |
| body.eventId | イベントID |
| body.success | 成功かどうか |
| body.errorMessage | 失敗時のエラーメッセージ |

<a id="event-ingest-api-batch-v1"></a>
##### イベント複数件転送 { #event-ingest-api-batch-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/events/batch |

複数件の変更イベントを一度に転送します。1回のリクエストにつき最大5,000件まで転送できます。

curlの例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| events | Array | O | イベント一覧。1回のリクエストにつき最大5,000件であり、各項目のフィールドは単件転送と同じです |

レスポンスの`body`はイベントごとの処理結果の配列です。

<a id="metrics-ingest-api-v1"></a>
#### 指標収集 { #metrics-ingest-api-v1 }

タイプがPrometheus APIのデータソースに指標データを転送します。転送した指標は分析メニューで照会でき、単変量時系列異常検出アプリの入力としても使用できます。

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/data-sources/{dataSourceId}/ingest/metrics |

curl例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| metrics | Array | O | 指標一覧。空にすることはできず、1回のリクエストあたり最大5,000件 |
| metrics[].timestamp | Long | O | 指標の時刻。ミリ秒epoch |
| metrics[].value | Double | O | 測定値 |
| metrics[].labels | Array | O | ラベル一覧。ラベルの組み合わせが時系列を、グループラベルがグループを決定します |
| metrics[].labels[].name | String | O | ラベル名。英字またはアンダースコア(_)で始まり、英字、数字、アンダースコア(_)のみ使用可能 |
| metrics[].labels[].value | String | O | ラベル値。カンマと等号は使用不可 |
| metrics[].metadata | Object | X | 付加情報。解釈せずそのまま保存・転送します。identityKeyキーはシステムが使用するため使用不可 |

成功するとHTTP `202 Accepted`を返します。リクエストが拒否された場合はHTTP `200`に`header.isSuccessful`が`false`で返されるため、`header`で成功かどうかを判定します。

収集規則は次のとおりです。

- 必須フィールドの欠落、ラベル名・値のルール違反、1回のリクエストで5,000件超過、必須ヘッダの欠落は、リクエスト全体が拒否され、どの項目も保存されません。
- 1回のリクエストに複数の時系列の指標をまとめることができます。時系列はラベルの組み合わせで区別されるため、時系列ごとにリクエストを分ける必要はありません。
- 同じ時系列は1分に1回だけ送信します。より短い間隔で収集する場合は、1分平均にまとめて送信します。同じ分に値が複数届いた場合は、最初に到着した値のみ分析に使用され、残りは破棄されます。
- `timestamp`はミリ秒単位のepochです。秒単位で送信すると、誤った時刻で保存されます。
- データソースにグループラベルを指定した場合は、常にそのラベルを含めて転送します。ラベルが欠けると意図したグループに属しません。
- `value`がNaNまたはInfinityの項目は保存せずスキップします。同じリクエストの残りの項目は正常に処理されます。
- `202`レスポンスは受信完了を意味します。保存は少し後に反映され、同じリクエストを再送すると同じデータが重複して保存される場合があります。
- 転送が遅延したデータは保存されますが、リアルタイム推論の対象から除外される場合があります。

!!! tip "ポイント"
    ロードは転送周期とは関係ありません。ただし、このデータソースを単変量時系列異常検出アプリに接続した場合は、同じ時系列を1分に1つずつ途切れなく送信する必要があります。アプリが指標を1分単位でまとめて判定するため、それより長い間隔で送信すると空白区間が生じ、正確モードで準備が完了しない場合があります。
    学習にも条件があります。データソースに時系列が1つだけの場合は学習が失敗するため、時系列を2つ以上設定する必要があります。また、時系列ごとに約4時間以上途切れなくデータが蓄積されることで、正常に学習できます。

<a id="univariate-api-v1"></a>
### 単変量時系列異常検出 API { #univariate-api-v1 }

<a id="univariate-group-api-v1"></a>
#### グループの使用開始・停止・削除 { #univariate-group-api-v1 }

単変量時系列異常検出アプリのグループの使用開始、停止、削除を行います。3つのAPIのリクエスト形式は同じで、パスのみ異なります。

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/enable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/disable |
| POST | /api/v1.0/serving-pipelines/{servingPipelineId}/groups/delete |

`servingPipelineId`はコンソールのアプリ詳細に表示されるアプリIDです。

curlの例:

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| groupKey | Array | 条件付き | 対象グループを指定するラベルのリスト。データソースにグループラベルを指定した場合のみ使用 |
| groupKey[].name | String | O | グループラベル名。データソースに指定したグループラベルと名前が完全に一致すること |
| groupKey[].value | Array | O | そのラベルの値リスト。値1つがグループ1つに対応 |

成功すると`header.isSuccessful`が`true`で返され、`body`はありません。

リクエストのルールは次のとおりです。

- データソースにグループラベルを指定していない場合は、`groupKey`を送信しません。データソース全体が1つのグループのため、そのグループが対象となります。`groupKey`を一緒に送信するとリクエストが拒否されます。
- データソースにグループラベルを指定した場合は`groupKey`は必須で、送信したラベル名のセットがデータソースのグループラベルと完全に一致する必要があります。同じラベル名を2回送信すると拒否されます。
- グループラベルが複数の場合、値リストを同じ順番でまとめてグループを作成します。例えば、`rule_id`に`["a", "b"]`、`instance_id`に`["q", "w"]`を送信すると、`(a, q)`と`(b, w)`の2つのグループが対象です。すべてのラベルの値の個数が同じである必要があり、異なる場合は拒否されます。
- 値リストが空の場合は拒否されます。同じグループが複数回指定された場合は、1回のみ処理されます。
- 登録されていないグループを停止または削除するとエラーが返されます。

!!! tip "ポイント"
    データソースにグループラベルを指定していない場合、アプリ作成完了時にデータソース全体がグループ1つとして登録されるため、このAPIを使用しなくても動作します。グループラベルを指定した場合、グループは自動的に登録されないため、使用開始APIで対象グループを登録する必要があります。この操作はAPIでのみ提供されます。登録されたグループと状態はコンソールのアプリ詳細の**グループ一覧**タブで確認します。

!!! danger "注意"
    グループを停止しても、検出結果転送は停止しません。グループ一覧に表示される状態のみ非アクティブに変わります。
    削除したグループは状態の記録とともに消え、復元することはできません。

<a id="recommendation-api-v1"></a>
### 推薦照会 API { #recommendation-api-v1 }

作成した推薦システムアプリに推薦結果をリクエストします。ユーザーの履歴が十分な場合はモデルベース(Sequential)、不足している場合は属性ベース(Cold Start)で推論します。

<a id="recommendation-api-recommend-v1"></a>
#### 推薦リクエスト { #recommendation-api-recommend-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/recommend |

curl例:

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
| userId | String | O | 推薦対象のユーザーID。匿名ユーザーへの推薦をリクエストするには、空文字列("")を指定します |
| context.currentItemKey | String | X | 現在閲覧中のアイテムキー |
| context.recentlyViewed | Array | X | 最近閲覧したアイテムキーの一覧 |
| context.availableItems | Array | X | 推薦対象のアイテムキーの一覧。指定すると、この一覧に含まれるアイテムの中からのみ推薦します |
| context.pageType | String | X | 現在のページタイプ(自由形式。例: home、item_detail) |
| context.sessionId | String | X | セッションID |
| context.impressions | Array | X | ユーザーに推薦結果として表示されたアイテムの一覧 |
| context.interactions | Array | X | ユーザーのアイテム行動情報 |
| context.feedback | Array | X | ユーザーがアイテムに対して残した評価 |
| userAttributes | Object | X | ユーザー属性情報(Cold Start推論に使用) |
| options.maxRecommendations | Integer | X | 最大推薦数(1～100)。100を超える値はエラーなく100に調整され、未指定の場合は100が適用されます。推薦可能なアイテムがこの値より少ない場合は、実際のアイテム数分のみ返します |
| options.mode | String | X | 推論方式を指定します。sequential(履歴ベース)、cold_start(属性ベース)、popular(人気ベース)のいずれか。未指定の場合、サーバーが自動的に決定します |
| options.longtail | Boolean | X | 人気の低い項目まで含めて推薦の多様性を向上させます。sequentialのときのみ適用されます |
| options.excludeItemKeys | Array | X | 推薦から除外するアイテムキーの一覧。除外したアイテムは最大推薦数に含まれません |

- `options.mode`を指定しない場合、サーバーが推論方式を決定します。このとき、決定された方式のモデルがアプリにない場合は、アプリに連携している別の方式で代わりに推薦します。実際に使用した方式は、レスポンスの`body.metadata.inferenceType`で確認します。
- リクエストした方式のモデルがアプリになく、代替方式もない場合、HTTP `503`と結果コード`5030001`を返します。アプリにどのモデルが作成されているか、および学習が完了しているかを確認してから再度呼び出してください。`options.mode`で方式を指定したリクエストは代替されないため、このレスポンスを受け取る可能性があります。

<a id="recommendation-api-signal-v1"></a>
##### 行動シグナル { #recommendation-api-signal-v1 }

`context.impressions`は、ユーザーに表示された推薦情報をもとに推薦結果を並び替えるために使用されます。
`context.interactions`、`context.feedback`は、ユーザーが推薦結果に示した行動を伝えるフィールドで、ユーザー行動ベースのデータをモデル推論に反映します。

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

| フィールド | タイプ | 必須 | 説明 |
| --- | --- | --- | --- |
| requestId | String | O | 該当する行動が発生した推薦レスポンスのbody.metadata.requestId |
| itemKeys | Array | O | 表示したアイテムキーの一覧。表示順で入力し、impressionsで使用します |
| itemKey | String | O | 対象のアイテムキー。interactions、feedbackで使用します |
| type | String | O | interactionsはCLICK、CONVERSION。feedbackはPOSITIVE、NEGATIVE |
| occurredAt | String | O | 行動が発生した日時 |

- `occurredAt`は、タイムゾーンオフセットを含むISO 8601形式で送信します。オフセットがない場合はエラーとして処理されます。
- `requestId`、`occurredAt`、アイテムキーの3つのフィールドがすべて存在する場合のみ、シグナルとして使用されます。
- 各フィールドは古いものから新しい順に送信します。
- `impressions`は最大10件で、1件あたりの`itemKeys`は最大100個です。`interactions`と`feedback`は`type`別に最大10件です。上限を超えるとリクエストが拒否されます。
- 行動シグナルは今回の推薦リクエストの推論入力としてのみ使用され、保存されません。同じアイテムの`feedback`が変わった場合は最新の値のみ反映されるため、効果を維持するには毎回のリクエストで再送信します。
- 反応イベントを保存して分析に活用するには、[推薦イベント API](#recommendation-event-api-v1)を併用します。

!!! tip "ポイント"
    `userAttributes`スキーマは、今後の選好度誘導(Preference Elicitation)の実装方針に応じて、収集方式やフィールドの種類が変更される場合があります。

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
| body.recommendations[].itemKey | 推薦アイテムキー |
| body.recommendations[].score | 推薦スコア(0.0～1.0) |
| body.recommendations[].position | 推薦順位 |
| body.metadata.modelVersion | 使用されたモデルバージョン |
| body.metadata.requestId | リクエスト追跡ID。推薦イベントAPI送信時にこの値を使用します |
| body.metadata.inferenceType | 推論タイプ。sequential(履歴ベース)、cold_start(属性ベース)、popular(人気ベース) |
| body.metadata.abTestGroup | A/Bテストグループ(現在は空の値を返します) |

<a id="recommendation-event-api-v1"></a>
### 推薦イベント API { #recommendation-event-api-v1 }

推薦結果に対してユーザーが示した反応(クリックなど)のイベントを収集します。蓄積されたイベントデータで推薦の成功率を分析できます。

<a id="recommendation-event-api-send-v1"></a>
#### 推薦イベント転送 { #recommendation-event-api-send-v1 }

| メソッド | URI |
| --- | --- |
| POST | /api/v1.0/recommendation-apps/{appId}/events |

curl例:

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

`requestId`、`itemKey`、`userId`は、推薦照会APIのレスポンスから受け取った値をそのまま送信します。

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| eventType | O | イベントタイプ。自由に定義可能(例: CLICK、PURCHASE、IMPRESSION)。英字・数字・アンダースコア(_)のみ使用可(^[A-Za-z0-9_]+$)、最大64文字。大文字・小文字を区別せず大文字に正規化して保存されます。REQUEST、RESPONSEは予約語のため使用不可 |
| requestId | O | 推薦APIレスポンスのbody.metadata.requestId値(opaque string、最大128文字) |
| itemKey | O | ユーザーが反応した推薦アイテムのitemKey |
| userId | X | 推薦APIレスポンスのbody.userId値 |
| context | X | イベントの付加情報(自由形式のキー・値。例: 表示位置 position、面 placement) |
| userAttributes | X | ユーザー属性情報(自由形式のキー・値) |
| options | X | 付加オプション(自由形式のキー・値) |

成功レスポンス(200)は`header`のみ返します。

```json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  }
}
```

!!! tip "ポイント"
    - 成功レスポンス(200)は、収集パイプラインがイベントを受信したことを意味し、分析テーブルへの積み込み完了を保証するものではありません。
    - イベントAPIリクエスト後、データセットへの積み込みまで最大10分かかる場合があります。
    - タイムアウト後に再試行すると、同じイベントが重複して積み込まれる可能性があります。分析時には重複除去を考慮してください。