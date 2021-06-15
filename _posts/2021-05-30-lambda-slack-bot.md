---
layout: post
title: AWS Lambda로 Slack 사진 업로드 봇 만들기
tags: [crawling, cloud]
categories: [Cloud]
excerpt_separator: <!--more-->
---
AWS Lambda를 활용하여 Slack 사진 업로드 봇을 만들어 보았다.<!--more--> 목차는 아래와 같다.

1. [Lambda Function에 Layer 올리기](#lambda-function에-layer-올리기)
2. [Lambda에서 크롤링 테스트](#lambda에서-크롤링-테스트)
3. [EventBridge로 cron job 생성하기](#eventbridge로-cron-job-생성하기)

[지난 포스팅](https://sulmasulma.github.io/etc/2021/02/17/slack-photo-upload-bot.html)에서 물리적인 서버를 이용하여 리눅스 crontab으로 슬랙 봇을 만드는 과정을 소개했다. 하지만 클라우드 서버는 무료 체험 기간에만 무료로 사용할 수 있고, 그 이후에는 계속 요금이 부과된다. GCP에서 무료로 제공하는 서버가 있지만, 사양이 너무 좋지 않아 `selenium`이 제대로 작동되지 않았다.

그래서 이번 글에서는 월 100만 건까지 무료로 사용할 수 있는 AWS `Lambda`를 이용하여 **서버리스로 슬랙 봇** 을 만드는 과정을 소개한다.

크롤링과 `selenium` 사용에 대해서는 지난 글에서 서술했기 때문에, 이 글에서는 주로 Lambda 환경 세팅에 대해 다루고자 한다. Lambda function 코드는 [GitHub](https://github.com/sulmasulma/forfun/blob/master/lambda/lambda_function.py)에 올려 두었다. 코드 자체는 [지난 코드](https://github.com/sulmasulma/forfun/blob/master/photo_bot_linux.py)와 크게 다르지 않다.

<br>

### Lambda Function에 Layer 올리기

Lambda 환경은 내가 필요한 python 패키지가 모두 설치되어 있지 않다. 그래서 필요한 패키지를 포함하여 코드와 함께 업로드해야 한다. (이에 대해서는 [AWS Lambda에서 외부 라이브러리 사용하기
](https://sulmasulma.github.io/data/2020/06/24/aws-lambda-with-external-library.html)에 서술해 놓았다)

하지만 `Layers`라는 기능이 추가되어, 필요한 패키지 파일을 따로 Layer에 넣어 스크립트 코드에서 Layer 에 있는 패키지를 import 할 수 있게 되었다. 이 부분에 대해서는 [inahjeon's devlog](https://inahjeon.github.io/devlog/side%20project/2020/05/24/news2.html) 글을 참고했다.

<br>

#### Layer 구성하기

Layer를 처음부터 모두 구성할 수도 있지만, 리눅스용 크롬 브라우저와 크롬 드라이버가 모두 들어있는 파일을 사용했다. [https://github.com/inahjeon/AWS-LAMBDA-LAYER-Selenium](https://github.com/inahjeon/AWS-LAMBDA-LAYER-Selenium) 링크로 들어가 `SELENIUM_LAYER.zip` 파일을 다운로드한다.

압축을 해제하고 확인해 보면 `selenium` 패키지 폴더를 확인할 수 있고, `bin` 폴더 안에 크롬, 크롬 드라이버가 모두 들어있다. 추가적으로 필요한 패키지(내 경우는 `slack_sdk`)는 다운로드하여 아래와 같이 폴더째로 추가하면 된다.

![20210530-9-layer](/assets/20210530-9-layer.png)

최종적으로 이 디렉토리를 통째로 다시 압축하여 Lambda Function에 올리면 된다.

<br>

#### Layer를 Lambda Function에 올리기

Lambda 화면으로 돌아가면 아래와 같이 Layers(n) 창이 보일 것이다. 나는 이미 Layer를 추가했기 때문에 Layer(1)로 뜨지만, 추가하기 전에는 Layer(0)으로 뜬다.

![20210530-10-layer2](/assets/20210530-10-layer2.png)

Layer를 만들기 위해, 함수 밖으로 나가 Lambda 메인 메뉴를 보면 좌측에 **계층** 메뉴가 있다. **계층 생성** 을 클릭하고, 위에서 압축한 `.zip` 파일을 업로드한다.

![20210530-11-addlayer](/assets/20210530-11-addlayer.png)

그리고 함수 안으로 다시 돌아가 페이지 하단을 보면 [Add a layer] 메뉴가 있다. 클릭하여 위에서 만든 Layer를 Lambda Function에 추가하면 된다.

![20210530-12-applylayer](/assets/20210530-12-applylayer.png)

- 과정을 보면 알겠지만, 한 번 생성한 Layer는 여러 Lambda 함수에 적용할 수 있다.

<br>

#### lambda_function.py에 Lambda용 chrome, chromedriver 설정하기

Lambda 환경에서는 `/opt/` 폴더에서 binary 파일을 찾아 실행한다. 따라서 파이썬 스크립트에서 아래와 같이 chrome 및 chromedriver 설정을 해 주어야 한다.

```py
from selenium.webdriver.chrome.options import Options

# chrome for lambda layer
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--window-size=1280x1696')
chrome_options.add_argument('--user-data-dir=/tmp/user-data')
chrome_options.add_argument('--hide-scrollbars')
chrome_options.add_argument('--enable-logging')
chrome_options.add_argument('--log-level=0')
chrome_options.add_argument('--v=99')
chrome_options.add_argument('--single-process')
chrome_options.add_argument('--data-path=/tmp/data-path')
chrome_options.add_argument('--ignore-certificate-errors')
chrome_options.add_argument('--homedir=/tmp')
chrome_options.add_argument('--disk-cache-dir=/tmp/cache-dir')
chrome_options.add_argument('user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36')
chrome_options.binary_location = "/opt/python/bin/headless-chromium"

driver = webdriver.Chrome('/opt/python/bin/chromedriver', chrome_options=chrome_options)
```

전체 코드는 [GitHub](https://github.com/sulmasulma/forfun/blob/master/lambda/lambda_function.py)에 올려 놓았다.

<br>

#### 부가적인 설정

Lambda에서 `selenium`을 위해 크롬과 크롬 드라이버를 정상적으로 실행하기 위해서는, 어느 정도의 메모리 용량과 실행 시간이 필요하다. 이를 위해 **구성** 탭의 **일반 구성** 에서 메모리는 1024MB 정도로, 실행 시간은 여유롭게 10분 정도로 잡아 준다.

![20210530-7-configure1](/assets/20210530-7-configure1.png)

또한 **환경 변수** 에서 필요한 환경변수도 설정해 준다.

![20210530-8-configure2](/assets/20210530-8-configure2.png)

<br>

---

### Lambda에서 크롤링 테스트

리눅스 서버는 디스크 저장공간을 포함하고 있기 때문에, 크롤링하여 얻은 이미지를 저장하여 슬랙에 올리는 방식을 사용했었다. 하지만 Lambda는 서버리스 환경으로, 저장공간을 사용하는 방식이 다르다. 코드 실행을 위한 파일(스크립트, 패키지 등)을 저장할 수는 있지만, 스크립트 실행 도중에 파일을 저장하면 다음과 같이 파일을 쓸 수 없다는 에러가 발생한다.

```
[ERROR] OSError: [Errno 30] Read-only file system: 'temp.gif'
```

그래서 Amazon `S3`와 같은 외부 저장소에 이미지를 저장하고 이를 슬랙에 올리는 방법을 고민했었는데, [JG Ahn님 블로그](https://ahnjg.tistory.com/14)를 통해 **Lambda에서도 임시적으로 파일을 올릴 수 있다** 는 사실을 발견했다!

이미지를 스크립트 실행 후 재활용 할 수는 없는 것 같지만, 슬랙에 이미지를 올리려면 스크립트 실행 시에만 파일이 있으면 된다. Lambda에서는 `/tmp/` 폴더에 파일을 저장하여 사용할 수 있다. 그래서 아래와 같이 `/tmp/` 폴더에 저장하여 사진을 업로드했다. (실제 코드와는 다소 차이가 있다)

```py
# 파일 저장
filename = "/tmp/temp.jpg"
headers = {'User-Agent': 'whatever'}
req = Request(src, headers=headers)
html = urlopen(req)
source = html.read()

with open(filename, "wb") as f:
    f.write(source)

# slack에 파일 올리기
upload_file("#channel_name", filename)
```

<br>

#### 실행 테스트

Lambda 함수의 트리거를 설정하지 않아도, 스크립트를 실행해 볼 수 있는 메뉴가 있다. 트리거에 대해서는 밑에서 다시 설명하겠다.

![20210530-5-lambda-test](/assets/20210530-5-lambda-test.png)

**Test** 버튼을 클릭하고 정상적으로 실행이 완료 되었다면, 아래와 같은 결과를 확인할 수 있다.

![20210530-6-lambda-test-result](/assets/20210530-6-lambda-test-result.png)

<br>

---

### EventBridge로 cron job 생성하기

이제 이 명령을 자동화하기 위해, 리눅스 서버를 이용할 때처럼 cron job을 이용해야 한다.

이를 위해 Lambda 함수의 트리거 개념에 대해 알아보자. 트리거는 Lambda 함수 스크립트를 실행하도록 하는 역할을 한다. [카카오톡 챗봇](https://sulmasulma.github.io/data/2020/06/03/kakaotalk-chatbot.html)에서는 아래와 같이 `API Gateway`를 트리거로 이용했다. 사용자가 메시지를 입력하면 API Call을 보내 Lambda 함수가 작동하게 하는 방식이다.

![20210530-1-lambda-kakaobot](/assets/20210530-1-lambda-kakaobot.png)

Lambda 함수로 들어가 **+트리거 추가** 메뉴를 클릭하면, 아래와 같이 API Gateway 말고도 다양한 트리거를 설정할 수 있다.

![20210530-2-add-trigger](/assets/20210530-2-add-trigger.png)

나는 `EventBridge`를 이용하여, 리눅스 crontab처럼 내가 원하는 시간에 주기적으로 명령을 실행할 수 있게 할 것이다. 아래와 같이 설정하면, 매일 평일 8시에 실행되도록 할 수 있다.

![20210530-3-eventbridge](/assets/20210530-3-eventbridge.png)

- 시간은 GMT 기준(한국 시각 - 9시간)으로, 매주 일요일~목요일 23시로 설정해야 한국 시각으로 매주 월요일~금요일 8시에 실행된다.

<br>

설정을 마치면 Lambda 함수 창에서 아래와 같이 트리거가 설정된 모습을 볼 수 있다.

![20210530-4-lambda-slackbot](/assets/20210530-4-lambda-slackbot.png)

<br>

#### Lambda에서 cron을 통해 selenium 사용하기

로컬 환경이나 리눅스 서버에서 `selenium`을 사용할 때에는 `driver.close()` 코드를 통해 크롬 드라이버를 닫아 주었다. 그래야 크롬 드라이버가 제대로 종료되기 때문이다.

Lambda에서 아래와 같이 **Test** 를 통해 실행할 때에도 `driver.close()` 코드를 사용하는 것이 전혀 문제가 되지 않았다.

![20210530-5-lambda-test](/assets/20210530-5-lambda-test.png)

하지만 내가 설정한 시간에 cron job을 실행될 때에는 다음과 같은 에러가 발생했다.

```
WebDriverException: Message: no such session
```

[구글링](https://stackoverflow.com/questions/38065688/webdrivererror-no-such-session-error-using-chromedriver-chrome-through-selenium/40129123#40129123)해 보니, 이전 실행(Test)에서 드라이버를 닫으면 다음에 실행할 때 드라이버를 새로 열리지 않아 위 에러가 발생하는 것으로 보인다. Lambda 코드에서 해당 부분을 없앴더니, cron job에서도 정상적으로 크롤링된다.

Lambda Function 코드에서도 해당 부분을 실행하지 않도록 주석 처리해 놓았다.

```py
# 드라이버 닫으면 cron job 작동이 되지 않음
# driver.close()
```

<br>

---

### 마무리하며

최대한 돈을 내지 않고 슬랙 크롤링&업로드 봇을 만드는 법을 계속해서 찾은 결과였다. NAS든 클라우드든 내 서버가 하나 있어야 편하긴 하다. 하지만 지금 상황에서는 서버가 하루 종일 필요한 것도 아니고 하루에 딱 5분 정도만 필요했기 때문에, 이전에 카카오톡 챗봇에서 사용했던 Lambda를 활용해 보았다.

Lambda는 프리 티어 기간이 끝나도 월 100만 회 + 일정 컴퓨팅 용량 내에서 무료로 사용할 수 있지만, GCP Cloud Function은 월 200만 회까지 무료로 제공하여 범위가 더 넓은 것 같다.

클라우드 업체에서 온갖 서비스 사용에 요금을 부과하지만, **이건 정말 귀한 것 같다..** 서버리스를 최대한 활용하여 일상 생활에 필요한 코딩을 지속적으로 해야겠다.

<br>

---
#### 참고 문서
- [내맘대로 뉴스 요약 큐레이션 (2) - AWS Lambda 로 크롤링 자동화하기](https://inahjeon.github.io/devlog/side%20project/2020/05/24/news2.html)
- [GitHub: inahjeon/AWS-LAMBDA-LAYER-Selenium](https://github.com/inahjeon/AWS-LAMBDA-LAYER-Selenium)
- [AWS Lambda: errno 30 read-only file system](https://ahnjg.tistory.com/14)
- [stackoverflow: webdrivererror-no-such-session-error-using-chromedriver-chrome-through-selenium](https://stackoverflow.com/questions/38065688/webdrivererror-no-such-session-error-using-chromedriver-chrome-through-selenium/40129123#40129123)
