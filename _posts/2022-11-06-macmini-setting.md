---
layout: post
title: 맥미니 서버 세팅
tags: [setting]
categories: [Etc]
excerpt_separator: <!--more-->
---
매달 나가는 AWS 요금에 지쳐서 맥미니 서버를 샀다. <!--more--> 특히 DB 비용이 부담되기도 하고.. 앞으로 로컬 서버로 개인 프로젝트를 진행할 예정이다.
클라우드를 아예 사용하지 않는 건 아니고, AWS `Lambda`, GCP `Cloud Function` 같은 Serverless 서비스는 프리 티어 범위가 매우 넓어서 계속 사용할 것 같다.

---

### 1. ssh 설정

- 외부 기기에서 맥미니로 원격 접속을 위해 필요

#### 포트 포워딩
- [맥 미니, 맥북, 우분투를 ssh 서버로 사용하기](https://dev-repository.tistory.com/m/96) 참조
- 공유기 회사별로 설정 방법 다름
- 난 머큐시스(Mercusys) 공유기
  1. [mwlogin.net](http://mwlogin.net/) 접속하여 Advanced로 들어감
  2. NAT Forwarding > Port Forwading에서 +Add 버튼 눌러 맥미니 설정하고, 포트번호 설정 (맥은 기본적으로 22)
  3. Network > Status에서 내 IP Address 확인

#### 포트 변경
- [맥 SSH/SFTP 포트 변경 방법](https://circumeo.tistory.com/33) 참조
- 위에서 맥 디폴트 포트번호인 22로 설정할 경우 원격 접속시 `ssh username@hostname`로 포트번호 옵션 없이 원격 접속 가능
- 포트번호를 변경하려면 아래 옵션들 적용 필요
- `sudo vi /etc/ssh/sshd_config`
  - **#Port 22** 찾아서 수정
- `sudo vi /etc/services`
  - **ssh 22/udp** 찾아서 수정
  - **ssh 22/tcp** 찾아서 수정
- ssh 재시작
  - `sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist`
  - `sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist`
- 이제 `ssh username@hostname -p "변경된 포트번호"`로 접속 가능

<br>

---

### 2. Homebrew 설치

- [설치 사이트](https://brew.sh/index_ko)
  ```sh
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
- 설치 완료 후 나오는 추가 명령 실행
  ![logo](/assets/20221106-1-homebrew.png)
  ```sh
  echo '# Set PATH, MANPATH, etc., for Homebrew.' >> /Users/matthew/.zprofile
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/matthew/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
- 설치 확인
  ```
  matthew@imataeui-Macmini ~ % brew -v
  Homebrew 3.6.8
  Homebrew/homebrew-core (git revision 9acf969c74a; last commit 2022-11-04)
  ```

<br>

---

### 3. mysql 설치

- [Mac에 MySQL 설치하기 (M1칩)](https://velog.io/@haleyjun/MySQL-Mac%EC%97%90-MySQL-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-M1%EC%B9%A9) 참조
  - 요즘 확실히 `velog` 블로그가 대세인 것 같다. 사이트는 매우 좋으나 자유도가 떨어진다고 해서 그대로 github blog 사용 중
- `brew install mysql`
- 설치 완료 확인
  ```
  ==> mysql
  We've installed your MySQL database without a root password. To secure it run:
      mysql_secure_installation

  MySQL is configured to only allow connections from localhost by default

  To connect run:
      mysql -u root

  To restart mysql after an upgrade:
    brew services restart mysql
  Or, if you don't want/need a background service you can just run:
    /opt/homebrew/opt/mysql/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql
  ```
- mysql 시작
  - `mysql.server start`
- mysql 기본 설정
  - `mysql_secure_installation`
- mysql 사용하기
  - `mysql -u root -p`
- 추가 설정: mysql 서버가 재부팅 상관없이 켜져있게 함 (mysql 종료 후 bash에 입력)
  - `brew services start mysql`
  - 맥미니를 서버로 사용할 예정이므로, 해당 옵션 필요

<br>

---
출처
- [맥 미니, 맥북, 우분투를 ssh 서버로 사용하기](https://dev-repository.tistory.com/m/96)
- [맥 SSH/SFTP 포트 변경 방법](https://circumeo.tistory.com/33)
- [Mac에 MySQL 설치하기 (M1칩)](https://velog.io/@haleyjun/MySQL-Mac%EC%97%90-MySQL-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-M1%EC%B9%A9)
