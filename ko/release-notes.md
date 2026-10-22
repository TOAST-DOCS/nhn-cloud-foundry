<!-- pre-align:aligned sig=e1019368a5e9 -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > 릴리스 노트 { #foundry }

<a id="foundry-release-notes-2026-10-27"></a>
### 2026. 10. 27. { #foundry-release-notes-2026-10-27 }

<a id="foundry-release-notes-2026-10-27-query"></a>
#### 분석 / 쿼리 { #foundry-release-notes-2026-10-27-query }

- `UNION` 등 집합 연산과 StarRocks 전용 문법을 포함한 쿼리를 실행할 수 있습니다.

<a id="foundry-release-notes-2026-10-27-app"></a>
#### 앱 { #foundry-release-notes-2026-10-27-app }

- 앱 목록의 **동작 상태** 칼럼과 앱 상세의 **앱 상태** 탭에서 앱이 데이터를 받아 결과를 내고 있는지 단계별로 확인할 수 있습니다.

<a id="foundry-release-notes-2026-10-27-recommendation"></a>
#### 추천 앱 { #foundry-release-notes-2026-10-27-recommendation }

- **서빙 관리** 탭에서 모델별로 학습본을 선택해 배포하고 배포 이력을 확인할 수 있습니다.
- 진행 중인 학습을 앱 전체 또는 모델별로 취소할 수 있습니다.
- 자동 재학습을 재개하면 중지한 동안 놓친 회차를 바로 실행하지 않고 다음 주기부터 실행합니다.

<a id="foundry-release-notes-2026-10-27-api"></a>
#### API { #foundry-release-notes-2026-10-27-api }

API v2.0을 제공합니다. 요청 경로의 버전은 `/api/v2.0`이며, v1.0과 비교해 응답의 HTTP 상태 코드와 오류 코드가 다음과 같이 바뀌었습니다.

- 요청이 실패해도 모든 API가 HTTP 상태 코드 `200`으로 응답합니다. 성공 여부는 `header.isSuccessful`로 판정합니다.
- 오류 코드(`header.resultCode`)가 음수로 바뀌었습니다. 성공은 `0`, 오류는 음수 7자리이며 `-` 뒤 세 자리는 HTTP 상태 코드와 같은 의미입니다.
- API 가이드에 [공통 오류 코드](./api-guide/#auth-common-error-codes)와 API별 오류 코드 표가 추가되었습니다.
- API v1.0 지원 종료 예정.

<a id="foundry-release-notes-2026-09-18"></a>
### 2026. 09. 18. { #foundry-release-notes-2026-09-18 }

<a id="foundry-release-notes-2026-09-18-chart"></a>
#### 분석 / 차트 { #foundry-release-notes-2026-09-18-chart }

- 차트 설정이 잘못된 경우 화면에 사유가 표시되며, 한 차트의 조회 실패가 다른 차트에 영향을 주지 않습니다.

<a id="foundry-release-notes-2026-09-18-recommendation"></a>
#### 추천 앱 { #foundry-release-notes-2026-09-18-recommendation }

- 추천 API 요청에 노출(impressions), 상호작용(interactions), 피드백(feedback) 정보를 전달하면 추천 결과에 반영됩니다.

<a id="foundry-release-notes-2026-09-18-univariate"></a>
#### 단변량 시계열 이상탐지 앱 { #foundry-release-notes-2026-09-18-univariate }

- 단변량 시계열 이상탐지 앱이 추가되었습니다.

<a id="foundry-release-notes-2026-08-25"></a>
### 2026. 08. 25. { #foundry-release-notes-2026-08-25 }

<a id="foundry-release-notes-2026-08-25-new-service"></a>
#### 신규 서비스 출시 { #foundry-release-notes-2026-08-25-new-service }

- NHN Cloud Foundry가 출시되었습니다.
- 다음 기능을 사용할 수 있습니다.
    - 데이터 소스: 스키마를 정의해 데이터 소스를 생성하고, 파일 업로드 또는 Ingest API(스냅숏 업로드)로 데이터를 적재하여 추천·분석에 사용할 수 있습니다.
    - 분석: 적재한 데이터를 쿼리로 조회하고 차트·대시보드로 시각화하여 분석할 수 있습니다.
    - 파이프라인: 데이터 소스의 데이터를 필터링·집계·조인 등으로 가공해 분석 가능한 데이터셋으로 변환하며, 배치 스케줄에 따른 자동 실행을 지원합니다. 변환한 데이터셋은 분석이나 추천 모델 학습에 사용할 수 있습니다.
    - 앱: 사용자·아이템·상호작용 데이터로 추천 모델을 학습한 추천 시스템 앱을 생성하고, 추천 API로 추천 결과를 서비스에 활용할 수 있습니다.
