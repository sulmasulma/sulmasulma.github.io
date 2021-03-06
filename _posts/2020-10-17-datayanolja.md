---
layout: post
title: 데이터야놀자 2020
tags: [seminar]
categories: [Etc]
excerpt_separator: <!--more-->
---
2020 [데이터야놀자](https://datayanolja.github.io)에서 들은 내용을 적어 보았습니다.<!--more-->

![schedule](https://eventusstorage.blob.core.windows.net/evs/Image/datayanolja2020/23268/ProjectInfo/d786f217133f4c998230995b2688563a.png)

---

### 1. 당근마켓이 데이터와 함께 노는 법

#### 모델 학습 및 서빙

- 머신러닝 모델을 통한 글 분류
- 게시글의 텍스트를 활용하여 필터링
- 동네생활: 4만 개, 중고거래: 40만 개 정도의 글로 학습
- 사람같은 경우 텍스트를 이해하고 글 판단. 머신러닝 모델은 숫자를 입력으로 받기 때문에, 텍스트를 tokenize하여 숫자로 바꿔야 함
- 구글에서 개발한 `sentencepiece` 라이브러리 활용: 속도가 월등함
- NLP에서 baseline이 된 `BERT` 사용
  - 적은 데이터로 높은 성능 낼 수 있음
  - 라벨링되지 않은 데이터로 사전 학습?
  - `transformers` 라이브러리 사용
- 데이터의 불균형 문제
  - 필터링되지 않아도 되는 정상적인 게시글이 대부분 차지
  - 클래스별 불균형이 큼
- 해결책
  - loss function 변형 (focal loss-비전 분야에서 많이 사용-등)
  - oversampling: focal loss 사용시보다 높은 성능
- F1 score 0.84
  - 카테고리별 F1 score의 평균 수치
  - 특정 카테고리 비율이 낮기 때문. 후처리하면 더 성능 높게 나옴
- model server
  - `flask`도 가능
  - 머신러닝 모델에 특정하게 필요한 기능이 필요
  - pytorch의 `torch-serve` 사용: 여러 API 제공, 모델 버전 관리 등 운영에 편리한 요소 있음
  - flask보다 더 좋은 성능 + 낮은 cpu, memory 사용
- 이런 식으로 모델 학습 + 서빙하고 있음

#### 데이터 인프라

- 1000만 MAU -> 빠르고 안전하게 데이터를 저장하고, 쉽고 빠르게 꺼낼 수 있어야 함
  - 데이터 민주화(Democratization): 접근성 높임
  - 빠르게 꺼낼 수 있게
  - 안전하게: 사용자별 접근 권한 제어, 유실되지 않고 안전하게 저장
- **ELT**, not ETL
  - 클라우드로 인해 데이터의 저장/처리 비용이 싸고 빨라져서 가능해짐
  - 문제: 저장하기 전에 변환해야 함. 근데 이러면 저장 형태를 정하는 시간이 오래 걸림. 결과적으로 저장하는 데 시간이 오래 걸림
  - 그래서 저장해 놓고, 꺼내 쓸 때 처리
  - 예쁘게 쌓는 것보다, 일단 쌓고 활용하는 것이 중요
- Lambda Architecture
  - 2트랙
    - **배치** 프로세스를 이용하여 작은 단위로 데이터를 쪼개서 주기적으로 처리
    - **스트리밍** 데이터로 실시간 처리
  - 과거의 히스토리컬 데이터부터 실시간 데이터까지 처리할 수 있음
- `BigQuery`
  - 메인 DW (DL에 가깝긴 함)
  - **편리한 웹 UI** -> 일반 사용자들의 접근성 높아짐
  - 저렴한 비용과 빠른 조회. 처리 시간이 아닌 **스캔한 데이터의 양** 으로만 과금
    - 쿼리를 정교하게 짜지 않아도, 결과 데이터의 양만 신경 쓰면 됨
  - 보안 기능도 제공
  - BigQuery omni: 빅쿼리 인프라를 AWS 같은 다른 클라우드에서 사용할 수 있는 것?
- 고급 분석
  - `S3`, `Athena`, `Spark`, `Zeppelin/Jupyter` 등 사용
- 셀프 BI 시스템
  - Firebase를 사용하고 있었는데, 사용자와 데이터가 많아지다 보니 BigQuery의 문제 발생
  - 범용성으로 인해 데이터 저장 시점이 늦어지고, 데이터 포맷 지정할 수 없는 등의 문제
  - 그래서 프로젝트 진행 중
- 파이프라인 설계 과정에서 티어(단계)별 데이터 정리
  - 다음 단계에서는 전 단계에서 나온 데이터를 모두 사용하지는 않음
  - 필요에 따라 가공된 테이블을 미리 생성해 둠
- 기술 발전 속도가 성장 속도를 따라가지 못하고 있는 상태. 그래서 지속적으로 발전+채용 중

<br>

---

### 2. 오픈소스 기여: 코드로 세상에 기여하는 또다른 방법

- 존경하는 개발자들이 만든 거대한 오픈소스 프로젝트(e.g. `tensorflow`)에 관심을 가지게 됨
- 첫 오픈소스 경험. 실패담
  - 무작정 오픈소스 프로젝트를 찾아서 코딩했었음. 코드를 수정하고 압축해서 메일로 보냄. 그리고 무시당함
  - `git`을 통한 버전관리도 모를 시절
- 개인 프로젝트의 한계 체감
  - 실력 향상에 대한 의구심이 들어, 고수들의 코드를 보고 싶었음
- 오픈소스 경진대회 참가 (공개 SW 컨트리뷰톤)
  - 굵직한 프로젝트, 코드 난이도가 높은 편
  - 통째로 취약점 진단 툴로 돌린다면? -> 효과 없었음
  - 유저의 입장으로서 프로젝트 분석 시작. 영문 문서를 꼼꼼히 읽고, 모든 기능과 원리 파악
  - use case로서의 컨트리뷰션
- 문서화 작업을 하며, maintainer의 꼼꼼한 검토 받음
  - 문서화를 위해서는 꼼꼼한 user로서의 경험이 필요
  - 이를 통해 code commit 외의 기여(문서 기여 + user case 기여)
- contribute 할 만한 프로젝트
  - 최근까지 활발하게 논의되고 있는 프로젝트. 즉 issue나 maintainer의 comment가 최근일 경우
- git의 용도
  - 개인 프로젝트에서는 백업 용도이지만, 오픈소스 프로젝트에서는 다름
- 오픈소스에 code 기여하려면, 프로젝트를 모두 알아야 할까?
  - 쉬운 contibution은 issue에서 실마리를 찾을 수 있음. **issue를 해결할 수 있을 정도면 됨**
  - `GitHub`에 있는 issue들 중 해결 가능한 것 찾기. issue도 분류가 되어 있음 (버그, 기능 향상, Good First Issue(쉬운 이슈) 등)
  - 실력이 압도적으로 향상된 이후에 기여하기보다는, 작은 이슈부터 해결하면서 성장하는 것이 바람직
- 오픈소스 프로젝트 기여의 이점
  - 채용시 우대
  - **높은 수준의 개발자들과 논의하면서 협업 경험**
  - **정교한 코드** 를 작성하는 경험
  - SW Tester로서의 안목
  - 영어 실력

<br>

---

### 3. Kubernetes와 함께 데이터 흐름을 더 나이스하게 바꾸기

- 데브시스터즈 플랫폼셀의 데이터 엔지니어
- 데이터 플랫폼에서 `Kubernetes`의 이점
- 데이터 플랫폼
  - 사내 데이터 수집, 적재, 가공하여 서비스
  - 쿠키런에서 발생하는 데이터를 `Kafka`를 통해 수집, `ELK stack`을 통해 실시간 데이터 시각화
  - 데이터는 parquet 형태로 `S3`(cloud object storage)에 저장
  - 복잡한 데이터는 `pyspark` + `Jupyter` 사용
  - 데이터가 흐르고는 있지만, **나이스하게 흐르고 있을까?** 에 대한 의문
- `Kibana`에서 전문가가 아니더라도 용도에 따른 데이터 시각화 가능
  - Kibana만으로는 모든 목적을 달성할 수 없음
  - 오래된 데이터는 삭제하고 일정 기간의 데이터만 저장
  - 고급 집계와 정확한 집계 불가능
- S3에 모든 데이터 저장
  - 오래된 데이터는 유실되는게 아니라, S3에 있음
  - DL, 가공된 DW 등도 모두 S3에 저장
  - 여기에 모든 데이터가 있긴 한데, 모든 구성원에게 나이스하게 흐르고 있지 못함
- 대시보드: `Athena` + BI Tool(Mode Analytics)
- 데이터 분석, 추출: `pyspark` + `Jupyter`
- 이게 나이스하지 않음
- Athena
  - Scan당 비용. Full Scan을 남발하면 비용 폭발 (지난달 대비 5천불 증가한 적도 있음)
  - 성능이 좋지 않음. 느린 쿼리를 최적화할 수 있는 여지가 적음
  - On Demand가 아님(성능 옵션이 없음). 쿼리 개선만 가능
  - Mode Analytics는 사용자당 비용. Athena는 시각화 툴이 아니기 때문에, 시각화 기능 생각해야 함
- `Spark`로 천하통일?
  - Spark Thrift Server: JDBC로 쿼리 가능(Spark SQL)
  - `flintrock`이라는 CLI 사용
    - Spark 클러스터 생성 툴
    - CLI를 웹에 래핑해서 사용
  - 기존에는 데이터 과학자 등 특정 구성원만 사용. 근데 이제는 모든 사용자에게 접근 가능해야 함
- `Kubernetes`
  - Spark 2.3부터 native하게 지원. 3.0에서는 동적 리소스 할당도 지원
- Spark on Kubernetes
  - 이 부분은 Spark와 Kubernetes를 좀 더 이해하고 읽어야 할 듯(Pod 개념)
  - Cluster Autoscaler
- 시각화는 범용성이 중요 -> `Superset`
  - Spark Thrift Server + Superset: 비개발 직군에서 오는 데이터 추출 작업 요청에 대해 Superset으로 SQL 작성 후 공유. 업무 부담 적어짐
- 일반 사용자는 개선됐지만, 분석가의 데이터 분석 환경은 괜찮은가?
  - 이 경우도 Kubernetes 사용
  - Jupyterhub: 각 사용자가 웹에서 Jupyter 요청하면, 사용자별 Spark 서버 사용. Kubernetes가 자동으로 Pod 사용하여 사용자별 할당. Jupyter 사용시 로컬이 아닌 원격 서버 사용하는 느낌인 듯
- Spark Thrift Server를 Kubernetes 상에서 사용
  - 사용량에 따른 동적 할당
  - JDBC를 이용해서 모든 데이터 분석
  - ETL pipeline도 Kubernetes에서 사용
- 궁극적인 이점
  - 잡다한 문제 대신 데이터 분석에 집중
  - 본연의 미션, 가치에 몰입할 수 있게 됨
- Q&A
  - 데이터 엔지니어링 기본적인 공부: `Kafka`, `Spark`, `Airflow` 정도가 업계 표준같은 느낌

<br>

---

### 4. 데이터를 중심으로 성장 전략과 문화 만들기

- 소개팅 서비스 Cupist(GLAM) 분석가
- **Business** 를 성장시키기 위한 **Analyst** 의 입장에서 다루는 세션
- 데이터 분석가: 데이터를 통해 회사의 성장에 기여하는 역할
  - 프로덕트 설계 단계에서도 참여
- 서비스의 성장세가 기대에 비해 더딤 -> 글램이 데이터를 잘 활용하고 있는지에 대한 의문
- 회사의 분기 목표에 따른 전략 수립
  - 거래액, 유저 수 같은 **궁극적인 지표** 를 달성하는 데 집중
  - 이를 위해 어디에 집중해야 하는가?
- Growth Accounting
  - DAU 성장세를 신규 유입/재유입으로 나눴을 때 양상 보기
  - 사용자 유입부터 정착 단계까지 단계별 지표 확인. 이 때 생각의 관성을 버리고(그동안 많이 봤던 지표를 위주로 보지 않고), 단기가 아닌 장기적인 데이터를 바라봄
  - KPI 목표(e.g. 가입률 늘리기)의 우선순위 정함. 재방문율 같은 수치는 많은 결과에 영향을 받는 후행 지표이기 때문에, 재방문율에 영향을 미치는 수치부터 우선 확인
  - KPI 목표별로 테스트 방법 여러 가지 설계. 리소스와 결과를 반영하여 테스트 방법 결정
- 불필요한 리소스는 투입하지 않음
  - A/B 테스를 위한 통계적 유의성 확보를 위한 데이터 수집 작업은 엔지니어링 리소스가 크기에, 하지 않음
  - KPI 목표별 테스트를 진행하며, 마무리된 테스트도 있고 중간에 중단된 테스트도 있음
  - 1~2% 개선이 아닌 5%, 10% 개선을 찾고 있기 때문에, 영향이 별로 없을 정밀한 GUI 테스트보다는 굵직한 테스트에 집중
- 목표 지향 방식으로 인한 3가지 이점
  - 구성원들이 목표에 집중할 수 있는 문화. 조직을 응집시킬 수 있음
  - 데이터 팀도 임팩트에 집중할 수 있는 업무로 정착. 이 과정에서 기존 업무는 일부 중단
    - 기존에는 구성원들의 데이터 요청 작업을 지원해 주는 것이 가장 의미가 있다고 생각했지만, 그걸 넘어서 데이터를 통한 성장을 보여 주는 것이 데이터가 흐르는 조직이라고 생각
  - 프로젝트 완수하며 과도한 리소스 발생. 이제는 **지속 가능성** 을 위해 데이터가 흐르는 시스템과 분석가가 더 필요

<br>

---

### 5. 대학교 경제학 강의를 "힙"한 데이터 특강으로 피봇팅한 썰

- RPA를 활용한 기사 자동화 스타트업인 M-Robo 대표
- 비대면 경제학 실습 과목에서 학생들의 니즈 파악을 통한 강의 진행
- 강의 방식 바꾼 이유: 은행, 증권사, 특히 국책은행 등 금융 기관 가기 위한 취업 skill 필요. 특히 **디지털 직군**
  - 상경계에서 배우기 힘든 인공지능, 빅데이터 등 요구로 함. 하지만 경제학 과목에선 다루기 힘든 내용
  - IT 직군이 영업 직군보다 많이 준비하고 있는 상황
  - 학교 수업은 취업에 도움이 되지 않는 교양 수준에 그칠 수도 있음
  - 비대면 수업에서는 라이센스 문제로 SPSS가 아닌 오픈 소스를 사용해야 함
- 강의 구성의 문제
  - 강의계획서 상에서는 경제학 이론을 다루어야 함
  - 마땅한 교재가 없음
  - 학생들이 이해하고 있는지 확인할 수도 없음
- 강의 구성
  - 블로그 형태로 만듦. 5-10분 정도에 이해할 수 있게
  - 어려우니 처음에는 무조건 짤. 비지도 학습 같은 경우 눈 가리고 맞추게 하는 짤
  - 코드와 **코드에 대한 설명** 반드시 추가. 코드 자체보다는 설명에 중점
  - 면접과 실무에 필요한 지식을 다루려고 함
  - 데이터를 정리해서 제공, Kaggle 데이터도 제공(신용카드 이상탐지)
  - 적절한 보상(스벅 기프티콘)을 통해 흥미 유도
  - 경제에서 딥러닝을 적용하긴 어려우니, 통계 중심의 머신러닝 모델 사용(K-means 등)
  - 암호 화폐: 시계열 데이터 다룸. 강의 사이트 트래픽 그래프도 보여줌(시험 기간에 급증하는 것을 확인함으로 흥미 유발)

<br>

---

### 6. 의료 데이터에서의 임베딩

- 라인웍스
- Skip-Gram + Matrix Factorization
- 임베딩이란
  - `Word2Vec`에서 처음 접하게 됨
  - 정의: 범주형 자료를 연속형 벡터로 변환하는 것
  - 예) 점심봇의 경우, 어제 참치김치찌개를 먹었다고 하면 꽁치김치찌개를 추천해 줌(..)
  - 메뉴 자체를 원핫 인코딩할 경우, 비슷한 메뉴도 완전히 다른 벡터가 나오게 됨
    - 참김: [1,0,0]
    - 꽁김: [0,1,0]
    - 된장찌개: [0,0,1]
  - 메뉴의 특성을 이용하여 임베딩해야 함
    - 참김: [1.0, 0.5, 0.2]
    - 꽁김: [1.0, 0.4, 0.2]
    - 된장찌개: [0.5, 0.0, 1.0]
    - 어제 먹은 메뉴와 거리가 일정 값 이상 차이 나는 메뉴를 추천해 주도록 할 수 있음
- 의료 데이터에서의 임베딩
  - ICD-9 질병 코드: 코드 자체만으로는 질병 사이의 유사도를 확인할 수 없음
- Skip-Grams
  - Word2Vec 임베딩을 생성하기 위해 연구된 모델
  - 입력한 단어 기준으로 주변 단어 예측
  - window size에 따라 주변 단어 범위가 바뀜
  - 의료 데이터
    - 일별 증상-처방을 나열하면, 동시에 발생하는 이벤트(증상)에 대응하지 못함
- Matrix Factorization
  - 유저-영화 추천 행렬 예. 각 유저와 영화를 K차원의 행렬로 생성
  - 환자-질병 예측 행렬로 적용. 의료 이벤트를 K차원으로 정한 행렬
  - 동시 발생 이벤트는 처리할 수 있지만, 이벤트 순서는 고려되지 않음
  - 모든 카테고리를 한 번에 학습?? 하면 결과가 좋았음
- Skip-Grams, Matrix Factorization 모두 원핫 인코딩보다 높은 AUROC
- 아직 의료 데이터에서는 SOTA가 없음
  - 공개된 데이터셋이 없고, 병원마다 데이터 스키마가 모두 다르며, 코드 체계도 다름
  - 한 병원에서 최고의 성능을 보인 알고리즘이 다른 병원에선 최고가 아닐 수 있음
  - 다른 병원 논문을 우리 병원에서도 적용해 봐야 함
- 발표 내용은 라인웍스 블로그에도 있음

<br>

---

### 7. Airflow로 똑똑한 배치관리하기

- FLO 데이터 엔지니어 (전 DBA)
- 실적 지표 자동화하기
  - Before: GCP의 예약된 쿼리
  - After: `Airflow` (예약된 쿼리의 문제점 개선)
- 실적지표 데이터 마트 + 시각화 도구와 연계
- 데이터 구조
  - 13개의 데이터 마트 간의 dependency 존재
  - BigQuery의 예약된 쿼리: 설정한 시간에 특정 쿼리 수행. 자동화 목적은 달성하지만, 알람이 올 경우 성공인지 실패인지 알 수 없음. 그리고 앞 테이블에 잘못된 데이터가 들어갈 경우도 있음. 이 경우 수동으로 처리해야 함 -> Airflow 도입
- `Airflow`
  - 워크플로우 관리 도구. 현재 Apache의 탑 레벨 프로젝트
  - 설정한 시간에 특정 작업 진행
  - Python 기반 스크립트
  - 태스크를 DAG 형태로 구성
- DAG
  - 작업 단위인 태스크로 구성
  - 방향이 있는 비순환 그래프
- GCP Cloud Composer
  - 편리하게 모니터링 가능
  - `Kubernetes`에 `Airflow`가 올라가 있는 구조. 여러 서비스들이 연계되어 있어, 잘 모르고 구성하기에는 적합하지 않다고 생각하여 사용하지 않음
- VM 위에 Airflow 운영하고 있는 상태
- Airflow 웹 UI에서 DAG의 작업 상태 확인 가능. 성공/실패시 각각 다른 slack 메시지를 받음
- 도입 후 장점
  - 스케줄링
    - 기존에는 A -> B 작업의 경우, A 종료 예상 시간에 맞춰 B 작업 시작
    - A 작업에 문제가 생길 경우, 비정상 데이터로 B 작업하게 됨
    - Airflow는 A 작업 완료 후 B 작업 -> 데이터 정합성 높임
  - 중간에 실패한 작업 복구 쉬움
    - 클릭 한 번으로 Clear 가능. 해당 작업을 완전히 새로 시작하는 것처럼 할 수 있음
  - **웹을 통한 편리한 UI + 연속적인 작업 가능!**
- 레퍼런스가 많아서 Airflow를 선택하게 됨

<br>

---
