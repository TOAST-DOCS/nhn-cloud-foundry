<!-- machine_translated: true -->

<!-- pre-align:aligned sig=e1019368a5e9 -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > リリースノート { #foundry }

<a id="foundry-release-notes-2026-10-27"></a>
### 2026. 10. 27. { #foundry-release-notes-2026-10-27 }

<a id="foundry-release-notes-2026-10-27-query"></a>
#### 分析 / クエリ { #foundry-release-notes-2026-10-27-query }

- `UNION`などの集合演算とStarRocks専用の構文を含むクエリを実行できます。

<a id="foundry-release-notes-2026-10-27-app"></a>
#### アプリ { #foundry-release-notes-2026-10-27-app }

- アプリ一覧の**動作状態**カラムとアプリ詳細の**アプリ状態**タブで、アプリがデータを受け取り結果を出力しているかどうかを段階ごとに確認できます。

<a id="foundry-release-notes-2026-10-27-recommendation"></a>
#### 推薦アプリ { #foundry-release-notes-2026-10-27-recommendation }

- **サービング管理**タブで、モデルごとに学習済みモデルを選択してデプロイし、デプロイ履歴を確認できます。
- 進行中の学習を、アプリ全体またはモデルごとにキャンセルできます。
- 自動再学習を再開すると、停止中に実行されなかった回は即時実行されず、次のサイクルから実行されます。

<a id="foundry-release-notes-2026-10-27-api"></a>
#### API { #foundry-release-notes-2026-10-27-api }

API v2.0を提供します。リクエストパスのバージョンは`/api/v2.0`で、v1.0と比較してレスポンスのHTTPステータスコードとエラーコードが次のように変更されました。

- リクエストが失敗しても、すべてのAPIがHTTPステータスコード`200`でレスポンスします。成功かどうかは`header.isSuccessful`で判定します。
- エラーコード(`header.resultCode`)が負の数に変更されました。成功は`0`、エラーは負の数7桁で、`-`に続く3桁はHTTPステータスコードと同じ意味を持ちます。
- APIガイドに[共通エラーコード](./api-guide/#auth-common-error-codes)とAPI別エラーコード表が追加されました。
- API v1.0のサポート終了予定。

<a id="foundry-release-notes-2026-09-18"></a>
### 2026. 09. 18. { #foundry-release-notes-2026-09-18 }

<a id="foundry-release-notes-2026-09-18-chart"></a>
#### 分析 / チャート { #foundry-release-notes-2026-09-18-chart }

- チャートの設定が正しくない場合、画面に理由が表示されます。また、1つのチャートの照会失敗が他のチャートに影響を与えません。

<a id="foundry-release-notes-2026-09-18-recommendation"></a>
#### 推薦アプリ { #foundry-release-notes-2026-09-18-recommendation }

- 推薦 API リクエストに表示（impressions）、インタラクション（interactions）、フィードバック（feedback）情報を渡すと、推薦結果に反映されます。

<a id="foundry-release-notes-2026-09-18-univariate"></a>
#### 単変量時系列異常検出アプリ { #foundry-release-notes-2026-09-18-univariate }

- 単変量時系列異常検出アプリが追加されました。

<a id="foundry-release-notes-2026-08-25"></a>
### 2026. 08. 25. { #foundry-release-notes-2026-08-25 }

<a id="foundry-release-notes-2026-08-25-new-service"></a>
#### 新規サービスリリース { #foundry-release-notes-2026-08-25-new-service }

- NHN Cloud Foundry がリリースされました。
- 次の機能を使用できます。
    - データソース: スキーマを定義してデータソースを作成し、ファイルアップロードまたは Ingest API（スナップショットアップロード）でデータを取り込んで、推薦・分析に使用できます。
    - 分析: 取り込んだデータをクエリで照会し、チャート・ダッシュボードで可視化して分析できます。
    - パイプライン: データソースのデータをフィルタリング・集計・ジョインなどで加工し、分析可能なデータセットに変換します。バッチスケジュールに基づく自動実行をサポートします。変換したデータセットは分析や推薦モデルの学習に使用できます。
    - アプリ: ユーザー・アイテム・インタラクションデータで推薦モデルを学習した推薦システムアプリを作成し、推薦 API で推薦結果をサービスに活用できます。