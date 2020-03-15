---
layout: post
title: The rise of SQL-based data modeling and DataOps
tags: [Data Engineering, Data Modeling, SQL, NoSQL, DataOps, Data Pipeline]
categories: [Readings]
excerpt_separator: <!--more-->
---
 Medium에서 데이터 모델링 관련 글: [The rise of SQL-based data modeling and DataOps](https://towardsdatascience.com/the-rise-of-sql-based-data-modeling-and-dataops-5b8e3270e101) 정리 <!--more-->

### SQL 기반 RDBMS의 부활 (resurgence)
- 최근 수 년간 데이터의 양 증가
- 몇 년 전까지 SQL 기반의 전통적인 데이터 관리법(RDBMS)은 과거의 유물로 여겨짐. 대용량의 데이터를 scale할 수 없었기 때문
- 대신 `HBase`, `Cassandra`, `MongoDB`와 같은 NoSQL이 사용하기도 쉽고, scalable 한 방법으로 각광받음
- **최근에는 SQL 기반 DB의 부활**
- Amazon CTO는 `PostgreSQL`, `MySQL`과 호환 가능한 `Aurora DB`가 AWS 역사상 가장 빠르게 성장하고 있는 서비스라고 밝힘
- 가격이 비싸 대기업에서만 사용했던 SQL 기반 **MPP**(Massive parallel processing) data warehouse는 클라우드로 이동하는 양상
  - `Google BigQuery`, `Amazon Redshift`, `Azure Data Warehouse` 등
  - `Snowflake`: 독립된 DW 서비스로, 기업가치 39억 달러에 이름
- 클라우드 DW의 등장으로 이전에는 불가능했던 복잡한 분석 쿼리를 초/분 단위로 낼 수 있도록 해 줌. 전통적인 DW에서는 몇 시간 걸리던 작업

### SQL을 통한 데이터 분석
- 이렇게 SQL 기반 DB가 부활하게 된 이유는 다음과 같이 이해할 수 있음
  - NoSQL은 분석 쿼리에 적합하지 않음: join이 없어 복잡한 분석 쿼리를 할 수 없음
  - 표준화된 query language가 존재하지 않음: NoSQL DB vendor는 각자의 고유한 language를 사용함으로 분석가들 간에 마찰과 learning curve가 생기게 됨
- 대신 SQL은 거의 모든 데이터 분석가들이 친숙한, 범용성 있는 언어
- Google Spanner (MPP data management system)에 따르면,

> Spanner의 NoSQL API는 `point lookup`, `range scan`과 같은 메소드들을 제공하며, simple retrieval scenario에는 적합합니다. 다만 SQL은 더 복잡한 데이터 접근 패턴을 표현함으로 추가적인 value들을 산출할 수 있습니다.
> 
> Spanner의 SQL 엔진은 Standard SQL을 사용하여 Google의 내부 시스템인 `F1`, `Dremel`과 외부 시스템인 `BigQuery`와도 호환이 가능합니다. 이 호환성은 구글 사용자들이 시스템 간의 벽을 자유롭게 넘나들 수 있도록 해 줍니다.

- 클라우드 기반 MPP data warehouse가 각광받으면서, `Mode Analytics`, `Periscope Data`, `Redash`와 같은 스타트업들이 인기를 얻음
- SQL에 능숙한 분석가들은 이런 최신 클라우드 DW를 사용하여, 다른 언어나 tool을 배울 필요 없이 효과적인 차트와 대시보드를 구성할 수 있음
- 또한 SQL 코드는 단순한 text이기 때문에 관리하기도 쉬움

### SQL의 문제점
> 진입 장벽과 복잡성

- SQL이 장점만 가지고 있는 것은 아님
- SQL에 많이 의존하고 있는 tool들은 정적인 차트와 대시보드 이상의 것을 원하지만, SQL을 모른다는 장벽이 있음
- 이로 인해 business user들은 과거에 Excel로 raw data를 가져와 분석하던 방식으로 돌아갈 수 밖에 없게 만들고, 이로 인해 계산의 accuracy와 consistency가 떨어지게 됨
- 테이블 간의 join이 많아져 데이터 구조가 복잡해지게 되면, SQL 구문도 복잡해지고 쿼리를 작성할 때마다 상황에 맞는 적절한 join 관계를 찾아야 함
- 이는 SQL이 재사용 불가하다는 것을 말하며, 매번 비슷한 코드를 가지고 조금씩 다른 로직을 작성해야 하는 현상이 반복

### 전통적인 data modeling
- BI를 사용하는 전통적인 구조는 다음과 같음
- 분석가가 raw data를 가공하여 비즈니스 유저(기술 직군 X)에게 데이터를 제공
- 비즈니스 유저는 드래그앤 드롭 인터페이스로 BI 기능을 사용
- SQL 같은 query languague 없이 데이터 탐색 가능
- 대표적인 BI Tool: `Sisense`, `Tableau`, `PowerBI`
- **그러나 이런 Tool들은 현대의 DW에 맞게 설계되지 않음**
  1. 현대의 강력한 MPP DW는 데이터를 복제하여 여러 소스에서 사용하는데, 전통적인 DW 시스템에 맞춰 설계된 이런 BI Tool들은 가공되지 않은 raw data와 연결되어 있음
  2. data modeling 과정이 대부분 GUI 기반 -> 재사용하기 쉽지 않고, `git` 같은 version control 시스템의 이점을 누릴 수 없는 구조

### SQL 기반 data model approach
> SQL의 파워 + data modeling의 이점을 결합할 수 있어야 함

- `Looker`가 이런 접근법의 선구자 역할을 함
- data modeling language인 'LookML' 사용 -> database의 abstraction layer 역할을 함
- 기술 직군이 아닌 사람도 데이터를 탐색할 수 있게 **modeling input을 SQL query로** 변환해 주고, 이 query를 DB에 보냄
- SQL을 모르는 사용자도 현대식 MPP data Warehouse를 이용할 수 있게 해 줌
- 텍스트 기반 modeling language를 사용하는 것의 또 다른 장점은 modeling layer가 version control 가능하다는 것. 이로 인해 이슈 발생시 변경 사항을 추적할 수 있음

### SQL 기반 end-to-end data pipeline
- SQL 기반 data model approach는 빙산의 일각과 같음
- ETL 과정은 다음과 같음
  1. Extract - data source에서 데이터 추출
  2. Transform - 데이터를 최종적으로 사용 가능한 형식으로 변환
  3. Load - data warehouse로 데이터 저장
- MPP data warehouse가 출현하면서, **ETL** 접근법이 **ETL** 로 바뀌고 있음
  - 데이터 추출 -> DW로 직접 데이터 저장(Transform 없이)
  - 데이터가 저장되면 분석가들이 SQL로 데이터를 최종적으로 적절한 형태로 변환
  - 데이터 변환 과정이 SQL로 가능하기 때문에, SQL 기반 data modeling 방법 적용
- `Holistics`와 같은 Tool들이 `Looker`의 SQL 기반 data modeling 접근법 적용
  - 이에 L(Load), T(Transform) layer까지 나아감으로 end-to-end 분석 파이프라인이 완성됨
- 정리하면, 이와 같은 SQL 기반 data modeling의 장점은 3가지로 요약할 수 있음
  1. 데이터 분석가들이 end-to-end 데이터 분석 파이프라인을 형성할 수 있음
    - 업무에 필요한 SQL 지식은 그대로 유지하면서, 인사이트 형성을 위한 시간을 줄여줌
  2. Data context가 보존됨
    - 분석 결과의 정합성 보장. 데이터의 정확도 이슈를 디버깅하기 용이해짐
  3. data stakeholers(Data Engineer, Data Analyst, Data Scientist)들과 비즈니스 유저들의 간격을 좁혀, 데이터 기반 문화의 형성을 도움

### DataOps
- 소프트웨어 엔지니어링과 데이터 분석 비교 <!-- 표로 바꾸기 -->
  - 소프트웨어 엔지니어링: 소프트웨어를 개발함으로 가치 창출
  - 데이터 분석: 데이터 탐색 및 시각화를 통해 가치 창출
  - 둘 다 logical code (프로그래밍 언어나 SQL)를 deploy 하여 "production"하는 과정
- 소프트웨어 엔지니어링의 `DevOps`(개발 과정 효율화, 자동화)를 모티브로 한 `DataOps` 제안
- **DataOps**
  - `Analytics as code`
    - 분석 logic을 code화 하여 코드 관리 및 디버깅 자동화
  - `Unit tests for data`
    - 모든 transformation step에 테스트 과정을 거치게 하여 데이터 accuracy, quality 개선
  - `Centralized data wiki`
    - 조직 구성원 모두가 한 곳에서 데이터 탐색을 할 수 있도록 함

### 새로운 분석 패러다임
- SQL, data modeling, DevOps 각각은 서로 관련성이 별로 없는 오래 된 컨셉
- 그러나 이 셋이 결합되면 새로운 패러다임이 만들어짐 -> `DataOps paradigm`
- `Holistics` -> 통합 분석 플랫폼. 저자가 2년 이상 개발 중
