---
layout: post
title: Amazon Athena, EMR, Redshift 차이
tags: [Data Engineering, AWS, Amazon, Athena, EMR, Redshift]
categories: [Data]
excerpt_separator: <!--more-->
---
AWS `Athena`를 `S3` 쿼리용으로 사용하고 있었는데, `Redshift`도 데이터 쿼리 기능을 제공하는 MPP (Massive parallel processing) data warehouse로 알고 있다. 그래서 두 서비스 및 같이 비교하여 제공하고 있는 EMR까지 비교해 보았다.<!--more-->
<!-- - 표로 정리할 필요도 있음 -->

### Redshift
- **데이터 웨어하우스** (DW)
- **여러 조인 및 하위 쿼리가 포함** 된 매우 복잡한 SQL과 관련된 워크로드에 대해 **가장 빠른 쿼리 성능** 을 가짐
- 인벤토리 시스템, 금융 시스템, 소매 판매 시스템 등 **다양한 소스의 데이터** 를 하나의 공통 형식으로 취합하여 장기간 보관하고 과거 데이터에서 비즈니스 리포트를 작성할 필요가 있을 때 사용
- TPC-DS: 이러한 사례에 맞게 설계된 표준 벤치마크
  - Redshift는 비정형 데이터에 최적화된 쿼리 서비스보다 최대 20배 더 빠르게 실행함
  - 결국 Redshift는 **복잡한 정형 데이터** 에 대한 쿼리에 최적화

### Athena
- `Presto` 기반의 **쿼리 서비스**
- 서버를 설정하거나 관리할 필요 없이 `S3`의 데이터에 대한 쿼리를 가장 쉽게 제공
- `Redshift`가 복잡한 정형 데이터에 최적화된 쿼리 서비스인 반면, Athena는 데이터 형식 지정, 인프라 관리에 관계 없이 데이터에 대한 대화형 쿼리를 쉽게 실행할 수 있음
- 사이트에서 성능 문제를 해결하기 위해 일부 웹 로그에서 빠른 쿼리를 실행하기만 될 경우에 좋음
- 데이터에 대한 테이블 정의하고, 표준 SQL 사용하면 됨
  - Athena DDL은 `Apache Hive` 기반. DDL 및 테이블/파티션을 작성/수정/삭제할 경우에 한해 Hive 쿼리 사용 가능


### EMR (Elastic Map Reduce)
- **데이터 처리** 프레임워크
- SQL 쿼리를 실행하는 것 외에도 다양한 작업 수행
  - Machine Learning, 그래프 분석, 데이터 변환, 스트리밍 데이터 등 애플리케이션에서 필요한 코딩 작업들
- `Hadoop`, `Spark`, `Presto`, `Hbase` 등 방대한 양의 데이터를 고도로 분산된 처리 프레임워크로 처리 및 분석
- 인프라, 클러스터를 구성하고 관리해야 함
  - 이에 대한 관리가 필요 없이 `S3` 데이터에 대한 쿼리를 실행하려면 `Athena` 사용
  - `Athena`에서 DDL 문을 통해 테이블을 정의하면 EMR을 통해 데이터 쿼리 가능

---
참고 문서
- [Amazon Athena FAQ](https://aws.amazon.com/ko/athena/faqs/) - AWS 공식 홈페이지
