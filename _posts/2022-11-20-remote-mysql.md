---
layout: post
title: MySQL 원격 접속 허용하기
tags: [setting, mysql]
categories: [Etc]
excerpt_separator: <!--more-->
---
<!--more-->[지난 글](https://sulmasulma.github.io/etc/2022/11/06/macmini-setting.html)에서 맥미니 서버 초기 세팅부터 mysql 설치까지 다루었는데, 이번에는 다른 환경에서 mysql에 원격 접속하기 위한 내용을 적어 보았다.

---

### 1. 원격 접속 사용자 생성

- 이 부분은 [How to install MySQL on Mac and Allow Remote Access](https://medium.com/@haydane/how-to-install-mysql-on-mac-and-allow-remote-access-b6c730aba09b) 참고

- 원래 mysql이 설치되어 있는 환경에서 root 계정으로 접속
  - `mysql -u root -p`

- remote 계정 생성
  ```sql
  create user 'sulmasulma'@'%' identified by 'password'; -- 계정 생성(%: 모든 ip에서 접속 가능)
  grant all on *.* to 'sulmasulma'@'%'; -- 모든 db 접속 권한 부여
  flush privileges; -- 권한 적용
  ```

<br>

---

### 2. 원격 접속 허용

- my.cnf 파일 찾기
  ```sh
  matthew@imataeui-Macmini ~ % mysql --verbose --help | grep my.cnf
                        order of preference, my.cnf, $MYSQL_TCP_PORT,
  /etc/my.cnf /etc/mysql/my.cnf /opt/homebrew/etc/my.cnf ~/.my.cnf
  ```

- Homebrew로 설치한 mysql의 경우 `/opt/homebrew/etc/my.cnf`에 해당함. 해당 파일을 열고 아래와 같이 수정
  ```sh
  bind-address = 0.0.0.0
  ```
  - 기본값은 `127.0.0.1`: localhost만 허용
  - `0.0.0.0`: 모든 IP 허용

- 변경사항 적용 위해 서비스 재시작
  - `brew services restart mysql`

<br>

---

### 3. 포트 포워딩 및 원격 접속

- 포트 포워딩
  - [지난 글](https://sulmasulma.github.io/etc/2022/11/06/macmini-setting.html)의 ssh 포트 포워딩과 같이, mysql 디폴트 포트 번호인 3306으로 포트 포워딩

- 이제 mysql이 설치된 PC 말고, 원격 접속할 PC에서 mysql 접속
  ```sh
  mysql -u username -h {host IP} -P 3306
  ```
  - 예시: `mysql -u sulmasulma -h 111.222.123.123 -P 3306`

### python (pymysql)을 이용한 접속

```py
import pymysql, logging

dbinfo = {
    'host': '111.222.123.123',
    'username': 'sulmasulma',
    'password': 'password',
    'database': 'mysql',
    'port': 3306,
}

# connect MySQL
try:
    conn = pymysql.connect(
        host=dbinfo['host'],
        user=dbinfo['username'],
        passwd=dbinfo['password'],
        db=dbinfo['database'],
        port=dbinfo['port'],
        use_unicode=True, charset='utf8'
    )
    cursor = conn.cursor()
except:
    logging.error("could not connect to rds")
    sys.exit(1)
```

- 정상적으로 코드 실행되는 것 확인

<br>

---

### 마치며

클라우드 mysql을 사용할 때에는 인스턴스를 생성하고 바로 이용할 수 있었는데, 로컬 서버에서 구축하려니 생각보다 신경 써야 할 게 많은 것 같다. 이 경험이 뼈와 살이 되길 기대한다.

<br>

---
#### 참고 문서
- [How to install MySQL on Mac and Allow Remote Access](https://medium.com/@haydane/how-to-install-mysql-on-mac-and-allow-remote-access-b6c730aba09b)
- [mac brew에서 설치한 mysql my.cnf가 안먹을때 (tistory.com)](https://harrythegreat.tistory.com/entry/mac-brew%EC%97%90%EC%84%9C-%EC%84%A4%EC%B9%98%ED%95%9C-mysql-mycnf%EA%B0%80-%EC%95%88%EB%A8%B9%EC%9D%84%EB%95%8C)
