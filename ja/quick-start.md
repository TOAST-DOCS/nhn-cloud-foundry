<!-- machine_translated: true -->

<!-- pre-align:aligned sig=8c004103ffd6 -->

<a id="foundry.getting.started"></a>
## Machine Learning > NHN Cloud Foundry > はじめに { #foundry.getting.started }

この文書では、NHN Cloud Foundry でアプリを作成し、結果を活用するまでの流れを説明します。
事前準備（サービス利用申請、データの準備）を完了した後、作成するアプリのタイプに応じて以下の順序に従います。

**推薦システムアプリ**

1. データソースの作成
2. アプリの作成
3. アプリのステータス確認
4. レコメンド結果の照会
5. レコメンドイベントの収集

**単変量異常検知アプリ**

1. 指標データソースを作成する
2. 指標を転送する
3. アプリを作成する
4. 検出結果を確認する

<a id="preparation"></a>
## 事前準備 { #preparation }

<a id="preparation.service.enable"></a>
### サービス利用申請 { #preparation.service.enable }

NHN Cloud Foundry はコンソールから直接有効化することはできません。サービスを利用するには、[1:1 お問い合わせ](https://www.nhncloud.com/kr/support/inquiry)から申請する必要があります。

1. NHN Cloud コンソールで、サービスを利用する組織とプロジェクトを選択します。
2. **[Machine Learning > NHN Cloud Foundry > 現況]** タブで **[1:1 お問い合わせ]** ボタンをクリックし、希望するリソースサイズを含めて利用を申請します。
3. 担当者が該当プロジェクトにサービスを有効化すると、すべての機能を使用できます。

![サービス利用申請](../static/images/quick-start/서비스이용신청.png){ height="70%" }

<a id="preparation.data"></a>
### データの準備 { #preparation.data }

レコメンドシステムアプリを作成するには、次の 3 つの CSV データが必要です。

| データ | 必須カラム | 説明 |
| --- | --- | --- |
| ユーザーテーブル | ユーザー ID | ユーザー情報（追加の特性カラムは任意） |
| アイテムテーブル | アイテム ID | アイテム情報（追加の特性カラムは任意） |
| 履歴テーブル | ユーザー ID、アイテム ID、タイムスタンプ | ユーザーとアイテムの相互作用履歴（評価、カテゴリカラムは任意） |

単変量異常検出アプリは、CSV の代わりに収集 API で送信するメトリクス（時系列）データが必要です。「単変量異常検出アプリを作成する」を参照してください。

<a id="datasource.create"></a>
## 1. データソースの作成 { #datasource.create }

**[Machine Learning > NHN Cloud Foundry > データソース]** タブに移動します。
各設定項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#datasource.create)の「データソースの作成」を参照してください。

1. **[データソースの作成]** ボタンをクリックします。

    ![データソースの作成](../static/images/quick-start/데이터소스생성모달1.png){ height="70%" }

2. 基本設定にデータソース名とテーブル名を入力します。
3. 接続設定でデータソースのタイプが**ファイルのアップロード**であることを確認します。
4. 詳細設定で CSV ファイルを選択します。ファイルの先頭行がカラム名の場合は、**最初の行はヘッダーです**をチェックします。プライマリキーフィールド（例: `user_id`）を入力します。
5. **[タイプ推論]** ボタンをクリックすると、CSV サンプルからスキーマが自動的に入力されます。誤って推論されたタイプは直接修正します。

    ![データソースの作成 - ファイル選択とタイプ推論](../static/images/quick-start/데이터소스생성모달2.png){ height="70%" }

6. **[追加]** ボタンをクリックすると、データソースが作成されます。
7. 同じ方法で、**[ユーザー]**、**[アイテム]**、**[ヒストリー]** のデータソースをそれぞれ作成します。
8. 一覧でステータスが `COMPLETED` になるまで待ちます。

    ![データソース一覧](../static/images/quick-start/데이터소스목록.png){ height="70%" }

指標(時系列)データを受け取る Prometheus API タイプは、作成方法が異なります。「単変量異常検出アプリの作成」の「指標データソースの作成」を参照してください。

<a id="app.create"></a>
## 2. アプリの作成 { #app.create }

**[Machine Learning > NHN Cloud Foundry > アプリ]** タブに移動し、**[アプリの作成]** ボタンをクリックします。
各設定項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#app.create)の「アプリの作成」を参照してください。

<a id="app.create.basic"></a>
### 基本設定 { #app.create.basic }

アプリ名とアプリの説明を入力し、アプリタイプとして **[レコメンドシステム]** を選択してから、**[次へ]** をクリックします。

![アプリの作成 - 基本設定](../static/images/quick-start/앱생성화면1.png){ height="70%" }

<a id="app.create.detail"></a>
### 詳細設定 { #app.create.detail }

1. **[モデルの追加]** ボタンをクリックして、使用するモデルを追加します。新規サービスの場合は **[Cold User]**、ユーザーの行動履歴が十分にある場合は **[Warm User(Transformer)]** モデルをお勧めします。

    ![アプリの作成 - モデル設定](../static/images/quick-start/앱생성화면2.png){ height="70%" }

2. モデルカードの **[データ連携設定]** で、「1. データソースの作成」で作成したユーザー・アイテム・履歴のデータソースをそれぞれ選択します。
   ユーザー ID・アイテム ID のカラムと、履歴の時間カラムを指定します。Feature カラムは必要な場合にのみ選択します。

    ![アプリの作成 - データ連携設定](../static/images/quick-start/앱생성화면3.png){ height="70%" }

3. 必要に応じて **[追加設定(Skills)]** でスキルテーブルなどを連携します。基本モデル設定の Longtail モード（人気度の低いアイテムもレコメンドに含める）を指定します。設定が完了したら **[次へ]** をクリックします。

    ![アプリの作成 - 追加設定](../static/images/quick-start/앱생성화면4.png){ height="70%" }

<a id="app.create.review"></a>
### 最終確認 { #app.create.review }

1. 入力した基本設定、モデル設定、追加設定を確認します。
2. **[保存]** ボタンをクリックすると、アプリが作成されます。

![アプリの作成 - 最終確認](../static/images/quick-start/앱생성화면5.png){ height="70%" }

<a id="app.status"></a>
## 3. アプリのステータス確認 { #app.status }

アプリの作成後、学習とデプロイが自動的に進行します。ステータスは、初期化中、学習中、デプロイ中、有効化中を経て、アクティブに変わります。
アプリ一覧でステータスがアクティブになるまで待ちます。

![アプリ一覧](../static/images/quick-start/앱목록.png){ height="70%" }

ステータス値の詳細については、[コンソールユーザーガイド](../console-user-guide/#app.list.status)の「アプリのステータス」を参照してください。

!!! tip "ヒント"
    アプリ作成直後の学習・デプロイは、アプリを準備するプロセスです。レコメンドモデルの初回学習は、バッチスケジュール設定で指定した時刻に実行されます。それ以前にレコメンド API がレスポンスを返しても、学習済みモデルのレコメンド結果ではありません。

<a id="recommendation.query"></a>
## 4. レコメンド結果の照会 { #recommendation.query }

アプリがアクティブ状態になると、コンソールのレコメンド API 呼び出し画面でレコメンド結果を確認するか、レコメンド照会 API を呼び出してレコメンド結果を照会できます。
各項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#app.detail.recommend)の「レコメンド API 呼び出し」を参照してください。

1. アプリ一覧で作成したアプリをクリックし、詳細画面の **[レコメンド API 呼び出し]** タブに移動します。
2. ユーザー ID を入力し、レコメンドモードと最大レコメンド数を指定します。
3. **[レコメンドリクエスト]** ボタンをクリックすると、レコメンド結果に順位、アイテムキー、スコアが表示され、総結果数と応答時間も確認できます。

    ![レコメンド API 呼び出し](../static/images/quick-start/추천API호출.png){ height="70%" }

**[リクエストプレビュー]** には、入力値で構成された実際の API リクエスト JSON が表示されます。**[コピー]** ボタンでコピーして、API 連携開発に活用できます。
レコメンド照会 API を直接呼び出す方法については、[API ガイド](../api-guide/#recommendation.api)の「レコメンド照会 API」を参照してください。

レスポンスには、リクエスト識別子（`metadata.requestId`）とレコメンドアイテム一覧（`recommendations[].itemKey`）が含まれます。この値は、次のステップのレコメンドイベント送信に使用されます。

**[アプリ情報]** タブでは、API 呼び出しに使用するアプリ ID、ステータス、バージョンを確認できます。

![アプリ情報](../static/images/quick-start/앱정보.png){ height="70%" }

<a id="recommendation.event"></a>
## 5. レコメンドイベントの収集 { #recommendation.event }

ユーザーがレコメンド結果をクリックするなどの反応が発生した場合、レコメンドイベント API で送信します。蓄積されたイベントデータを使用して、レコメンドの成功率を分析できます。
リクエストフィールドの詳細については、[API ガイド](../api-guide/#recommendation.event.api)の「レコメンドイベント API」を参照してください。

```bash
curl -X POST '{URL}/api/v1.0/recommendation-apps/{APP_ID}/events' \
  -H "X-NC-APP-KEY: {APP_KEY}" \
  -H "Content-Type: application/json" \
  -H "X-NHN-Authorization: {AUTH_TOKEN}" \
  -d '{
    "eventType": "CLICK",
    "requestId": "{RecommendApiResponse.body.metadata.requestId}",
    "itemKey": "{RecommendApiResponse.body.recommendations.itemKey}",
    "userId": "{RecommendApiResponse.body.userId}",
    "context": {
      "position": 1,
      "placement": "home_main"
    }
  }'
```

!!! tip "ヒント"
    イベント API のリクエスト後、データセットへの反映まで最大 10 分かかる場合があります。

<a id="univariate"></a>
## 単変量異常検出アプリの作成 { #univariate }

メトリクスで正常範囲を外れた値を自動的に検出するには、単変量異常検出アプリを使用します。推薦システムとは独立したフローです。

<a id="univariate.datasource"></a>
### 1. 指標データソースの作成 { #univariate.datasource }

**Machine Learning > NHN Cloud Foundry > データソース** タブで **[データソース作成]** ボタンをクリックします。

1. 基本設定にデータソース名とテーブル名を入力します。
2. 接続設定でデータソースのタイプを **[Prometheus API]** に選択します。
3. 詳細設定でシリーズ識別ラベルとグループラベルを指定します。
    - スキーマは静的のため、直接入力しません。
4. **[追加]** ボタンをクリックし、一覧でステータスが `COMPLETED` になるまで待ちます。

    ![指標データソースの作成](../static/images/quick-start/지표데이터소스생성.png){ height="70%" }

各項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#datasource.create.detail.prometheus)の「Prometheus API 詳細設定」を参照してください。

<a id="univariate.ingest"></a>
### 2. 指標の転送 { #univariate.ingest }

作成したデータソースの詳細画面に移動し、**[収集方法]** タブを開きます。エンドポイント、リクエストヘッダ、リクエスト本文の例を **[コピー]** ボタンでコピーして指標を転送します。

![収集方法](../static/images/quick-start/수집방법.png){ height="70%" }

```bash
curl -X POST '{URL}/api/v1.0/data-sources/{DATA_SOURCE_ID}/ingest/metrics' \
  -H "X-NC-APP-KEY: {APP_KEY}" \
  -H "X-NHN-Authorization: {AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "metrics": [
      {
        "timestamp": 1776149886528,
        "value": 4.99,
        "labels": [
          { "name": "__name__", "value": "cpu_usage" },
          { "name": "instance_id", "value": "instance-001" }
        ]
      }
    ]
  }'
```

リクエスト形式の詳細については、[APIガイド](../api-guide/#metrics.ingest.api)の「指標収集」を参照してください。

!!! tip "知っておくべきこと"
    アプリを作成した後は、同じ時系列の指標を1分間隔以下で途切れなく送信します。それより長い間隔で送信すると空白区間が生じ、精密モードで準備が完了しない場合があります。

<a id="univariate.app"></a>
### 3. アプリの作成 { #univariate.app }

**Machine Learning > NHN Cloud Foundry > アプリ** タブで **[アプリ作成]** ボタンを押します。

1. 基本設定にアプリ名と説明を入力し、アプリタイプとして **[単変量異常検知]** を選択します。

    ![アプリ作成 - 基本設定](../static/images/quick-start/이상탐지앱생성1.png){ height="70%" }

2. 詳細設定で、先ほど作成した指標データソースを選択します。
    - モデルリソースと再学習周期、検知オプション、結果転送を指定します。

    ![アプリ作成 - 詳細設定](../static/images/quick-start/이상탐지앱생성2.png){ height="70%" }

3. 最終確認で入力内容を確認し、**[保存]** ボタンを押します。

各項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#app.create.detail.univariate)の「単変量異常検知 詳細設定」を参照してください。

!!! tip "ヒント"
    指標データソース1つに対して、単変量異常検知アプリは1つのみ作成できます。結果転送の転送モードはデフォルトの正確モードを推奨します。準備が完了する前の値をすぐに受け取りたい場合は、即時モードを選択します。

<a id="univariate.result"></a>
### 4. 탐지 결과 확인하기 { #univariate.result }

アプリ一覧から作成したアプリをクリックして、詳細画面に移動します。

1. **[アプリ情報]** タブで学習状態とグループの状況を確認します。

    ![単変量異常検知アプリ情報](../static/images/quick-start/이상탐지앱정보.png){ height="70%" }

2. **[グループ一覧]** タブでグループの状態を確認します。
    - グループはメトリクスが届いた後に登録されるため、アプリを作成した直後は一覧が空になっています。
    - 有効化待機中はは判定に使用するデータを収集中であり、有効化されると検知結果が送信されます。

    ![グループ一覧](../static/images/quick-start/이상탐지그룹목록.png){ height="70%" }

3. 検知結果である異常スコアとしきい値は、指定した Prometheus に送信され、結果データソースにも保存されます。
4. 保存された結果は、**[分析]** タブのクエリやチャートで照会します。

各項目の詳細については、[コンソールユーザーガイド](../console-user-guide/#app.detail.univariate)の「単変量異常検知アプリの詳細」を参照してください。