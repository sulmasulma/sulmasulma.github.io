---
layout: post
title: AWS Lambda를 이용한 Serverless 카카오톡 챗봇 만들기 1
tags: [kakaotalk, serverless, Data Engineering, AWS, chatbot]
categories: [Data]
excerpt_separator: <!--more-->
---

서버리스 기반으로 작동하는 카카오톡 챗봇을 만드는 과정을 소개한다.<!--more--> 기본적으로는 음원 서비스에서 제공하는 음악 추천 서비스를 컨셉으로 하여, 아티스트를 입력하면 관련 아티스트 및 그 아티스트의 노래를 추천해 주려고 한다. 분량이 많아서 글을 나누었다.

1. **기본적인 환경설정 및 메시지 응답 테스트**
2. [입력한 아티스트의 정보 제공](https://sulmasulma.github.io/data/2020/06/07/kakaotalk-chatbot2.html)
3. 관련 아티스트 및 노래 추천

사용한 기술 스택은 아래와 같다.

- 데이터: `Spotify` API
- 서버 (서버리스): AWS `Lambda`
- DB: AWS RDS-`MySQL`, `DynamoDB`
- 이외 AWS 서비스: `API Gateway`(API 생성 및 관리), `S3`(서비스에 필요한 코드 저장)

[올인원 패키지 : 데이터 엔지니어링](https://www.fastcampus.co.kr/data_online_engineering)에서 `Spotify` API와 AWS 데이터 파이프라인을 이용하여 **페이스북 메신저** 챗봇을 개발하는 내용을 다루었다. 하지만 페이스북 메신저의 경우 잘 사용하지도 않는 플랫폼일 뿐만 아니라, 배포하려면 채널이 비즈니스 인증을 받아야 하는데 이마저도 코로나19로 인해 업무가 중단되었다고 한다. 반면 **카카오톡** 챗봇은 비즈니스 인증을 받지 않아도 서비스 배포가 가능하다. 따라서 카카오톡 챗봇을 만든 과정을 소개하고자 한다.

목차는 아래와 같다.

1. [카카오톡 채널 생성 및 카카오 오픈빌더 봇 기본 설정](#카카오톡-채널-생성-및-카카오-오픈빌더-봇-기본-설정)
2. [AWS Lambda 및 API Gateway 설정](#aws-lambda-및-api-gateway-설정)
3. [챗봇 테스트](#챗봇-테스트)

<br>
<br>

### 카카오톡 채널 생성 및 카카오 오픈빌더 봇 기본 설정

카카오톡 챗봇을 만들려면 2개의 과정이 필요하다.

1. [카카오톡채널 관리자센터](https://center-pf.kakao.com)에서 채널 생성
2. [카카오 i 오픈빌더](https://i.kakao.com/)에서 봇 생성 후 채널에 연결

채널 생성은 바로 되지만, 카카오 i 오픈빌더에서 계정 승인을 받으려면 2-3일 정도 걸린다. 필요시 미리 해 두는 것을 추천한다.

카카오 오픈빌더 계정 승인 후 봇 생성을 하고, 봇으로 들어가면 **설정** 메뉴에서 카카오톡 채널을 연결할 수 있다.

![20200603-1-channel](/assets/20200603-1-channel.png)

<br>

#### 시나리오 생성 및 엔티티 설정

이제 **시나리오** 메뉴에서 시나리오 블록을 만들고, 예상되는 사용자 발화 패턴을 몇 개 입력한다. 오픈빌더에서는 사용자의 발화 패턴을 인식하여 메시지 중 필요한 부분을 추출할 수 있다. **엔티티** 라는 데이터 사전을 정의하여, 엔티티에 들어있는 단어를 추출하는 방식이다. 이미 정의된 엔티티(시스템 엔티티)를 사용할 수도 있고, 사용자 정의 엔티티(나의 엔티티)를 사용할 수도 있다. 상단 메뉴의 "엔티티"로 들어가면 된다.

*시나리오, 엔티티 등 오픈빌더의 용어 설명에 대해서는 [공식 문서](https://i.kakao.com/docs/key-concepts-entity)에 자세히 나와 있다.*

나는 아티스트(가수) 이름에 기반한 서비스를 제공할 것이므로, 시스템 엔티티의 콘텐츠 항목에 있는 `@sys.person.group`, `@sys.person.name` 두 엔티티를 사용할 것이다.

![20200603-2-entity](/assets/20200603-2-entity.png)

- 이미 등록되어 있는 엔티티 외에도, 아티스트 목록을 가져와 `csv` 파일로 나의 엔티티에 등록하여 사용할 수도 있다.

<br>

#### 사용자 발화 입력

오픈빌더는 머신러닝으로 사용자의 발화를 학습한다. 예상되는 사용자 발화(아티스트 이름)를 50개 이상 입력하고 각 발화에 맞는 엔티티는 등록한다.

![20200603-3-utterance](/assets/20200603-3-utterance.png)
- 국내 아티스트와 해외 아티스트를 골고루 입력해 주었다.

사용자 발화를 충분히 입력하고 **머신러닝 실행** 버튼을 눌러 학습을 진행하면, 웬만한 아티스트는 엔티티에 기반하여 인식이 된다.

<br>

#### 엔티티 매핑

이제 각 발화에 엔티티를 매핑해야 한다. 예를 들어 `태연`을 입력하고 더블클릭하면, 이 발화에 맞는 엔티티를 등록할 수 있다. 태연은 인물 > 개인에 해당하므로, `@sys.person.name` 엔티티를 선택한다.

![20200603-3-1-entitymapping](/assets/20200603-3-1-entitymapping.png)

- 엔티티를 선택해 주면 해당 단어는 파란색 테두리로 싸이게 되고, 카카오톡 메시지를 보낼 때 파라미터로 인식할 수 있다.

<br>

#### 파라미터 설정

아직 `태연`을 입력하면 파라미터가 할당이 되는 것이 아니다. 아래로 내려가 **파라미터 설정** 메뉴에서 변수 이름을 설정해야 한다.

![20200603-3-2-parameter](/assets/20200603-3-2-parameter.png)

아티스트가 `오마이걸`과 같은 그룹일 경우 `group`으로, `아이유`와 같은 솔로 가수일 경우 `solo`라는 파라미터가 생기도록 설정했다. +를 눌러 파라미터를 추가하면, 엔티티를 인식하여 파라미터가 할당된다.

![20200603-3-3-parameter2](/assets/20200603-3-3-parameter2.png)

여기까지 진행하고 시나리오 창 상단에 있는 **저장** 버튼을 누른다.

<br>

---

### AWS Lambda 및 API Gateway 설정

AWS `Lambda`는 서버가 항상 켜져 있는 것이 아니라, 사용자의 request가 있을 때만 클라우드 자원을 할당하는 서버리스(Serverless) 서비스이다. 서버를 항시 가동하는 것에 비해서 컴퓨팅 자원이 적은 편이다. AWS에서도 프리티어 기준 월 100만 회 request가 가능하여, 내가 운영하는 서비스에서는 사실상 무한정 서비스가 가능하다.

`Lambda`에서 lambda function을 만들고, `API Gateway`에서 API를 생성하여 lambda function과 연결해 준다. 이렇게 하면 카카오톡 메시지 request가 발생했을 때 `API Gateway`에서 생성한 endpoint로 요청이 가고, lambda function이 작동하여 사용자에게 response를 응답해 준다.

<카카오톡 사용자 -> API Gateway -> Lambda -> 카카오톡 사용자>

#### Lambda 함수 생성

AWS Lambda에서 함수를 생성한다. 나는 런타임으로 `Python 3.7`을 선택했고, 권한 부분은 따로 건드리지 않았다.

![20200603-4-lambda function](/assets/20200603-4-lambda%20function.png)

<br>

#### API Gateway에서 REST API 생성

생성한 lambda function으로 들어가 **트리거 추가** 버튼을 누른다.

![20200603-5-api](/assets/20200603-5-api.png)

트리거 구성에서 **API 게이트웨이** 를 선택하여 새 API를 생성한다. API type은 **REST API** 로, 보안은 **열기** 를 선택한다. 보안은 **API key** 로 해야 좋지만, 오픈빌더에서 API 키를 헤더로 설정해도 도저히 요청이 가지 않아서 API 키가 없는 `열기`를 선택했다. 이외의 부분은 따로 건드리지 않았다.

![20200603-6-trigger](/assets/20200603-6-trigger.png)

생성한 API Gateway로 들어가면 **ANY** 리소스를 확인할 수 있다. GET, POST 등 어떤 메소드로 요청해도 받을 수 있다는 것이다. 작업에서 **API 배포** 를 눌러 새 스테이지에 배포한다.
- *참고로 카카오톡 메시지 request는 모두 `POST` 메소드로 요청된다.*

![20200603-7-apideploy](/assets/20200603-7-apideploy.png)

- 스테이지는 아래와 같이 API 엔드포인트에 들어갈 일종의 함수 이름이라고 보면 된다.
  - `API URL/{스테이지 이름}`

보안을 **API key** 로 선택했다면, 메서드 요청으로 들어가 **API 키가 필요함** 을 true로 바꾸고, API 키를 발급받으면 된다.

![20200603-8-apikey](/assets/20200603-8-apikey.png)

- API 키를 사용하는 방법은 [AWS 공식 문서](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/api-gateway-api-key-source.html)에 나와 있다. 다만 카카오 오픈빌더에서 이 키를 정상적으로 사용 가능한지는 모르겠다.

<br>

#### Lambda에서 코드 수정

이제 카카오톡 메시지를 request로 보냈을 때 Lambda에서 정상적으로 response를 리턴하는지 확인하는 단계이다.
위에서 생성한 Lambda 함수로 들어가 **함수 코드** 탭에서 Code entry type을 **코드 인라인 편집** 으로 설정하고, `lambda_function.py` 파일을 선택한다.

![20200603-9-lambda code](/assets/20200603-9-lambda%20code.png)

예시 코드로 다음과 같이 입력하고, 상단 메뉴에 있는 **저장** 을 누른다. 기본적인 텍스트 응답을 해 주는 코드이다.

```py
import json

def lambda_handler(event, context):

    # 메시지 내용은 request의 ['body']에 들어 있음
    request_body = json.loads(event['body'])
    params = request_body['action']['params']
    solo = params['solo'] # 솔로 아티스트 파라미터 생기는지 테스트

    result = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleText": {
                        "text": "요청한 아티스트는 {}입니다.".format(solo)
                    }
                }
            ]
        }
    }

    return {
        'statusCode':200,
        'body': json.dumps(result),
        'headers': {
            'Access-Control-Allow-Origin': '*',
        }
    }

```

<br>

---

### 챗봇 테스트

이제 위에서 생성한 API 엔드포인트로 HTTP request를 보낼 것이다. 오픈 빌더로 돌아가 **스킬** 탭에서 스킬을 생성하고, API URL을 입력한다. 위에서 API 키를 설정했다면, 헤더값의 key로 `x-api-key`, value로 키를 입력하면 된다.

![20200603-10-skill](/assets/20200603-10-skill.png)

API URL은 Lambda에서 API 게이트웨이를 클릭하면 해당 API에 대한 정보가 나오고, 삼각형의 토글 버튼을 클릭하여 펼치면 나온다.

![20200603-11-apiurl](/assets/20200603-11-apiurl.png)

스킬 설정 창에서 아래로 내려가 파라미터를 임의로 설정하고 request를 테스트해 볼 수 있다.

![20200603-12-skilltest](/assets/20200603-12-skilltest.png)

그러면 **응답 미리보기** 에서 정상적으로 응답하는지 확인할 수 있는데, API 키를 설정한 것과 관계없이 처리가 되지 않는다.

![20200603-13-responsetest](/assets/20200603-13-responsetest.png)

AWS `CloudWatch`에서 request를 수신하면 로그를 확인할 수 있는데, 로그가 아예 나오지 않는다. [오픈빌더 공식 문서](https://i.kakao.com/docs/skill-build)에서는 Lambda를 사용했을 때 API 키 처리를 해 줘야 한다고 나와 있는데, 나는 키를 설정하지 않아도 실제 서비스에선 문제가 없었다. 테스트 창에 오류가 있는 것으로 보인다.

어쨌든, 실제 메시지를 주고받는 데에는 문제가 없다. 우선 스킬 설정에서 API 정보를 입력하고 **저장을 누른다.** 그리고 테스트 창 말고, 시나리오로 돌아간다.

**파라미터 설정** 메뉴에서, 위에서 저장한 스킬을 연결한다.

![20200603-14-skillconnect](/assets/20200603-14-skillconnect.png)

하단의 **봇 응답** 메뉴에서 스킬데이터를 누르면, 스킬에서 설정한 대로 response를 줄 수 있다.

![20200603-15-skilldata](/assets/20200603-15-skilldata.png)

`성시경`을 사용자 발화 예시 중 하나로 입력하고, `@sys.person.name`엔티티를 이용한 `solo`라는 파라미터로 설정했다고 가정하자. 상단 메뉴에 있는 **봇테스트** 창을 열어 `성시경`을 입력하면, Lambda 코드에서 입력한 대로 응답이 정상적으로 수신되는 것을 확인할 수 있다.

![20200603-16-responseexample](/assets/20200603-16-responseexample.png)

<br>
<br>

여기까지 했으면, Lambda를 통한 챗봇 만들기에 대한 기본적인 설정은 끝났다. [이어지는 글](https://sulmasulma.github.io/data/2020/06/07/kakaotalk-chatbot2.html)에서 `Spotify` API 데이터를 이용하여 아티스트의 정보를 제공하는 것에 대해서 다루겠다.


---
출처
- [[AWS] AWS Lambda + API Gateway와 카카오 오픈빌더로 급식 메뉴 챗봇 만들기](https://yuda.dev/278)
- [AWS Lambda 에서 NumPy, Pandas 쓰는 법](https://smartshk.tistory.com/9)
