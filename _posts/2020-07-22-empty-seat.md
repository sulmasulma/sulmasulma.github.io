---
layout: post
title: 도서관에 빈 자리 나면 메일로 알림받기
tags: [crawling, mail]
categories: [Etc]
excerpt_separator: <!--more-->
---
도서관 사이트 크롤링하여, 원하는 구역에 빈 자리가 났을 경우 메일로 알림받기<!--more-->

취업준비를 하며 학교 도서관을 이용하고 있다. 코딩을 하거나 이력서를 쓰려면 노트북을 써야 하는데, 정해진 구역에서만 노트북을 사용할 수 있다. 문제는 코로나 때문에 4자리에 1자리 꼴로만 자리가 열려 있기 때문에, 늦게 가면 자리 구하기가 쉽지 않다. 자리 현황을 계속 새로고침하여 보고 있자니 시간 낭비가 커져서, 문득 이런 생각이 들었다.

> 노트북 쓸 수 있는 자리가 나면 알림을 띄울 수 있을까?

맥북에서 알림을 하기엔 생각해야 할 게 너무 많은 것 같아서, 알림 대신 메일을 발송하기로 했다. 어차피 크롬을 보면서 메일 탭을 계속 띄워 놓고 있기 때문이다. 그래서 이번 글에선 자리 현황 사이트를 계속해서 크롤링하여, 내가 원하는 구역에 **자리가 나면 자동으로 메일을 발송** 하는 것에 대해 다루려고 한다. 크롤링의 기초에 대해서는 다루지 않을 것이지만, 전체 코드는 [Github](https://github.com/sulmasulma/forfun/blob/master/seat.py)에 올려 놓았다.

- 크롤링엔 `selenium`, 파싱에는 `BeautifulSoup`을 사용하였다.
- 브라우저는 `Chrome`이다.

목차는 다음과 같다.

1. [자리 사이트 크롤링](#자리-사이트-크롤링)
2. [자동으로 메일 발송하기](#자동으로-메일-발송하기)

<br>

---

### 1. 자리 사이트 크롤링

전체 좌석 중 내가 원하는 구역은 아래 안내도의 왼쪽 노란 구역(노트북 이용)에서 361~368번의 동떨어진 자리를 제외한 자리들이다. (309~360, 369~388)

![20200722-1-map](/assets/20200722-1-map.png)

이 사이트에서 html 태그 구조는 다음과 같다.
- 전체 구조도: `<div id="maptemp">`
  - 배경 이미지: `<div id="backimage">`
  - 자리 1: `<div id="Layer1">`
  - 자리 2: `<div id="Layer2">`
  - 자리 3: `<div id="Layer3">`
  - ...
- `maptemp`라는 id를 가진 `div` 태그 안의 2번째 요소부터 각 자리에 해당하는 `div` 태그들이 들어 있다.

이제 빈 자리(파란색)와 비어 있지 않은 자리(회색)의 차이를 알아 보자.

#### 빈 자리

```html
<div id="Layer2" style="position:absolute; left:928.90625px; top:281.640625px; width:22.65625px; height:19.140625px; z-index:2">
    <table border="0" cellpadding="0" cellspacing="0" height="96%" width="96%">
        <tbody>
            <tr>
                <td align="center" bgcolor="#5AB6CF" valign="middle">
                    <font style="color:black;font-size:10pt;font-weight:300;">155</font>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

- `td` 태그를 보면 배경 색(`bgcolor`)이 파란색(#5AB6CF)이다.

#### 비어 있지 않은 자리

```html
<div id="Layer1" style="position:absolute; left:928.90625px; top:260.44921875px; width:22.65625px; height:19.140625px; z-index:1">
    <table border="0" cellpadding="0" cellspacing="0" height="96%" width="96%">
        <tbody>
            <tr>
                <td align="center" bgcolor="#C9C9C9" valign="middle">
                    <font style="color:black;font-size:10pt;font-weight:300;">154</font>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

- `td` 태그를 보면 배경 색(`bgcolor`)이 회색(#C9C9C9)이다.

공통적으로 `font` 태그 안에 자리 번호가 들어 있다. 따라서 내가 원하는 **자리 번호** (309~360, 369~388)와 **자리 색** (파란색, #5AB6CF)을 만족하는 자리를 수집하면 된다.

```py
from selenium import webdriver
from bs4 import BeautifulSoup

driver = webdriver.Chrome('../chromedriver')
url = '사이트 url'
driver.get(url)

html = driver.find_element_by_id('maptemp').get_attribute('innerHTML') # 전체 구조도
soup = BeautifulSoup(html, 'html.parser')
data = [] # 원하는 자리 담기

for ele in soup.find_all('div')[1:]: # 1번째부터 각 자리를 나타내는 div 태그가 있음
    d = ele.table.tbody.tr.td # 번호와 색 공통으로 해당하는 부분으로 들어감
    num = int(d.font.get_text()) # 자리 번호는 태그의 값
    color = d.get('bgcolor') # 자리 색은 태그의 변수

    # 원하는 자리 났을 경우, 리스트에 자리와 색 넣기
    if ((num >= 309 and num <= 360) or num >= 369) and color == "#5AB6CF":
        data.append([num, color])
```

자리가 났을 경우 수집되는 데이터(위에서 `data`)는 아래와 같은 구조이다.

```py
[[309, '#5AB6CF'],
 [312, '#5AB6CF'],

 ...,

 [385, '#5AB6CF'],
 [388, '#5AB6CF']]
```

<br>

---

### 2. 자동으로 메일 발송하기

이 부분은 전적으로 [아무튼 워라밸 블로그](http://hleecaster.com/python-email-automation/) 글을 참고하였다. *좋은 글 감사합니다.*

#### MIME

이메일을 보낼 때는 MIME(Multipurpose Internet Mail Extensions)이라는 '전자우편 표준 형식'을 따라야 한다. 파이썬에서는 `email` 라이브러리를 통해 사용할 수 있다.

<br>

#### SMTP

SMTP(Simple Mail Transfer Protocol)는 이메일을 주고 받는 ''표준적인 약속 체계나 규약'이다. 메일을 보내려면 **SMTP 서버 주소** 와 **포트 번호** 가 필요하다. 파이썬에서는 `smtplib` 라이브러리를 통해 구현할 수 있다. email과 smtplib 모두 파이썬 표준 라이브러리에 해당하여, 별도 설치할 필요가 없다.

Gmail의 경우 서버 주소는 `smtp.gmail.com`, 포트 번호는 `465`이다. SMTP 서버를 통하여 이메일 발신자의 주소와 비밀번호, 수신자 주소, 메일 내용을 요청하면 된다.
- 비밀번호을 정확히 입력했는데도 파이썬에서 오류가 날 수 있다. 발신자의 구글 계정에서 적절한 보안 설정이 필요하기 때문이다.
- 이와 관련하여 [Gmail 고객센터 공식 문서](https://support.google.com/mail/answer/7126229?p=BadCredentials&visit_id=637309029748019312-4234417840&rd=2#cantsignin)에 자세히 나와 있다. 구글 계정에서 2단계 인증을 사용하는 경우 앱 비밀번호로 로그인해야 하며, 2단계 인증을 사용하지 않으면 보안 수준이 낮은 앱이 계정에 액세스하도록 허용해야 한다.

<br>

#### 메일 발송하기

(Gmail 기준) 위에서 수집한 자리 데이터를 메일로 발송하는 코드를 작성하면 된다. 코드의 핵심 부분만 소개한다.

```py
import smtplib, os, pickle # smtplib: 메일 전송을 위한 패키지
from email import encoders # 파일전송을 할 때 이미지나 문서 동영상 등의 파일을 문자열로 변환할 때 사용할 패키지
from email.mime.text import MIMEText # 본문내용을 전송할 때 사용되는 모듈
from email.mime.multipart import MIMEMultipart # 메시지를 보낼 때 메시지에 대한 모듈
from email.mime.base import MIMEBase # 파일을 전송할 때 사용되는 모듈

# SMTP 접속을 위한 서버, 계정 설정
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 465

# 발신자 계정, 비밀번호 (Gmail)
SMTP_USER = "발신자 주소"
SMTP_PASSWORD = pickle.load(open('pw.pickle', 'rb')) # 보안을 위해 pickle 파일로 저장하여 가져오면 좋음

# 수신자 주소
addr = "수신자 주소"

msg = MIMEMultipart("alternative")
msg["From"] = SMTP_USER # 발신자
msg["To"] = addr # 수신자
msg["Subject"] = '메일 제목' # 메일 제목

# 메일 내용(str 형식): 자리 데이터
# 리스트 [309, 312, ..., 385, 388] -> 문자열 "309 312 ... 385 388"로 변환
text = MIMEText(_text = ' '.join([str(d[0]) for d in data]), _charset = "utf-8")
msg.attach(text)

# smtp로 접속할 서버 정보를 가진 클래스변수 생성
smtp = smtplib.SMTP_SSL(SMTP_SERVER, SMTP_PORT)
# 해당 서버로 로그인
smtp.login(SMTP_USER, SMTP_PASSWORD)
# 메일 발송
smtp.sendmail(SMTP_USER, addr, msg.as_string())
# 닫기
smtp.close()
```

코드를 실행하면, 다음과 같은 메일을 받을 수 있다. 나는 발신자와 수신자가 같도록 하여, 내가 나에게 메일을 발송했다.

![20200722-2-mailresult](/assets/20200722-2-mailresult.png)

<br>

---

### 마무리하며

귀찮음을 해결하기 위해 몇 시간 들여 코딩해 보았다. 도서관이 열리자마자 가서 자리를 잘 차지하면 좋겠지만, 나의 게으름으로 인해(...) 늦게 가서 자리를 차지하지 못했을 경우 유용하게 활용해야겠다.


<br>

---
#### 참고 문서
- [파이썬 이메일 자동화 (이메일 대량 발송)](http://hleecaster.com/python-email-automation/)
