<!-- pre-align:aligned sig=11f4475601aa -->

<a id="foundry"></a>
## Machine Learning > NHN Cloud Foundry > 릴리스 노트 { #foundry }

<a id="foundry.release.notes.2026.09.18"></a>
### 2026. 09. 18. { #foundry.release.notes.2026.09.18 }

<a id="foundry.release.notes.2026.09.18.chart"></a>
#### 분석 / 차트 { #foundry.release.notes.2026.09.18.chart }

- 차트 설정이 잘못된 경우 화면에 사유가 표시되며, 한 차트의 조회 실패가 다른 차트에 영향을 주지 않습니다.

<a id="foundry.release.notes.2026.09.18.recommendation"></a>
#### 추천 앱 { #foundry.release.notes.2026.09.18.recommendation }

- 추천 API 요청에 노출(impressions), 상호작용(interactions), 피드백(feedback) 정보를 전달하면 추천 결과에 반영됩니다.

<a id="foundry.release.notes.2026.09.18.univariate"></a>
#### 단변량 시계열 이상탐지 앱 { #foundry.release.notes.2026.09.18.univariate }

- 단변량 시계열 이상탐지 앱이 추가되었습니다.

<a id="foundry.release.notes.2026.08.25"></a>
### 2026. 08. 25. { #foundry.release.notes.2026.08.25 }

<a id="foundry.release.notes.2026.08.25.new.service"></a>
#### 신규 서비스 출시 { #foundry.release.notes.2026.08.25.new.service }

- NHN Cloud Foundry가 출시되었습니다.
- 다음 기능을 사용할 수 있습니다.
    - 데이터 소스: 스키마를 정의해 데이터 소스를 생성하고, 파일 업로드 또는 Ingest API(스냅샷 업로드)로 데이터를 적재하여 추천·분석에 사용할 수 있습니다.
    - 분석: 적재한 데이터를 쿼리로 조회하고 차트·대시보드로 시각화하여 분석할 수 있습니다.
    - 파이프라인: 데이터 소스의 데이터를 필터링·집계·조인 등으로 가공해 분석 가능한 데이터셋으로 변환하며, 배치 스케줄에 따른 자동 실행을 지원합니다. 변환한 데이터셋은 분석이나 추천 모델 학습에 사용할 수 있습니다.
    - 앱: 사용자·아이템·상호작용 데이터로 추천 모델을 학습한 추천 시스템 앱을 생성하고, 추천 API로 추천 결과를 서비스에 활용할 수 있습니다.
