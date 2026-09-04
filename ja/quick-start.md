<!-- machine_translated: true -->

<!-- pre-align:aligned sig=8c004103ffd6 -->

<a id="foundry.getting.started"></a>
## Machine Learning > NHN Cloud Foundry > はじめに { #foundry.getting.started }

このドキュメントでは、NHN Cloud Foundry で**レコメンドシステムアプリ**を作成し、レコメンド結果を活用するまでの手順を説明します。
事前準備（サービス利用申請、データ準備）を完了した後、次の手順に従います。

1. データソースの作成
2. アプリの作成
3. アプリのステータス確認
4. レコメンド結果の照会
5. レコメンドイベントの収集

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