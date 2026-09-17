<!-- machine_translated: true -->

<!-- pre-align:aligned sig=fef6db797893 -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > リリースノート { #foundry }

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
    - データソース: スキーマを定義してデータソースを作成し、ファイルアップロードまたは Ingest API (スナップショットアップロード) でデータを取り込んで、レコメンデーション・分析に使用できます。
    - 分析: 取り込んだデータをクエリで照会し、チャート・ダッシュボードで可視化して分析できます。
    - パイプライン: データソースのデータをフィルタリング・集計・結合などで加工して分析可能なデータセットに変換し、バッチスケジュールに基づく自動実行をサポートします。変換したデータセットは分析やレコメンデーションモデルの学習に使用できます。
    - アプリ: ユーザー・アイテム・インタラクションデータでレコメンデーションモデルを学習したレコメンデーションシステムアプリを作成し、レコメンデーション API でレコメンデーション結果をサービスに活用できます。