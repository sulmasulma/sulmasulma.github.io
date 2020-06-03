---
layout: post
title: AWS Lambda를 이용한 Serverless 카카오톡 챗봇 만들기
tags: [kakaotalk, serverless, Data Engineering, AWS, chatbot]
categories: [Data]
excerpt_separator: <!--more-->
---

서버리스 기반으로 작동하는 카카오톡 챗봇을 만드는 과정을 소개한다.<!--more--> 기본적으로는 음원 서비스에서 제공하는 음악 추천 서비스를 컨셉으로 하여, 아티스트를 입력하면 관련 아티스트와 그 아티스트의 노래를 제공하려고 한다. 우선 이 글에서는 관련 아티스트를 추천해 주기 전에, **입력한 아티스트의 정보** 를 제공하는 과정에 대해 다룰 것이다.
사용한 기술 스택은 아래와 같다.

- 데이터: `Spotify` API
- 서버 (서버리스): AWS `Lambda`
- DB: AWS RDS-`MySQL`, `DynamoDB`
- 이외 AWS 서비스: `API Gateway`(API 생성 및 관리), `S3`(서비스에 필요한 코드 저장)

[올인원 패키지 : 데이터 엔지니어링](https://www.fastcampus.co.kr/data_online_engineering)에서 `Spotify` API와 AWS 데이터 파이프라인을 이용하여 **페이스북 메신저** 챗봇을 개발하는 내용을 다루었다. 하지만 페이스북 메신저의 경우 잘 사용하지도 않는 플랫폼일 뿐만 아니라, 배포하려면 채널이 비즈니스 인증을 받아야 하는데 이마저도 코로나19로 인해 업무가 중단되었다고 한다. 반면 **카카오톡** 챗봇은 비즈니스 인증을 받지 않아도 서비스 배포가 가능하다. 따라서 카카오톡 챗봇을 만든 과정을 소개하고자 한다.

목차는 아래와 같다.

1. [카카오톡 채널 생성 및 카카오 오픈빌더 계정 만들기](#카카오톡-채널-생성-및-카카오-오픈빌더-계정-만들기)
2. [AWS Lambda 함수 생성](#aws-lambda-함수-생성)
3. [카카오톡 메시지 입출력 및 봇 테스트](#카카오톡-메시지-입출력-및-봇-테스트)
4. [Spotify API 데이터 이용하여 아티스트 정보 제공](#spotify-api-데이터-이용하여-아티스트-정보-제공)
<br>
<br>

### 카카오톡 채널 생성 및 카카오 오픈빌더 계정 만들기

카카오톡 챗봇을 만들려면 2개의 과정이 필요하다.

1. [카카오톡채널 관리자센터](https://center-pf.kakao.com)에서 채널 생성
2. [카카오 i 오픈빌더](https://i.kakao.com/)에서 봇 생성 후 채널에 연결

채널 생성은 바로 되지만, 카카오 i 오픈빌더에서 계정 승인을 받으려면 2-3일 정도 걸린다. 필요시 미리 해 두는 것을 추천한다.

카카오 오픈빌더 계정 승인 후 봇 생성을 하고, 봇으로 들어가면 **설정** 메뉴에서 카카오톡 채널을 연결할 수 있다.

![스크린샷 2020-06-03 오후 3.51.41](https://i.imgur.com/EviDrsp.png)

#### 시나리오 생성 및 엔티티 설정

이제 **시나리오** 메뉴에서 시나리오 블록을 만들고, 예상되는 사용자 발화 패턴을 몇 개 입력한다. 오픈빌더에서는 사용자의 발화 패턴을 인식하여 메시지 중 필요한 부분을 추출할 수 있다. **엔티티** 라는 데이터 사전을 정의하여, 엔티티에 들어있는 단어를 추출하는 방식이다. 이미 정의된 엔티티(시스템 엔티티)를 사용할 수도 있고, 사용자 정의 엔티티(나의 엔티티)를 사용할 수도 있다. 상단 메뉴의 "엔티티"로 들어가면 된다.

*시나리오, 엔티티 등 오픈빌더의 용어 설명에 대해서는 [공식 문서](https://i.kakao.com/docs/key-concepts-entity)에 자세히 나와 있다.*

나는 아티스트(가수) 이름에 기반한 서비스를 제공할 것이므로, 시스템 엔티티의 콘텐츠 항목에 있는 `@sys.person.group`, `@sys.person.name` 두 엔티티를 사용할 것이다.

![스크린샷 2020-06-03 오후 4.32.20](https://i.imgur.com/Mh7GV28.png)

- 아티스트 목록을 가져와 `csv` 파일로 나의 엔티티에 등록할 수도 있지만, 이 엔티티로 웬만한 아티스트는 인식이 된다.

#### 사용자 발화 입력

오픈빌더는 머신러닝으로 사용자의 발화를 학습한다. 예상되는 사용자 발화(아티스트 이름)를 50개 이상 입력하고 각 발화에 맞는 엔티티는 등록한다. 이후 **머신러닝 실행** 버튼을 눌러 학습을 진행하면 웬만한 아티스트는 엔티티에 기반하여 인식이 되었다.

![스크린샷 2020-06-03 오후 4.34.04](https://i.imgur.com/9vSd492.png)
- 국내 아티스트와 해외 아티스트를 골고루 입력해 주었다.



### AWS Lambda 함수 생성




### 카카오톡 메시지 입출력 및 봇 테스트

- 오픈빌더의 스킬 이용. 모든 스킬은 POST 메소드로 요청됨
- 테스트 화면에선 오류 나는데, 시나리오-블록에서 봇테스트 하면 요청이 감

[출처 링크](https://i.kakao.com/docs/skill-build#%EC%8A%A4%ED%82%AC-%ED%85%8C%EC%8A%A4%ED%8A%B8)
파라미터를 입력하여 만들어진 JSON을 직접 서버에 입력하고 싶은 케이스가 있습니다. 클립보드로 복사 버튼을 누르면 현재 JSON 창에 있는 값이 클립보드에 복사됩니다. 이를 활용하여 더 수월하게 테스트를 진행할 수 있습니다.

파라미터를 입력하여 만들어진 JSON을 서버로 전송할 일만 남았습니다. 스킬 서버로 전송 버튼을 누르면 스킬 요청 전송이 시작됩니다. 스킬 테스트는 입력된 URL 필드를 확인하여 해당 URL을 엔드포인트로, 그리고 JSON값을 body로 하여 post 요청을 전송합니다.

### Spotify API 데이터 이용하여 아티스트 정보 제공

- `AWS Lambda`로 하려면 반드시 `header`(API 키) 필요
  - [오픈빌더 관련 링크](https://i.kakao.com/docs/skill-build#%EC%8A%A4%ED%82%AC-%ED%85%8C%EC%8A%A4%ED%8A%B8)
  - API Key 설정: [API 키 소스 선택](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/api-gateway-api-key-source.html)
- `header` 없어도 가능
  - [링크](https://smartshk.tistory.com/9)

###

- 요청할 파라미터: action > params
  - `artist_name`으로 the beatles 를 준 상태
- request 형식

```JSON
{
  "intent": {
    "id": "qqz5oy80luysrtiq4ck4ql4h",
    "name": "블록 이름"
  },
  "userRequest": {
    "timezone": "Asia/Seoul",
    "params": {
      "ignoreMe": "true"
    },
    "block": {
      "id": "qqz5oy80luysrtiq4ck4ql4h",
      "name": "블록 이름"
    },
    "utterance": "발화 내용",
    "lang": null,
    "user": {
      "id": "363763",
      "type": "accountId",
      "properties": {}
    }
  },
  "bot": {
    "id": "5ec3be532ca48c00011f5300",
    "name": "봇 이름"
  },
  "action": {
    "name": "la6nkw2a2r",
    "clientExtra": null,
    "params": {
      "artist_name": "the beatles"
    },
    "id": "3tllk8vhdoirf6bbkbl5k3co",
    "detailParams": {
      "artist_name": {
        "origin": "the beatles",
        "value": "the beatles",
        "groupName": ""
      }
    }
  }
}
```

- response 형식
- 아래 두 가지 형식의 메시지를 사용할 것이다.
- SimpleText

```json
{
    "version": "2.0",
    "template": {
        "outputs": [
            {
                "simpleText": {
                    "text": "간단한 텍스트 요소입니다."
                }
            }
        ]
    }
}
```

- BasicCard

```json
{
  "version": "2.0",
  "template": {
    "outputs": [
      {
        "basicCard": {
          "title": "보물상자",
          "description": "보물상자 안에는 뭐가 있을까",
          "thumbnail": {
            "imageUrl": "http://k.kakaocdn.net/dn/83BvP/bl20duRC1Q1/lj3JUcmrzC53YIjNDkqbWK/i_6piz1p.jpg"
          },
          "profile": {
            "imageUrl": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT4BJ9LU4Ikr_EvZLmijfcjzQKMRCJ2bO3A8SVKNuQ78zu2KOqM",
            "nickname": "보물상자"
          },
          "social": {
            "like": 1238,
            "comment": 8,
            "share": 780
          },
          "buttons": [
            {
              "action": "message",
              "label": "열어보기",
              "messageText": "짜잔! 우리가 찾던 보물입니다"
            },
            {
              "action":  "webLink",
              "label": "구경하기",
              "webLinkUrl": "https://e.kakao.com/t/hello-ryan"
            }
          ]
        }
      }
    ]
  }
}
```

- [github 링크](https://github.com/sulmasulma/kakao-chatbot/blob/master/lambda_function.py)에서 코드를 찾아보실 수 있습니다.


---
출처
- [응답 타입별 JSON 포맷](https://i.kakao.com/docs/skill-response-format)
  - Response 형태: [SkillResponse](https://i.kakao.com/docs/skill-response-format#skillresponse)
  - 메시지 종류에 따라 `json` 형태의 response를 어떤 형식으로 만들어야 하는지 안내하고 있음
- [[AWS] AWS Lambda + API Gateway와 카카오 오픈빌더로 급식 메뉴 챗봇 만들기](https://yuda.dev/278)
- [AWS Lambda 에서 NumPy, Pandas 쓰는 법](https://smartshk.tistory.com/9)
