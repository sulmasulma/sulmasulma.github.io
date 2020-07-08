---
layout: post
title: AWS Lambda를 이용한 Serverless 카카오톡 챗봇 만들기 3
tags: [kakaotalk, serverless, Data Engineering, AWS, chatbot]
categories: [Data]
excerpt_separator: <!--more-->
---
카카오톡 챗봇에서 데이터를 모델링 해 보았다.<!--more--> 아래와 같이 시리즈로 포스팅하고 있다.

1. [기본적인 환경설정 및 메시지 응답 테스트](https://sulmasulma.github.io/data/2020/06/03/kakaotalk-chatbot.html)
2. [입력한 아티스트의 정보 제공](https://sulmasulma.github.io/data/2020/06/07/kakaotalk-chatbot2.html)
3. **관련 아티스트 및 노래 추천**

[지난 글](https://sulmasulma.github.io/data/2020/06/07/kakaotalk-chatbot2.html)에서 `MySQL`과 `DynamoDB`를 이용하여, 아티스트를 요청받으면 해당 아티스트에 대한 인기 트랙 정보를 응답해 주는 내용을 다루었다. 이번 글에서는 Amazon `S3`를 이용하여, 요청받은 아티스트와 유사한 **관련 아티스트** 를 추천해 줄 것이다.

목차는 다음과 같다.

1. [다른 아티스트와의 거리 계산](#다른-아티스트와의-거리-계산)
- EC2에 파일 업로드하여 크론 탭으로 실행?

> 중간 중간에 코드를 첨부했는데, 전체 코드는 [Github]()에 올려 놓았다.

<br>

---

### 다른 아티스트와의 거리 계산

거리 계산법

<br>

#### 1. SimpleText



<br>
<br>

### 마무리하며



<br>

---
#### 참고 문서
- []()
