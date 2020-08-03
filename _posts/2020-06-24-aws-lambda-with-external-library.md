---
layout: post
title: AWS Lambda에서 외부 라이브러리 사용하기
tags: [AWS, lambda]
categories: [Data]
excerpt_separator: <!--more-->
---
<!--more-->
Lambda 환경은 로컬 환경과 다르다. 로컬에서 `numpy`, `pandas` 등의 라이브러리를 설치했다고 해도, [Lambda 환경](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html){:target="_blank"}에는 영향이 없다. Lambda에서 외부 라이브러리를 사용하려면, 아래 두 방법 중 하나를 선택하면 된다.

1. 코드 및 라이브러리 파일을 `.zip` 파일로 압축하여 Lambda에 업로드
2. 코드 및 라이브러리 파일을 `.zip` 파일로 압축하여 `Amazon S3`에 업로드하여 Lambda에서 사용

나는 두 번째 방법인 S3를 이용할 것이다.
- *참고로, 외부 라이브러리 설정 없이 python의 [표준 라이브러리](https://docs.python.org/ko/3/library/index.html)를 모두 사용 가능하다고 생각했는데 `requests` 패키지는 설정이 필요하다. 디폴트 Lambda 환경에서는 제한적인 라이브러리만 포함되어 있는 것으로 보인다.*

<br>

---

### 1. S3 버킷 생성

[S3](https://s3.console.aws.amazon.com/s3/home)에 들어가 버킷을 생성한다. 버킷은 일종의 폴더라고 생각하면 된다.

![20200624-1-createbucket](/assets/20200624-1-createbucket.png)

버킷 만들기 창으로 들어왔으면, 이름을 입력한다. 다른 사람의 버킷을 포함해서 모든 버킷의 이름은 고유해야 한다. **다음** 으로 넘어가기 전에 아래와 같이 버킷 이름이 이미 있다고 나오면, 고유한 이름으로 수정해 준다.

![20200624-2-bucketname](/assets/20200624-2-bucketname.png)

권한 설정 단계에서는 S3 파일에 다른 사람이 임의로 접근할 수 없도록 **모든 퍼블릭 액세스 차단** 설정에 체크해 준다.

![20200624-3-bucketaccess](/assets/20200624-3-bucketaccess.png)

마지막 검토 단계까지 진행하여 버킷 만들기를 누르면, S3 페이지에서 내 버킷 목록을 확인할 수 있다.

![20200624-4-bucketlist](/assets/20200624-4-bucketlist.png)

<br>

---

### 2. S3에 파일을 업로드하여 Lambda에서 사용하기

#### 코드 파일 저장

[카카오톡 챗봇 만들기 1편](https://sulmasulma.github.io/data/2020/06/03/kakaotalk-chatbot.html)에서는 Lambda 페이지에서 인라인 코드를 작성하여 외부 라이브러리 없이 사용했다. 이번에는 인라인 코드 편집 대신 S3를 이용할 것이므로, 로컬에서 코드를 작성하여 `lambda_function.py` 파일로 저장한다.

<br>

#### 라이브러리 저장

라이브러리는 로컬 환경에 설치하는 것이 아니고, Lambda에 업로드 용으로 별도의 폴더에 라이브러리를 저장할 것이다. 내가 사용한 외부 라이브러리는 `pymysql`과 `boto3`이므로, `requirements.txt`라는 파일을 만들어 아래와 같이 작성한다.

```
pymysql
boto3
```

이제 현재 폴더에서 터미널을 열어 `pip3 install -r requirements.txt -t ./libs/`를 입력한다. 하위 폴더로 `libs/`를 만들고 그 안에 위의 두 라이브러리를 설치하는 것이다.

그리고 `lambda_function.py` 코드의 가장 윗부분에 다음과 같이 입력한다.

```py
import sys
sys.path.append('./libs')
```

- 이렇게 해야 `libs/` 폴더에 설치한 외부 라이브러리를 사용할 수 있다.

<br>

#### S3에 업로드

`deploy.sh` 파일을 생성하여 아래와 같이 작성한다. `kakao.zip` 파일로 압축하여 S3의 `s3://spotify-lambda-matt/kakao.zip`(spotify-lambda-matt은 위에서 생성한 S3 버킷 이름)에 복사하고, Lambda 함수 `spotify-kakao`에 업데이트하는 과정이다. 로컬 폴더와 S3에 이미 있던 `kakao.zip` 파일을 삭제하고 새 파일을 업로드하여, 반복적으로 사용 가능하다.

```sh
#!/bin/bash

rm kakao.zip
zip kakao.zip -r *

aws s3 rm s3://spotify-lambda-matt/kakao.zip
aws s3 cp kakao.zip s3://spotify-lambda-matt/kakao.zip
aws lambda update-function-code --function-name spotify-kakao --s3-bucket spotify-lambda-matt --s3-key kakao.zip
```

여기까지 하면 폴더는 아래와 같이 구성된다.
- `lambda_function.py`: Lambda 함수가 실행될 코드
- `requirements.txt`: 필요한 외부 라이브러리 목록
- `./libs/`: 외부 라이브러리가 들어 있는 폴더
- `deploy.sh`: 명령어

이제 터미널로 돌아와 `./deploy.sh`를 실행한다. 진행 과정이 쭉 나오고, 마지막에 아래와 같이 출력되면 성공적으로 S3 및 Lambda에 업데이트된 것이다.

```
{
    "FunctionName": "spotify-kakao",
    ...
    "State": "Active",
    "LastUpdateStatus": "Successful"
}
```

- `lambda_function.py` 코드와 라이브러리 목록을 수정할 때마다 **반복적으로 이 커맨드를 실행** 하여 업데이트하면 된다.


<br>

이제 Lambda 페이지로 돌아가면, 코드 외에 필요한 파일들이 함께 업로드된 것을 확인할 수 있다. [공식 문서](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/python-package.html)에 따르면 파일들의 총 크기가 3MB 이하라면 인라인 코드 편집이 가능하지만, 3MB를 초과할 경우 아래와 같이 비활성화된다.

![20200624-5-codeineditable](/assets/20200624-5-codeineditable.png)

<br>
