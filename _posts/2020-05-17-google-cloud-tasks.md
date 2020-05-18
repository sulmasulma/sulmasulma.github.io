---
layout: post
title: Google Cloud Tasks 써 보기
tags: [cloud, google, api, distribution]
categories: [Data]
excerpt_separator: <!--more-->
---
클라우드 상에서 API 쿼리를 분산처리하는 방법 중 하나인 [Google Cloud Tasks](https://cloud.google.com/tasks/docs/dual-overview)를 소개한다.<!--more-->

광고회사에서 데이터 분석을 할 때, YouTube Analytics API에서 데이터를 일별/영상별로 쿼리하여 2년 간의 데이터를 수집해야 하는 경우가 있었다. 채널의 영상이 200개 정도 되었던 것 같은데, `730(2년) * 200(영상 개수) = 144,000`번의 쿼리를 요청해야 모든 데이터 수집이 가능했다. Google 계정 인증 후 `access_token`을 발급하여 쿼리를 요청해야 했는데, 문제는 이 token이 1시간마다 만료된다는 것이었다. 144,000번의 쿼리를 요청해서 데이터를 받는 데 예상 시간이 10시간은 족히 넘었던 것으로 기억한다. 그래서 로컬 머신에서 이 작업을 하려면 수십 시간동안 `access_token`을 재발급해가며 계속해서 쿼리 요청을 해야 하는데, 503 에러가 나서 도중에 멈출 우려도 있었다. 결국 작업을 포기하고 해당 데이터 없이 프로젝트를 진행했다. 데이터 엔지니어링 공부를 시작하게 된 계기이기도 하다.

서론이 길었는데, 오늘 내용은 이렇게 로컬 머신에서 반복해서 API 쿼리를 요청하는 대신 **클라우드 상에서 분산처리** 를 통해 데이터를 저장하는 것이다. 애플리케이션 외부에서 독립적인 태스크들을 비동기식으로 처리하기 때문에 같은 작업을 반복하는 것보다 작업 시간을 큰 폭으로 줄일 수 있는 내용이다.

목차는 아래와 같다.

1. [Cloud Tasks 개요](#cloud-tasks-개요)
2. [Google Cloud Platform에서 Cloud Tasks 설정]()
3. [HTTP Target Task 만들기]()
4. [MongoDB Atlas 계정 및 Cluster 생성]()
5. [API 데이터 요청, MongoDB 및 GCP에서 상태 확인]()
6. [최종 데이터 확인, BigQuery에 Load]()

- ~~실제 데이터 요청시 로컬 머신 vs Cloud Tasks 소요 시간까지 비교해 보기~~

### Cloud Tasks 개요

태스크들은 기본적으로 클라우드 상의 큐(Queue)에 추가되고, 태스크가 성공적으로 실행될 때까지 큐는 태스크를 지속한다.



- BigQuery에 데이터 적재까지 해 보기?
- 내용: 레코드 추가용 데이터는 `MongoDB`에, 최종 데이터는 `BigQuery`에 넣는 이유
  - `MongoDB`는 document 단위로 데이터가 적재되며, 동시다발적인 레코드 추가 속도가 빠른 편
  - `BigQuery`는 대용량 데이터 Load에 최적화된 Data Warehouse이기 때문에, 여러 레코드 추가보다는 하나의 큰 데이터에 적합함

---
출처
- [Cloud Tasks 개요](https://cloud.google.com/tasks/docs/dual-overview)
- [Google Cloud Platform에서 Cloud Tasks 설정](https://cloud.google.com/tasks/docs/quickstart-appengine)
- [HTTP Target Task 만들기](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
