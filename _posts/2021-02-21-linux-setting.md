---
layout: post
title: Linus 초기 세팅
tags: [cloud, gcp]
categories: [Cloud]
excerpt_separator: <!--more-->
---
<!--more-->앞으로 계속 AWS, GCP 등 클라우드 관련 포스팅을 많이 할 것 같아서 Cloud 테마를 별도로 생성했다. 이번 글에서는 리눅스(특히 GCP `Compute Engine`) 초기 세팅에 필요한 것들을 정리해 보았다.

우선 터미널에서 GCP Compute Engine의 VM 인스턴스를 사용하는 방법이다.

<br>

#### VM 인스턴스 접속

- `gcloud compute ssh --project={project_id} --zone={region_name} {instance_name}`
- 예시: `gcloud compute ssh --project=fine-*** --zone=asia-northeast3-a instance-***`

#### 로컬에서 VM 인스턴스로 파일 복사

- `gcloud compute scp {file_name} --project={projec_id} --zone={region_name} {instance_name}:{VM 인스턴스 위치}`
- `gcloud compute scp photo_bot_linux.py --project=fine-*** --zone=asia-northeast3-a instance-***:~/`

<br>

#### 초기 설치

보통 `yum`이나 `wget`으로 필요한 패키지를 설치하지만, VM 인스턴스 초기에는 이조차 설치되어 있지 않아 먼저 설치해야 한다. `sudo`를 통해 root 사용자로 설치해야 한다.

- `yum` 설치: `sudo apt-get install yum`
- `wget` 설치: `sudo apt-get install wget`
- 압축 관련 설치: `sudo apt install unzip`, `sudo apt install zip`
- `pip3` 설치: `sudo apt-get install python3-pip`

참고로, chrome 및 chromedriver를 설치하는 방법은 [Slack 사진 업로드 봇 만들기](https://sulmasulma.github.io/etc/2021/02/17/slack-photo-upload-bot.html)에 적어 놓았다.

<br>

#### 환경변수 설정
- `vi ~/.bashrc`
  - 편집기가 뜨면 빈 공간에 환경변수 입력: `export val_name = val`
- `source ~/.bashrc`
- `echo $val_name` 으로 확인

<br>

---
