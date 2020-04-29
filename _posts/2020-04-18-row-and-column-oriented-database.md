---
layout: post
title: 행 지향 데이터베이스 vs 열 지향 데이터베이스
tags: [Database, MPP, Big Data]
categories: [Data]
excerpt_separator: <!--more-->
---
데이터 기반 서비스를 운영하려면, 데이터 마트를 구축하여 초 단위로 처리가 이루어져야 한다. 처리 시간을 줄이는 방법들을 정리해 보았다.<!--more-->

- RDB의 경우 메모리 용량이 충분하여 모든 데이터를 메모리에 올릴 수 있다면 응답 속도가 매우 빠르지만 (latency가 적다고 표현), 메모리가 부족하면 성능이 급격히 저하됨
- 압축을 통해 데이터의 사이즈를 줄이고, 여러 디스크에 분산함으로써 데이터의 로드에 따른 지연을 줄임
- 빅데이터로 취급되는 데이터 대부분은 디스크 상에 있기 때문에, 쿼리에 필요한 최소한의 데이터만을 가져옴으로써 지연을 줄일 수 있음

### 지연을 줄이는 방법
1. 열 단위 데이터 압축 사용
  - **행 지향 (row-oriented) DB**: 일반적인 DB에 해당. 레코드 단위의 읽고 쓰기에 최적화되어 있음
    - `Oracle`, `MySQL` 등
    - 매일 발생하는 대량의 트랜잭션을 지연 없이 처리하기 위해 데이터 추가를 행 단위로 효율적으로 하는 구조
    - 데이터 검색을 고속화하기 위해 인덱스(index) 사용. 인덱스가 없다면 모든 데이터를 로드해야 원하는 레코드를 찾을 수 있기 때문에 많은 디스크 I/O가 발생하고, 성능 저하됨
    - 레코드 단위로 데이터가 저장되어 있기 때문에 **필요 없는 열까지 디스크로부터 로드** 됨
  - **열 지향 (column-oriented) DB**: 데이터 분석에 사용되는 DB. 칼럼 단위의 읽고 쓰기에 최적화되어 있음
    - `Teradata`, `Amazon Redshift` 등
    - 열별로 데이터가 저장되어 있기 때문에, 데이터 분석시 **필요한 열만을 로드** 하여 디스크 I/O를 줄임
    - 같은 열에는 유사한 데이터가 반복되기 때문에, **매우 작게 압축** 할 수 있음
        - 압축되지 않은 행 지향 DB와 비교하여 1/10 이하로 압축 가능

2. MPP 아키텍처에 의한 데이터 처리 병렬화
  - 행 지향 DB에서는 하나의 쿼리를 하나의 스레드에서 실행. 각 쿼리는 충분히 짧은 시간에 끝나는 것으로 생각하므로, 하나의 쿼리를 분산 처리하는 상황은 가정하지 않음
  - 반면 열 지향 DB에서는 디스크에서 대량의 데이터를 읽기 때문에 하나의 쿼리를 실행하는 시간이 길어짐
    - 하나의 쿼리를 다수의 작은 태스크로 분해하고, 이를 병렬로 실행함
    - 예: 1억 레코드로 이루어진 테이블의 합계(sum)를 계산하기 위해 10만 레코드씩 구분하여 1,000개의 태스크로 나눔. 각 태스크는 독립적으로 10만 레코드의 합계를 집계해 마지막에 결과를 모아 총 합계를 계산
    - 이런 방식 때문에 스몰 데이터보다는 수억 레코드 이상의 빅 데이터에 적합함

- 결론적으로, 데이터 마트의 지연을 작게 유지하기 위해 **레코드 수에 따라 다른 DB** 를 사용하는 것이 좋음
  - RDB: 수천만 이하의 레코드
  - MPP(열 지향) 데이터베이스: 수억 이상의 레코드