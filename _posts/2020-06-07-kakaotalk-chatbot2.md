---
layout: post
title: AWS Lambda를 이용한 Serverless 카카오톡 챗봇 만들기 2
tags: [kakaotalk, serverless, Data Engineering, AWS, chatbot]
categories: [Data]
excerpt_separator: <!--more-->
---
<!--more-->
카카오톡 챗봇 만들기를 시리즈로 포스팅하고 있다.

1. [기본적인 환경설정 및 메시지 응답 테스트](https://sulmasulma.github.io/data/2020/06/03/kakaotalk-chatbot.html)
2. **입력한 아티스트의 정보 제공**
3. [관련 아티스트 및 노래 추천]()

[지난 글](https://sulmasulma.github.io/data/2020/06/03/kakaotalk-chatbot.html)에서 AWS Lambda를 이용한 카카오톡 챗봇 세팅에 대해 다루었다. 이번 글에서는 [Spotify API 데이터](https://developer.spotify.com/documentation/web-api/reference/)를 이용하여, 아티스트를 요청받으면 해당 아티스트에 대한 정보를 response로 주는 것에 대해 다룰 것이다.



목차는 다음과 같다.

1. [카카오톡 말풍선 종류](#카카오톡-말풍선-종류)
2. [Spotify 가입 및 권한 설정](#spotify-가입-및-권한-설정)
3. [Spotify 데이터 종류](#spotify-데이터-종류)
4. [데이터를 말풍선 형태에 맞게 보내기](#데이터를-말풍선-형태에-맞게-보내기)

> 중간 중간에 코드를 첨부했는데, 전체 코드는 [github](https://github.com/sulmasulma/kakao-chatbot/blob/master/lambda_function.py)에 올려 놓았다.

<br>

---

### 카카오톡 말풍선 종류

카카오톡 챗봇으로 용도와 목적에 따라 여러 종류의 메시지를 보낼 수 있다. 여기서는 내가 사용할 두 가지만 간단히 소개하겠다.

#### 1. SimpleText

단순한 텍스트 출력이다.

![SimpleText](https://i.kakao.com/docs/assets/skill/skill-simpletext-example.png)

지난 글에서 테스트해 본 메시지 종류이다. Lambda 코드에서 아래 형태의 json 변수를 반환해야 한다.

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

<br>

#### 2. BasicCard

이미지와 설명, 버튼이 함께 있는 카드 형태의 메시지이다.

![BasicCard](https://i.kakao.com/docs/assets/skill/skill-basiccard-example.png)

json 형태는 다음과 같다.

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

`thumbnail` > imageUrl이 보이는 이미지의 주소이다. `profile` > imageUrl은 프로필 이미지인데 카드에 노출되는 부분은 아닌 것 같다. `social`도 마찬가지이다. 그래서 나는 `title`, `description`, `thumbnail`, `buttons` 부분만 담았다. 버튼의 경우 `messageText`는 누르면 해당 메시지가 입력되며, `webLinkUrl`은 누르면 해당 링크로 연결된다.

나는 안내 메시지는 `SimpleText` 형태로, 아티스트에 관한 정보는 아티스트 이미지와 함께 `BasicCard` 형태로 전달하기로 하여 이 두 타입을 소개해 보았다. 이외 여러 형태의 메시지에 대해서는 [공식 문서](https://i.kakao.com/docs/skill-response-format)에 나와 있다.

<br>

---

### Spotify 가입 및 권한 설정

Spotify API를 사용하기 위해서는 서비스에 우선 가입해야 한다. 한국에서는 서비스되지 않기 때문에 VPN을 통해 우회해야 한다. 우회 방법에 대해서는 [한국에서 Spotify(스포티파이) 사용하기](https://min7zz.tistory.com/834)에 나와 있다. TunnelBear라는 앱을 사용하면 된다. TunnelBear는 월 500MB만 무료로 제공하고 있는데, 가입 시에만 VPN을 사용하고 이후 로그인 정보가 남아 있는 세션에서는 다시 VPN을 사용할 필요가 없기 때문에 신경 쓰지 않아도 된다.
- 예전에 이와 관련해서 문서를 찾아보았을 때 VPN의 위치와 계정 상에서 위치를 같게 할 필요가 있던 것으로 기억한다. 나는 TunnelBear 위치와 계정 상의 위치를 미국으로 설정했다.

Spotify에 가입했으면, [Dashboard](https://developer.spotify.com/dashboard/applications)로 들어가 앱을 생성해야 한다. 생성 후 앱으로 들어가면 아래와 같은 창에서  `Client ID`, `Client Secret`이 보일 것이다. 이 정보가 API 데이터에 접근할 때 사용해야 할 인증 정보이다.

![20200607-2-dashboard](/assets/20200607-2-dashboard.png)

API 이용시 Authorization에 관해서는 크게 두 가지가 있다.

1. App Authorization: 앱을 통해 API에 접근하는 것이다. 서비스를 이용하는 사용자의 정보가 따로 필요하지 않다.
2. User Authorization (OAuth): 서비스를 이용하는 사용자 정보를 통해 API에 접근하는 것이다. 예를 들어 왓챠 같은 앱을 사용하려면 따로 가입할 필요 없이 Google로 로그인 할 수도 있는데, 이 때 이 방식을 사용한다. Google **계정 정보** 를 가지고 **왓챠** 서비스를 이용하는 것이다.

내가 개발하려는 카카오톡 챗봇에서는 사용자의 Spotify 계정 정보 없이 아티스트에 대한 서비스를 제공할 것이다. 따라서 **App Authorization** 방식을 사용했다. 이 방식은 다음 이미지의 순서대로 작동한다.

![auth](https://developer.spotify.com/assets/AuthG_ClientCredentials.png)

1. 카카오톡 사용자가 메시지를 요청하면 Spotify 앱 작동이 시작된다. 이 때 위에서 언급한 Spotify 계정 정보인 `cliend_id`, `cliend_secret`, 그리고 `grant_type`이 필요하다. grant_type은 `client_credentials`로 설정하면 된다.
2. 그러면 Spotify에서 사용자에게 `access_token`을 부여한다. 이 토큰을 가지고 API 데이터를 쿼리하면 된다.

이 과정을 위한 코드는 다음과 같다.

```py
# header 발급 위한 함수
def get_headers(client_id, client_secret):

    endpoint = "https://accounts.spotify.com/api/token"
    # cliend_id, client_secret은 Base 64 encoded string이어야 함
    encoded = base64.b64encode("{}:{}".format(client_id, client_secret).encode('utf-8')).decode('ascii')

    headers = {
        "Authorization": "Basic {}".format(encoded)
    }

    payload = {
        "grant_type": "client_credentials" # grant_type은 고정
    }

    r = requests.post(endpoint, data=payload, headers=headers)

    access_token = json.loads(r.text)['access_token']

    headers = {
        "Authorization": "Bearer {}".format(access_token)
    }

    return headers
```

<br>

---

### Spotify 데이터 종류

이제 데이터를 가져올 수 있다. [Web API Reference](https://developer.spotify.com/documentation/web-api/reference/)에서 아티스트, 앨범, 검색 결과, 트랙 등 여러 종류의 데이터에 대해 나와 있다. 이번 글에서는 다음 두 가지를 사용할 것이다. 코드를 작성하지 않아도, Spotify 로그인 후 각 링크에서 쿼리 결과를 테스트해 볼 수 있다.

1. [Search](https://developer.spotify.com/documentation/web-api/reference/search/search/): 검색어를 입력 결과로 나오는 아티스트 정보 (아티스트 ID 포함)
2. [Top Tracks](https://developer.spotify.com/documentation/web-api/reference/artists/get-artists-top-tracks/): 아티스트 ID를 입력면 나오는 아티스트의 인기 트랙

#### 1. Search

parameter는 다음과 같다. `queen`으로 검색해 보았다.

```py
params = {
      "q": "queen",
      "type": "artist",
      "limit": "1"
  }
```

- `q`는 검색어이다.
- `type`은 album, artist, playlist, track, show, episode 등을 정할 수 있는데, 나는 아티스트가 필요하므로 `artist`로 설정했다.
- `limit`은 검색 결과 개수를 정하는 부분이며, 가장 상위 검색어로 나온 결과를 사용할 것이므로 1로 설정했다.

결과는 다음과 같다.

```json
{
  "artists": {
    "href": "https://api.spotify.com/v1/search?query=queen&type=artist&offset=0&limit=1",
    "items": [
      {
        "external_urls": {
          "spotify": "https://open.spotify.com/artist/1dfeR4HaWDbWqFHLkxsg1d"
        },
        "followers": {
          "href": null,
          "total": 26100989
        },
        "genres": [
          "glam rock",
          "rock"
        ],
        "href": "https://api.spotify.com/v1/artists/1dfeR4HaWDbWqFHLkxsg1d",
        "id": "1dfeR4HaWDbWqFHLkxsg1d",
        "images": [
          {
            "height": 806,
            "url": "https://i.scdn.co/image/b040846ceba13c3e9c125d68389491094e7f2982",
            "width": 999
          },
          {
            "height": 516,
            "url": "https://i.scdn.co/image/af2b8e57f6d7b5d43a616bd1e27ba552cd8bfd42",
            "width": 640
          },
          {
            "height": 161,
            "url": "https://i.scdn.co/image/c06971e9ff81696699b829484e3be165f4e64368",
            "width": 200
          },
          {
            "height": 52,
            "url": "https://i.scdn.co/image/6dd0ffd270903d1884edf9058c49f58b03db893d",
            "width": 64
          }
        ],
        "name": "Queen",
        "popularity": 90,
        "type": "artist",
        "uri": "spotify:artist:1dfeR4HaWDbWqFHLkxsg1d"
      }
    ],
    "limit": 1,
    "next": "https://api.spotify.com/v1/search?query=queen&type=artist&offset=1&limit=1",
    "offset": 0,
    "previous": null,
    "total": 3174
  }
}
```

`artists > items[0]`에 원하는 아티스트의 정보가 담겨져 있다. 이 정보를 `MySQL`에 저장할 것인데, 이 때 `genres`는 위에서 볼 수 있듯이 - ["glam rock", "rock"] - 한 아티스트에 여러 개의 값이 있는 경우가 많다. 장르가 없는 경우도 있다.
아티스트 테이블을 구성할 때 장르 때문에 행이 많아지면 구조가 복잡해지므로, 아래와 같이 테이블을 구성할 것이다. artist 테이블의 `id`와 artist_genres 테이블의 `artist_id`가 join되는 구조이다.

- artist

| 칼럼 | 쿼리 결과에서의 위치 | 설명 | 데이터 타입 |
| :--------- | :------------- | :------------------- | :--------- |
| `id` | `id` | 아티스트 ID | varchar(255) |
| `name` | `name` | 아티스트 이름 | varchar(255) |
| `followers` | `followers > total` | 팔로워 수 | int(11) |
| `popularity` | `popularity` | 인기도 | int(11) |
| `url` | `external_urls > spotify` | Spotify에서 해당 아티스트에 대해 정리된 링크 주소 | varchar(255) |
| `image_url` | `images[0] > url` | 아티스트 썸네일 이미지 주소 (여러 이미지가 나오는데, 첫 번째 결과를 사용할 것) | varchar(255) |

- artist_genres

| 칼럼 | 쿼리 결과에서의 위치 | 설명 | 데이터 타입 |
| :--------- | :------------- | :------------------- | :--------- |
| `artist_id` | `id` | 아티스트 ID | varchar(255) |
| `genre` | `genres` | 각 장르 | varchar(255) |
| `updated_at` | `id` | 추가된 날짜와 시각 | timestamp |

artist_genres 테이블은 아래와 같이 한 아티스트에 장르 개수만큼의 행 개수를 가지는 구조이다.

![20200607-3-artistgenres](/assets/20200607-3-artistgenres.png)

- 물론 `NoSQL`을 사용하면 해결 될 문제이지만, 학습을 위해 여러 종류의 DB를 사용해 보고 있다. 아래에 나올 Top Tracks 데이터는 AWS에서 제공하는 NoSQL인 `DynamoDB`를 사용할 것이다.
- 참고로 실제 Spotify 서비스에서 검색 결과가 어떻게 나오는지 보고 싶다면, [open.spotify.com](https://open.spotify.com/)에서 확인해 볼 수 있다(이것도 Spotify 로그인이 필요하다). 한국 아티스트의 경우 한글로도 검색이 가능하며, 검색어에 따라서 아티스트가 아예 나오지 않을 수도 있다.

<br>

#### 2. Top Tracks

- invoke lambda 함수(top-tracks)를 통해 DynamoDB에 저장





<br>

---

### 데이터를 말풍선 형태에 맞게 보내기

사용자가 `안녕`이라는 메시지를 보내면, 아래의 `json` 형태로 전달된다.

```json
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
    "utterance": "안녕",
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

`userRequest` > `utterance` 가 사용자의 메시지가 담기는 부분이다.



#### 최종 결과

![20200607-1-result](/assets/20200607-1-result.png)
- 각자 음악을 듣는 앱이 모두 다르므로, 범용적인 YouTube 검색 링크를 사용했다.


---
출처
- [응답 타입별 JSON 포맷](https://i.kakao.com/docs/skill-response-format)
- [한국에서 Spotify(스포티파이) 사용하기](https://min7zz.tistory.com/834)
- [Spotify API: Quick Start](https://developer.spotify.com/documentation/web-api/quick-start/)
- [Spotify API: Authorization Guide](https://developer.spotify.com/documentation/general/guides/authorization-guide/)
- [Spotify API: Web API Reference](https://developer.spotify.com/documentation/web-api/reference/)
