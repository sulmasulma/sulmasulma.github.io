---
layout: post
title: Slack 사진 업로드 봇 만들기
tags: [crawling, cloud]
categories: [Etc]
excerpt_separator: <!--more-->
---
구글에서 사진을 크롤링하여 슬랙에 자동으로 올리는 봇을 만들어 보았다.<!--more--> 목차는 아래와 같다.

1. [사진 크롤링](#사진-크롤링)
2. [Slack에 사진 업로드](#slack에-사진-업로드)
3. [리눅스에서-실행하기](#리눅스에서-실행하기)
4. [crontab으로 자동화하기](#crontab으로-자동화하기)

이번 글에서 사용한 툴 및 환경은 다음과 같다.
- Mac OS: 10.15 `Catalina`
- Linux: Amazon `EC2`
- 언어: `Python`
- 크롤링 라이브러리: `selenium`
- [Slack API](https://api.slack.com/)

참고로, 이 글에서는 코드의 주요 부분만 소개하고자 한다. 전체 코드는 [GitHub](https://github.com/sulmasulma/forfun/blob/master/photo_bot.py)에 올려 두었다. 크롤링 부분은 `scrap_photo_google` 함수를 보면 된다.

<br>

### 사진 크롤링

먼저 Mac OS(또는 Windows)에서 사진을 크롤링한다. 구글 검색창은 기본적으로 동적 페이지이다. 페이지를 조작하며 크롤링해야 하기 때문에, `selenium`을 사용하였다.

크롤링시 검색창 썸네일 이미지를 그대로 가져오면, 화질이 매우 떨어진다.

![20210217-1-thumbnail](/assets/20210217-1-thumbnail.png)

그래서 **사진을 클릭** 하면 우측에 나오는 확대된 이미지를 가져와야 화질 손실을 줄일 수 있다. 원본 이미지보다는 화질이 다소 떨어지지만, 검색 창에서 원본 이미지 링크를 직접 제공하진 않아 이 방법이 최선인 것 같다.

![20210217-2-rawimage](/assets/20210217-2-rawimage.png)

<br>

#### 크롤링 시작하기

[chromedriver 홈페이지](https://chromedriver.chromium.org/downloads)에서 자신의 chrome 버전과 맞는 드라이버를 설치한 후, chromedriver를 실행한다.

```py
from selenium import webdriver
driver = webdriver.Chrome('chromedriver 위치')

keyword = '아린' # 검색어
# 고화질(800x600보다 큰 이미지) + 최근 1달로 검색하는 url
url = "https://www.google.com/search?q={}&tbm=isch&hl=ko&safe=images&tbs=qdr:m%2Cisz:lt%2Cislt:svga".format(keyword)
driver.get(url)
```

검색 결과에서 내가 원하는 사진을 **클릭해야 한다.** 우선 이미지 목록을 가져온다. 이미지들은 공통적으로 클래스명이 `rg_i`로 시작하기 때문에, 아래와 같이 가져올 수 있다.

```py
photo_list = driver.find_elements_by_css_selector('img.rg_i')
```

이 중 내가 원하는 이미지를 클릭한다.

```py
img = photo_list[idx] # idx: 검색 결과에서 내가 원하는 이미지의 순서(첫 번째일 경우 0)
img.click()
```

확대된 이미지는 id가 `Sva75c`인 태그 안에 저장되어 있다. 확대된 이미지의 주소가 저장된 src 값을 찾는다.

```py
html_objects = driver.find_element_by_xpath('//*[@id="Sva75c"]/div/div/div[3]/div[2]/c-wiz/div/div[1]/div[1]/div/div[2]/a/img')
src = html_objects.get_attribute('src')
```

추출한 링크를 이용하여 이미지를 다운받는다.

```py
from urllib.request import urlretrieve

urlretrieve(src, 'filename')
```

- 참고로,  filename은 gif 파일도 `~~.jpg` 형식으로 하면 움짤 형태로 저장이 된다.

<br>

---

### Slack에 사진 업로드

슬랙에 자동으로 메시지를 쓰거나 파일을 업로드하려면, [Slack API](https://api.slack.com/)를 이용해야 한다. 앱을 만들고 자신의 Workspace에 추가하면 되는데, 이에 대해서는 [관련 블로그](https://twojobui.tistory.com/12)에 자세히 정리되어 있다.

아래와 같이 앱 생성 후 **Install App** 메뉴로 들어가면, 봇을 사용하기 위한 OAuth Token을 확인할 수 있다.

![20210217-3-slackbottoken](/assets/20210217-3-slackbottoken.png)

이제 이 값을 환경변수로 설정하고(`SLACK_BOT_TOKEN`), 코드에 적용한다. Python으로는 아래와 같이 환경변수를 불러올 수 있다.

```py
slack_bot_token = os.environ.get("SLACK_BOT_TOKEN")
```

앱 생성은 완료되었는데, 실제 슬랙 메신저에 텍스트를 쓰거나 파일을 올리려면 [scope](https://api.slack.com/scopes)를 설정해야 한다. 파일을 올리는 메소드는 `files:write` scope를 필요로 한다.

앱 설정 창으로 다시 돌아가, 왼쪽 메뉴 중 `OAuth & Permissions`에서 내 앱의 scope들을 지정할 수 있다. 나는 아래와 같이 테스트 용으로 파일 업로드에 필요한 `files:write`를 포함하여 수많은 scope를 넣었다.

![20210217-4-scope](/assets/20210217-4-scope.png)

이제 Python에서 슬랙 sdk를 사용할 준비를 해 준다.

```py
import logging
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError

# logger
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# slack client
slack_bot_token = os.environ.get("SLACK_BOT_TOKEN")
client = WebClient(token=slack_bot_token)
```
<br>

파일을 업로드하기 위한 코드는 아래와 같다. Slack API 중 [files.upload 메소드](https://api.slack.com/methods/files.upload)를 사용한다.

```py
# 올릴 파일 이름
file_name = "./myFileName.gif"
# 파일 업로드할 채널 id
# "#채널명" 을 넣어줘도 된다.
channel_id = "C12345"

try:
    # Call the files.upload method using the WebClient
    # Uploading files requires the `files:write` scope
    result = client.files_upload(
        channels=channel_id,
        initial_comment="Here's my file :smile:", # 파일과 함께 넣을 텍스트
        file=file_name,
    )
    # Log the result
    logger.info(result)

except SlackApiError as e:
    logger.error("Error uploading file: {}".format(e))
```

<br>

참고로, 채널에 파일을 올리려면 반드시 **봇을 채널에 초대해야 한다!!** 그렇지 않으면 계속해서 `not_in_channel` 에러가 뜨는데, 해결 방법을 몰라서 계속 헤매다가 스택오버플로우 글을 겨우 찾아서 해결했다..

![20210217-5-slack_channel_invite](/assets/20210217-5-slack_channel_invite.png)


<br>

---

### 리눅스에서 실행하기

리눅스 환경은 `Ubuntu`, `centos`, 클라우드 가상머신(GCP `VM 인스턴스`, Amazon `EC2`) 등 여러 가지가 있을 수 있다. 나는 Amazon EC2를 이용했기 때문에 이에 기반하여 적겠다.

기본적으로 리눅스에서는 headless 브라우저를 사용한다. display가 있지 않은 환경이기 때문이다. chromedriver를 이용하면 display 없이도 크롤러를 개발할 수 있다. 이를 위해 리눅스용 chrome 및 chromedriver를 설치한다.

<br>

#### 1. 리눅스용 chrome 설치하기

```sh
sudo curl https://intoli.com/install-google-chrome.sh | bash
sudo mv /usr/bin/google-chrome-stable /usr/bin/google-chrome
google-chrome –version && which google-chrome
```

`google-chrome –version` 명령을 통해 설치된 chrome 버전을 확인한다. 나는 `Google Chrome 88.0.4324.150`으로, 88.0 버전을 받았다. 이 버전과 일치하는 chromedriver를 설치해야 한다.

<br>

#### 2. chromedriver 설치하기

```sh
cd /tmp/
sudo wget https://chromedriver.storage.googleapis.com/88.0.4324.27/chromedriver_linux64.zip
sudo unzip chromedriver_linux64.zip
sudo mv chromedriver /usr/bin/chromedriver
chromedriver –version
```

참고로 버전명은 [http://chromedriver.storage.googleapis.com](http://chromedriver.storage.googleapis.com/)에서 확인할 수 있다. 나는 리눅스용 88.0 버전에 해당하는 `88.0.4324.27` chromedriver를 받았다.

<br>

#### 3. 리눅스용 코드 작성

참고로 리눅스용 `.py` 파일에서 한글 주석을 사용하려면 코드 최상단에 `# -*- coding: utf-8 -*-`을 추가해야 한다.

리눅스는 Mac OS/Windows와 다르게, 눈에 보이지 않는 headless 브라우저를 사용한다. 그래서 코드를 실행해도 chrome 창이 뜨지 않고 실행될 것이다.

그리고 headless 브라우저를 실행하려면 python 코드에 아래 부분을 추가해 주어야 한다. chromedriver도 Mac OS/Windows와 같이 파일 위치를 지정해 주지 않고, 시스템에 등록한 것을 사용한다.

```py
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument("--headless")
options.add_argument("window-size=1400,1500")
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")
options.add_argument("start-maximized")
options.add_argument("enable-automation")
options.add_argument("--disable-infobars")
options.add_argument("--disable-dev-shm-usage")

driver = webdriver.Chrome(options=options)
```

기존 코드에 위 부분만 추가하면 그대로 실행할 수 있다. 리눅스용 코드 역시 [GitHub](https://github.com/sulmasulma/forfun/blob/master/photo_bot_linux.py)에 올려 두었다.

<br>

---

### crontab으로 자동화하기

봇이 기능을 하려면 리눅스 `crontab`을 이용해 자동화해야 한다. crontab에 관련해서는 [Amazon EC2와 리눅스 crontab을 이용한 배치 처리](https://sulmasulma.github.io/data/2020/07/09/ec2-crontab.html)에 정리해 놓았다.

#### crontab 용 환경변수 설정

위에서 Mac OS용 환경변수(`SLACK_BOT_TOKEN`)를 설정해 주었다. 리눅스용 환경변수도 당연히 설정해야 한다. 하지만 `crontab`을 사용하기 위해서는 별도의 환경변수를 설정해 주어야 한다.

여러 방법이 있지만, 나는 `crontab -e`를 실행하면 나오는 vim 편집기에 crontab 목록과 함께 환경변수를 지정했다.

```sh
SLACK_BOT_TOKEN=xoxb-~~~~

00 23 * * 0-4 /usr/bin/python3 /home/ec2-user/photo_bot_linux.py
```

매일 평일 오전 8시(GMT 기준 일~목요일 23시 00분)에 자동으로 스크립트를 실행하기 위해, 위와 같이 crontab 명령을 작성했다.

<br>

#### 최종 확인

아래와 같이 파일이 정상적으로 업로드되는 것을 확인할 수 있다. 위에서도 언급했듯이 `gif` 파일을 `.jpg`로 저장해도 움짤이 정상적으로 저장 및 업로드된다.

![20210217-6-test](/assets/20210217-6-test.png)

<br>

---

### 마무리하며

별거 아닌 것 같았지만 문제들이 많았다. 크롤링 시 고화질 사진을 src로 가져오기, 슬랙 API 이용하기, 리눅스 환경별로 명령어가 다른 등등...

취업 후 첫 코딩이었다는 것에 의미를 두며 마친다. 그동안 보여주기 식으로 이런저런 포스팅을 많이 작성했었는데, 취업하니 매우 귀찮아진 것이 사실이다. 다음에는 좀 더 뼈와 살이 되는 기술적인 글을 올려야겠다.

여기까지 읽어주셔서 감사합니다. :)


<br>

---
#### 참고 문서
- [Google 검색 결과에서 원본 이미지 저장하기](https://hanryang1125.tistory.com/5)
- [Slack App 만들기](https://twojobui.tistory.com/12)
- [EC2에 Chrome 및 ChromeDriver 설치](https://understandingdata.com/install-google-chrome-selenium-ec2-aws/)
- [Linux에서 ChromeDriver 삭제](https://stackoverflow.com/questions/57570005/delete-chromedriver-from-ubuntu)
- [crontab 편집시 shell 환경변수](https://brainio.tistory.com/1)
