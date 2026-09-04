<!-- pre-align:aligned sig=8c004103ffd6 -->

<a id="foundry.getting.started"></a>
## Machine Learning > NHN Cloud Foundry > 시작하기 { #foundry.getting.started }

이 문서에서는 NHN Cloud Foundry에서 **추천 시스템 앱**을 생성하고 추천 결과를 활용하기까지의 과정을 설명합니다.
사전 준비(서비스 이용 신청, 데이터 준비)를 마친 뒤 다음 순서를 따릅니다.

1. 데이터 소스 생성하기
2. 앱 생성하기
3. 앱 상태 확인하기
4. 추천 결과 조회하기
5. 추천 이벤트 수집하기

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
8. 목록에서 상태가 COMPLETED가 될 때까지 기다립니다.

    ![데이터 소스 목록](../static/images/quick-start/데이터소스목록.png){ height="70%" }

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
