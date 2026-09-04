<!-- pre-align:aligned sig=8c004103ffd6 -->

<a id="foundry.getting.started"></a>
## Machine Learning > NHN Cloud Foundry > 시작하기 { #foundry.getting.started }

이 문서에서는 NHN Cloud Foundry에서 앱을 생성하고 결과를 활용하기까지의 과정을 설명합니다.
사전 준비(서비스 이용 신청, 데이터 준비)를 마친 뒤 만들려는 앱 유형에 따라 다음 순서를 따릅니다.

**추천 시스템 앱**

1. 데이터 소스 생성하기
2. 앱 생성하기
3. 앱 상태 확인하기
4. 추천 결과 조회하기
5. 추천 이벤트 수집하기

**단변량 이상 탐지 앱**

1. 지표 데이터 소스 생성하기
2. 지표 전송하기
3. 앱 생성하기
4. 탐지 결과 확인하기

<a id="preparation"></a>
## 사전 준비하기 { #preparation }

<a id="preparation.service.enable"></a>
### 서비스 이용 신청 { #preparation.service.enable }

NHN Cloud Foundry는 콘솔에서 직접 활성화할 수 없습니다. 서비스를 이용하려면 [1:1 문의](https://www.nhncloud.com/kr/support/inquiry)로 신청해야 합니다.

1. NHN Cloud 콘솔에서 서비스를 이용할 조직과 프로젝트를 선택합니다.
2. **Machine Learning > NHN Cloud Foundry > 현황** 탭에서 **1:1 문의** 버튼을 클릭하고, 원하는 리소스 크기를 포함하여 이용을 신청합니다.
3. 담당자가 해당 프로젝트에 서비스를 활성화하면 모든 기능을 사용할 수 있습니다.

![서비스 이용 신청](../static/images/quick-start/서비스이용신청.png){ height="70%" }

<a id="preparation.data"></a>
### 데이터 준비 { #preparation.data }

추천 시스템 앱을 만들려면 다음 3개의 CSV 데이터가 필요합니다.

| 데이터 | 필수 칼럼 | 설명 |
| --- | --- | --- |
| 사용자 테이블 | 사용자 ID | 사용자 정보(추가 특성 칼럼 선택 가능) |
| 아이템 테이블 | 아이템 ID | 아이템 정보(추가 특성 칼럼 선택 가능) |
| 히스토리 테이블 | 사용자 ID, 아이템 ID, 타임스탬프 | 사용자-아이템 상호작용 이력(평점, 카테고리 칼럼 선택 가능) |

단변량 이상 탐지 앱은 CSV 대신 수집 API로 보낼 지표(시계열) 데이터가 필요합니다. '단변량 이상 탐지 앱 만들기'를 참고합니다.

<a id="datasource.create"></a>
## 1. 데이터 소스 생성하기 { #datasource.create }

**Machine Learning > NHN Cloud Foundry > 데이터 소스** 탭으로 이동합니다.
각 설정 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#datasource.create)의 '데이터 소스 생성'을 참고합니다.

1. **데이터 소스 생성** 버튼을 클릭합니다.

    ![데이터 소스 생성](../static/images/quick-start/데이터소스생성모달1.png){ height="70%" }

2. 기본 설정에 데이터 소스 이름과 테이블 이름을 입력합니다.
3. 연결 설정에서 데이터 소스 유형이 **파일 업로드**인지 확인합니다.
4. 상세 설정에서 CSV 파일을 선택합니다. 파일 첫 행이 칼럼 이름이면 **첫 번째 행이 헤더입니다**를 체크합니다. 기본 키 필드(예: `user_id`)를 입력합니다.
5. **타입 추론** 버튼을 클릭하면 CSV 샘플로 스키마가 자동으로 채워집니다. 잘못 추론된 타입은 직접 수정합니다.

    ![데이터 소스 생성 - 파일 선택과 타입 추론](../static/images/quick-start/데이터소스생성모달2.png){ height="70%" }

6. **추가** 버튼을 클릭하면 데이터 소스가 생성됩니다.
7. 같은 방법으로 **사용자, 아이템, 히스토리** 데이터 소스를 각각 생성합니다.
8. 목록에서 상태가 `COMPLETED`가 될 때까지 기다립니다.

    ![데이터 소스 목록](../static/images/quick-start/데이터소스목록.png){ height="70%" }

지표(시계열) 데이터를 받는 Prometheus API 유형은 만드는 방법이 다릅니다. '단변량 이상 탐지 앱 만들기'의 '지표 데이터 소스 생성하기'를 참고합니다.

<a id="app.create"></a>
## 2. 앱 생성하기 { #app.create }

**Machine Learning > NHN Cloud Foundry > 앱** 탭으로 이동한 뒤 **앱 생성** 버튼을 클릭합니다.
각 설정 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#app.create)의 '앱 생성'을 참고합니다.

<a id="app.create.basic"></a>
### 기본 설정 { #app.create.basic }

앱 이름과 앱 설명을 입력하고 앱 유형으로 **추천 시스템**을 선택한 뒤 **다음**을 클릭합니다.

![앱 생성 - 기본 설정](../static/images/quick-start/앱생성화면1.png){ height="70%" }

<a id="app.create.detail"></a>
### 상세 설정 { #app.create.detail }

1. **모델 추가** 버튼을 클릭해 사용할 모델을 추가합니다. 신규 서비스라면 **Cold User**, 사용자 행동 이력이 충분하다면 **Warm User(Transformer)** 모델을 권장합니다.

    ![앱 생성 - 모델 설정](../static/images/quick-start/앱생성화면2.png){ height="70%" }

2. 모델 카드의 **데이터 연결 설정**에서 '1. 데이터 소스 생성하기'에서 만든 사용자·아이템·히스토리 데이터 소스를 각각 선택합니다.
   사용자 ID·아이템 ID 칼럼과 히스토리의 시간 칼럼을 지정합니다. Feature 칼럼은 필요할 때만 선택합니다.

    ![앱 생성 - 데이터 연결 설정](../static/images/quick-start/앱생성화면3.png){ height="70%" }

3. 필요 시 **추가 설정(Skills)**에서 스킬 테이블 등을 연결합니다. 기본 모델 설정의 Longtail 모드(인기도가 낮은 아이템도 추천에 포함)를 지정합니다. 설정을 마치면 **다음**을 클릭합니다.

    ![앱 생성 - 추가 설정](../static/images/quick-start/앱생성화면4.png){ height="70%" }

<a id="app.create.review"></a>
### 최종 검토 { #app.create.review }

1. 입력한 기본 설정, 모델 설정, 추가 설정을 검토합니다.
2. **저장** 버튼을 클릭하면 앱이 생성됩니다.

![앱 생성 - 최종 검토](../static/images/quick-start/앱생성화면5.png){ height="70%" }

<a id="app.status"></a>
## 3. 앱 상태 확인하기 { #app.status }

앱 생성 후 학습과 배포가 자동으로 진행됩니다. 상태는 초기화 중, 학습 중, 배포 중, 활성화 중을 거쳐 활성으로 바뀝니다.
앱 목록에서 상태가 활성이 될 때까지 기다립니다.

![앱 목록](../static/images/quick-start/앱목록.png){ height="70%" }

상태 값의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#app.list.status)의 '앱 상태'를 참고합니다.

!!! tip "알아두기"
    앱 생성 직후의 학습·배포는 앱을 준비하는 과정입니다. 추천 모델의 첫 학습은 배치 스케줄 설정에 지정한 시각에 실행되며, 그 전에는 추천 API가 응답을 반환하더라도 학습된 모델의 추천 결과가 아닙니다.

<a id="recommendation.query"></a>
## 4. 추천 결과 조회하기 { #recommendation.query }

앱이 활성 상태가 되면 콘솔의 추천 API 호출 화면에서 추천 결과를 확인하거나, 추천 조회 API를 호출하여 추천 결과를 조회할 수 있습니다.
각 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#app.detail.recommend)의 '추천 API 호출'을 참고합니다.

1. 앱 목록에서 생성한 앱을 클릭해 상세 화면의 **추천 API 호출** 탭으로 이동합니다.
2. 사용자 ID를 입력하고 추천 모드와 최대 추천 수를 지정합니다.
3. **추천 요청** 버튼을 클릭하면 추천 결과에 순위, 아이템 키, 점수가 표시되며, 총 결과 수와 응답 시간도 함께 확인할 수 있습니다.

    ![추천 API 호출](../static/images/quick-start/추천API호출.png){ height="70%" }

**요청 미리보기**에는 입력 값으로 구성된 실제 API 요청 JSON이 표시됩니다. **복사** 버튼으로 복사해 API 연동 개발에 활용할 수 있습니다.
추천 조회 API를 직접 호출하는 방법은 [API 가이드](../api-guide/#recommendation.api)의 '추천 조회 API'를 참고합니다.

응답에는 요청 식별자(`metadata.requestId`)와 추천 아이템 목록(`recommendations[].itemKey`)이 포함됩니다. 이 값은 다음 단계의 추천 이벤트 전송에 사용됩니다.

**앱 정보** 탭에서는 API 호출에 사용하는 앱 ID와 상태, 버전을 확인할 수 있습니다.

![앱 정보](../static/images/quick-start/앱정보.png){ height="70%" }

<a id="recommendation.event"></a>
## 5. 추천 이벤트 수집하기 { #recommendation.event }

사용자가 추천 결과를 클릭하는 등 반응이 발생하면 추천 이벤트 API로 전송합니다. 적재된 이벤트 데이터로 추천 성공률을 분석할 수 있습니다.
요청 필드의 자세한 설명은 [API 가이드](../api-guide/#recommendation.event.api)의 '추천 이벤트 API'를 참고합니다.

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

!!! tip "알아두기"
    이벤트 API 요청 후 데이터셋에 적재까지 최대 10분이 걸릴 수 있습니다.

<a id="univariate"></a>
## 단변량 이상 탐지 앱 만들기 { #univariate }

지표에서 정상 범위를 벗어난 값을 자동으로 찾으려면 단변량 이상 탐지 앱을 사용합니다. 추천 시스템과는 별개의 흐름입니다.

<a id="univariate.datasource"></a>
### 1. 지표 데이터 소스 생성하기 { #univariate.datasource }

**Machine Learning > NHN Cloud Foundry > 데이터 소스** 탭에서 **데이터 소스 생성** 버튼을 클릭합니다.

1. 기본 설정에 데이터 소스 이름과 테이블 이름을 입력합니다.
2. 연결 설정에서 데이터 소스 유형을 **Prometheus API**로 선택합니다.
3. 스키마는 고정이므로 직접 입력하지 않습니다. 상세 설정에서 시리즈 식별 라벨과 그룹 라벨을 지정합니다.
4. **추가** 버튼을 클릭하고, 목록에서 상태가 `COMPLETED`가 될 때까지 기다립니다.

    ![지표 데이터 소스 생성](../static/images/quick-start/지표데이터소스생성.png){ height="70%" }

각 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#datasource.create.detail.prometheus)의 'Prometheus API 상세 설정'을 참고합니다.

<a id="univariate.ingest"></a>
### 2. 지표 전송하기 { #univariate.ingest }

생성한 데이터 소스의 자세히 보기로 이동해 **수집 방법** 탭을 엽니다. 엔드포인트와 요청 헤더, 요청 본문 예시를 **복사** 버튼으로 복사해 지표를 전송합니다.

![수집 방법](../static/images/quick-start/수집방법.png){ height="70%" }

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

요청 형식의 자세한 설명은 [API 가이드](../api-guide/#metrics.ingest.api)의 '지표 수집'을 참고합니다.

!!! tip "알아두기"
    앱을 만든 뒤에는 같은 시계열의 지표를 1분 간격 이하로 끊김 없이 보냅니다. 그보다 긴 간격으로 보내면 빈 구간이 생겨 정확 모드에서 준비가 끝나지 않을 수 있습니다.

<a id="univariate.app"></a>
### 3. 앱 생성하기 { #univariate.app }

**Machine Learning > NHN Cloud Foundry > 앱** 탭에서 **앱 생성** 버튼을 클릭합니다.

1. 기본 설정에 앱 이름과 설명을 입력하고, 앱 유형으로 **단변량 이상 탐지**를 선택합니다.
2. 상세 설정에서 앞서 만든 지표 데이터 소스를 선택합니다. 모델 자원과 재학습 주기, 탐지 옵션, 결과 전송을 지정합니다.
    ![앱 생성 - 기본 설정](../static/images/quick-start/이상탐지앱생성1.png){ height="70%" }

3. 최종 검토에서 입력 내용을 확인하고 **저장** 버튼을 클릭합니다.

    ![앱 생성 - 상세 설정](../static/images/quick-start/이상탐지앱생성2.png){ height="70%" }

각 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#app.create.detail.univariate)의 '단변량 이상 탐지 상세 설정'을 참고합니다.

!!! tip "알아두기"
    지표 데이터 소스 하나에는 단변량 이상 탐지 앱을 하나만 만들 수 있습니다. 결과 전송의 전송 모드는 기본값인 정확 모드를 권장하며, 준비가 끝나기 전 값이라도 바로 받아 보려면 즉시 모드를 선택합니다.

<a id="univariate.result"></a>
### 4. 탐지 결과 확인하기 { #univariate.result }

앱 목록에서 생성한 앱을 클릭해 상세 화면으로 이동합니다.

1. **앱 정보** 탭에서 학습 상태와 그룹 현황을 확인합니다.

    ![단변량 이상 탐지 앱 정보](../static/images/quick-start/이상탐지앱정보.png){ height="70%" }

2. **그룹 목록** 탭에서 그룹 상태를 확인합니다. 활성화 대기 중은 판정에 사용할 데이터를 모으는 중이고, 활성화가 되면 탐지 결과가 전송됩니다.
3. 탐지 결과인 이상 점수와 임계값은 지정한 Prometheus로 전송되며, 결과 데이터 소스에도 저장됩니다. 저장된 결과는 **분석** 탭의 쿼리나 차트로 조회합니다.

각 항목의 자세한 설명은 [콘솔 유저 가이드](../console-user-guide/#app.detail.univariate)의 '단변량 이상 탐지 앱 상세'를 참고합니다.
