---
layout: post
title: Google Cloud Tasks 써 보기
tags: [cloud, google, api, distribution]
categories: [Data]
excerpt_separator: <!--more-->
---
클라우드 상에서 API 쿼리를 분산처리하는 방법 중 하나인 [Google Cloud Tasks](https://cloud.google.com/tasks/docs/dual-overview)를 소개한다.<!--more-->

광고회사에서 데이터 분석을 할 때, YouTube Analytics API에서 데이터를 일별/영상별로 쿼리하여 2년 간의 데이터를 수집해야 하는 경우가 있었다. 채널의 영상이 200개 정도 되었던 것 같은데, `730(2년) * 200(영상 개수) = 144,000`번의 쿼리를 요청해야 모든 데이터 수집이 가능했다. Google 계정 인증 후 `access_token`을 발급하여 쿼리를 요청해야 했는데, 문제는 이 token이 1시간마다 만료된다는 것이었다. 144,000번의 쿼리를 요청해서 데이터를 받는 데 예상 시간이 10시간은 족히 넘었던 것으로 기억한다. 그래서 로컬 머신에서 이 작업을 하려면 수십 시간동안 `access_token`을 재발급해가며 계속해서 쿼리 요청을 해야 하는데, 503 에러가 나서 도중에 멈출 우려도 있었다. 결국 작업을 포기하고 해당 데이터 없이 프로젝트를 진행했다. 데이터 엔지니어링 공부를 시작하게 된 계기이기도 하다.

서론이 길었는데, 오늘 내용은 이렇게 로컬 머신에서 반복해서 API 쿼리를 요청하는 대신 **Google Cloud에서 분산처리** 를 통해 데이터를 저장하는 것이다. 애플리케이션 외부에서 독립적인 태스크들을 비동기식으로 처리하기 때문에 같은 작업을 반복하는 것보다 작업 시간을 큰 폭으로 줄일 수 있는 내용이다.

각 태스크에서는 API 서버에서 데이터를 받아 `MongoDB`에 쌓고, 최종적으로 쌓인 데이터는 `BigQuery`에 적재할 것이다. 이유는 다음과 같다.
- `MongoDB`는 document 단위로 데이터가 적재되며, 동시다발적인 레코드 추가 속도가 빠른 편
- `BigQuery`는 대용량 데이터 Load에 최적화된 Data Warehouse이기 때문에, 여러 레코드 추가보다는 하나의 큰 데이터에 적합함

Cloud Tasks는 기본적으로 태스크들이 클라우드 상의 큐(Queue)에 추가되고, 태스크가 성공적으로 실행될 때까지 큐가 태스크를 지속하는 구조이다. Cloud Tasks 개요에 대해서는 [Google Cloud 공식 문서](https://cloud.google.com/tasks/docs/dual-overview)에 정리가 잘 되어 있어서, 해당 글을 보는 것을 추천한다. **이 글에서는 실제 사용 방법에 대해서** 소개하고자 한다.

목차는 아래와 같다.

1. [Google Cloud Platform에서 Cloud Tasks 설정](#google-cloud-platform에서-cloud-tasks-설정)
2. [HTTP Target Task 만들기](http-target-task-만들기)
3. [MongoDB Atlas 계정 및 Cluster 생성](mongodb-atlas-계정-및-cluster-생성)
4. [API 데이터 요청, MongoDB 및 GCP에서 상태 확인]()
5. [최종 데이터 확인, BigQuery에 Load]()

- ~~실제 데이터 요청시 로컬 머신 vs Cloud Tasks 소요 시간까지 비교해 보기~~

### Google Cloud Platform에서 Cloud Tasks 설정

먼저 [Google Cloud Platform](https://cloud.google.com/) (이하 GCP)에 가입해야 한다.

![스크린샷 2020-05-18 오후 4.07.09](https://i.imgur.com/d0FkWh4.jpg)

무료료 시작하기 버튼을 눌러 진행하면 되는데, 문제는 가입 진행 창에서 '계속' 버튼을 눌러도 진행이 되지 않는다. (글을 읽는 시점에서는 이 문제가 고쳐졌을 수도 있다.)<br>
이 문제에 대해서는 [GCP 가입 및 가입 오류 해결, 인스턴스 생성하기](https://private-space.tistory.com/41) 글을 참고하면 된다. 이 글에서는 EC2 (클라우드 Compute Engine) 인스턴스 생성에 대해서 다루고 있는데, 여기서는 Cloud Tasks를 사용할 것이므로 아래와 같이 생긴 대시보드 창만 확인하고 넘어가면 된다.

![스크린샷 2020-05-18 오후 4.13.16](https://i.imgur.com/YlVArXn.png)

이제 [Cloud Tasks 큐 빠른 시작](https://cloud.google.com/tasks/docs/quickstart-appengine) 문서를 따라 진행해야 한다. **Cloud Tasks 큐 만들기** 단계까지만 진행하면 된다. 그 밑에 **Cloud Tasks 큐에 태스크 추가** 단계도 있는데, 다른 태스크(API 쿼리 요청)를 추가할 것이므로 일단 넘어가도 된다.

먼저 App Engine 페이지로 이동하여 프로젝트를 만들고, 애플리케이션의 리전을 선택하면 된다. 어느 리전을 선택해도 큰 상관은 없는데, 필자는 `us-central`(이 경우 Cloud Tasks 명령어에서는 `us-central1`으로 사용할 수 있다)을 선택했다.

**시작하기** 버튼이 나오면 언어와 환경 설정이 나오는데, 이 글에서는 `python`으로 실습할 것이므로 python을 선택하고(물론 선호하는 언어가 있다면 변경 가능), `표준` Environment를 선택한다.

다음은 Cloud SDK를 다운로드하고 컴퓨터 환경에 적용하는 과정이다. [연결되는 링크](https://cloud.google.com/sdk/docs/quickstarts)에서 OS에 맞게 진행하면 된다.
- 터미널에서 `gcloud` 명령어를 실행하려면, `pip install gcloud` 설치 후 터미널을 재시작하면 된다. 이후 `gcloud init`으로 문서에 맞게 자신의 계정 정보를 입력하면 된다.

4-a 단계에서는 서비스 계정을 만들어야 한다.
- [서비스 계정 만들기 페이지](https://cloud.google.com/docs/authentication/getting-started#creating_a_service_account)에서 아래와 같은 코드를 실행하라고 나오는데, `pip install google-cloud-storage` 설치 후 진행하면 된다.

```py
def implicit():
    from google.cloud import storage

    # If you don't specify credentials when constructing the client, the
    # client library will look for credentials in the environment.
    storage_client = storage.Client()

    # Make an authenticated API request
    buckets = list(storage_client.list_buckets())
    print(buckets)

implicit()
```

실행하면 아래와 같이 출력될 것이다.

```py
[<Bucket: {프로젝트 ID}.appspot.com>, <Bucket: staging.{프로젝트 ID}.appspot.com>, <Bucket: us.artifacts.{프로젝트 ID}.appspot.com>]
```

중간 과정들을 많이 생략했는데, 추후 보완하겠다. **Cloud Tasks 큐 만들기** 단계에서 생성한 큐의 출력까지 확인했으면 다음으로 넘어가면 된다.
<br>
<br>

### HTTP Target Task 만들기

각 작업을 실행하는 `태스크 핸들러`를 생성하는 과정이다. API 쿼리를 300번 요청한다고 하면, 300개의 독립적인 태스크를 만들어 Google Cloud에서 컨트롤할 수 있도록 매핑해 주어야 한다.

먼저 GCP [Cloud Functions](https://console.cloud.google.com/functions/)에서 **함수 만들기** 로 들어가 함수를 생성한다.

함수 이름은 자유롭게, 할당 메모리는 기본값인 256MiB, 트리거 역시 기본값인 HTTP를 선택한다. 이 때 주의해야 할 점은, **인증되지 않은 호출 허용 옵션** 을 설정하여 추가적인 인증 없이 빠르게 태스크를 진행할 수 있게 한다.

![스크린샷 2020-05-18 오후 5.11.07](https://i.imgur.com/gc8ZMI5.png)

소스 코드는 인라인 편집기, 런타임은 `Python 3.7`, 실행할 함수는 `hello_world`로 그대로 둔다.

cloud function을 생성했으면, 생성한 함수와의 dispatch 작업을 위해 아래 코드를 실행한다. 몇몇 부분을 본인의 설정 사항에 따라서 변경해야 한다.
- `dispatch_task` 함수
  - `project`, `queue`, `location`에는 위의 gcloud 세팅 단계에서 설정한 값을 넣어주면 된다.
  - API 쿼리 시 들어갈 parameter들은 `payload`에 넣어주면 된다. 이 parameter를 Cloud Function에서는 `json`이 아닌 byte array로만 받기 때문에, 반드시 `str` 타입으로 변형해 주어야 한다.
- `create_task` 함수
  - Cloud Tasks에서는 반드시 `POST` 방식으로 요청해야 한다.
  - `url`은 본인의 cluod function 주소이다. GCP에서 해당 함수로 들어가면 **트리거** 탭의 URL에서 확인할 수 있다.

```py
import datetime
from google.cloud import tasks_v2
from google.protobuf import timestamp_pb2

def dispatch_task(name):

    payload = str({
        "name": name
    })

    resp = create_task(project='프로젝트 이름', queue='my-queue', location='us-central1', payload=payload)

def create_task(project, queue, location, payload=None, in_seconds=None):

    task_client = tasks_v2.CloudTasksClient()
    parent = task_client.queue_path(project, location, queue)

    # Construct the request body.
    task = {
            'http_request': {  # Specify the type of request.
                'http_method': 'POST',
                'url': 'cloud function URL'
            }
    }
    if payload is not None:
        # The API expects a payload of type bytes.
        converted_payload = payload.encode()

        # Add the payload to the request.
        task['http_request']['body'] = converted_payload

    if in_seconds is not None:
        # Convert "seconds from now" into an rfc3339 datetime string.
        d = datetime.datetime.utcnow() + datetime.timedelta(seconds=in_seconds)

        # Create Timestamp protobuf.
        timestamp = timestamp_pb2.Timestamp()
        timestamp.FromDatetime(d)

        # Add the timestamp to the tasks.
        task['schedule_time'] = timestamp

    # Use the client to build and send the task.
    response = task_client.create_task(parent, task)

    return response
```

코드 실행 후 `credential error`가 발생한다면, GCP credential 처리를 해 주지 않은 것이다. 이에 대해서는 [Google Application Default Credentials 사용하기](https://jungwoon.github.io/google%20cloud/2018/01/11/Google-Application-Default-Credential/)를 참조하여 자신의 보안 키 값이 들어있는 `json` 파일을 로컬에 다운받고 매핑시켜주면 된다. 다음 중 하나의 방법을 선택해서 진행하면 된다.

1. python에서 아래 코드를 실행하여 일시적으로 환경변수로 키 파일을 참조하도록 처리
  ```py
  import os
  os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = '키 파일 주소'
  ```
2. 환경변수 설정
  - 환경변수 `GOOGLE_APPLICATION_CREDENTIALS`로 키 파일을 설정하는 과정이다.
  - OS에 따라 다른데, Mac OS의 경우는 다음과 같다.
  - `vi ~/.bash_profile`
  - 빈 곳에 `export GOOGLE_APPLICATION_CREDENTIALS=키 파일 주소` 입력
  - `source ~/.bash_profile`

이제 위 코드를 다시 실행하면 된다.
<br>
<br>

### MongoDB Atlas 계정 및 Cluster 생성


---
출처
- [Cloud Tasks 개요](https://cloud.google.com/tasks/docs/dual-overview)
- [GCP 가입 및 가입 오류 해결, 인스턴스 생성하기](https://private-space.tistory.com/41)
- [Cloud Tasks 큐 빠른 시작](https://cloud.google.com/tasks/docs/quickstart-appengine)
- [Google Cloud Platform에서 Cloud Tasks 설정](https://cloud.google.com/tasks/docs/quickstart-appengine)
- [HTTP Target Task 만들기](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
