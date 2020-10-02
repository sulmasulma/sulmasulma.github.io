---
layout: post
title: Jupyter Lab을 브라우저가 아닌 Desktop App에서 사용하기
tags: [jupyter]
categories: [Data]
excerpt_separator: <!--more-->
---
Jupyter Lab을 브라우저가 아닌 Desktop App으로 사용하는 방법을 소개한다.<!--more-->

Jupyter Lab(Jupyter Notebook)은 기본적으로 Web browser 위에서 구동되지만, 이러면 개발 작업과 웹서핑 작업하는 앱이 겹쳐서 불편한 점이 있다. 그래서 브라우저가 아닌 Desktop App에서 사용하는 방법을 소개하고자 한다. 문서는 Mac OS 기준으로 작성하였다.

먼저 터미널을 켜고 아래와 같이 입력한다.
```
$ jupyter lab --no-browser
```

그러면 아래와 같은 메시지가 나온다.
```
[C 21:21:23.257 LabApp]

    To access the notebook, open this file in a browser:
        file:///Users/your_username/Library/Jupyter/runtime/nbserver-16530-open.html
    Or copy and paste one of these URLs:
        http://localhost:8888/?token=a735772a647a3fffb2e140424d8906b92f51b7162d735b2d
     or http://127.0.0.1:8888/?token=a735772a647a3fffb2e140424d8906b92f51b7162d735b2d
```

`Or copy and paste one of these URLs:` 뒤에 나오는 URL을 복사한다.
이제 다른 터미널 창을 열고, 복사한 URL을 이용하여 다음과 같이 입력한다.

```
$ /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --app=
http://localhost:8888/?token=a735772a647a3fffb2e140424d8906b92f51b7162d735b2d
```

그러면 Jupyter Lab이 브라우저 위에서가 아닌 독립된 앱 형태로 작동된다.

![스크린샷 2019-10-18 오후 2.06.45](https://i.imgur.com/UI9hTxJ.png)

하지만 매번 이렇게 여는 것은 번거롭다. 그래서 Default로 앱에서 작동하게 하는 방법이 있다. 우선 터미널에 아래와 같이 입력한다. Jupyter Lab의 configure 파일을 생성하는 과정이다.
```
$ jupyter lab --generate-config
```

그럼 `~/.jupyter/jupyter_notebook_config.py` 파일이 생성된다. 이 파일을 열기 위해서, 터미널에 아래와 같이 입력한다.
```
open ~/.jupyter/jupyter_notebook_config.py
```

파일을 열었으면, 아무 위치에 아래와 같이 추가한다. App 생성시 브라우저 기반으로 동작하는 것인데, 어느 브라우저를 대상으로 할지 설정하는 절차이다. 필자는 `Chrome`을 메인 브라우저로 쓰기 때문에 아래와 같이 입력했다.
```python
c.LabApp.browser = '/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --app=%s'
```

그리고 나서 터미널에서 `jupyter lab`을 실행하면, 앱으로 실행된다.
다만 완전히 독립된 주피터 앱이 아니고 `Chrome`으로서 작동하는 방식이다. 그래서 Dock(작업 표시줄)에서 구분이 되지 않는 단점이 있으며, 실행할 때마다 터미널에서 `jupyter lab` 명령어를 매번 쳐야 한다.

그래서 **앱을 생성하여 실행하는 방법이 있다.** 필자는 Mac OS를 사용하므로 맥의 경우를 소개하는데, Linux나 Windows의 경우 이 글 맨 밑에 있는 링크를 참고하면 된다.

### 1. Mac용 Anaconda 설치
터미널에 아래와 같이 입력한다. 이미 설치되어 있다면 다음으로 넘어가면 된다. 2018년 12월 Mac OS용 버전으로 나와 있는데, 다른 버전을 원하거나 다른 OS에 설치를 원한다면 [https://repo.anaconda.com/archive/](https://repo.anaconda.com/archive/)에서 버전, OS에 맞는 파일 이름을 입력하면 된다.

```
wget https://repo.anaconda.com/archive/Anaconda3-2018.12-MacOSX-x86_64.sh
bash /Download/Anaconda3-2018.12-MacOSX-x86_64.sh
```

### 2. jupyter lab configure file 생성
터미널에 아래와 같이 입력한다. 위 과정(Chrome 기반 앱 생성)을 통해 이미 configure file을 생성했다면, 이 단계는 넘어가면 된다.

```
jupyter-lab --generate-config
```

이제 `~/.jupyter/jupyter_notebook_config.py` 파일을 열고 아래의 행을 추가한다. Desktop App을 만들면 실행시 token 입력 과정이 필요한데, 이를 생략하기 위한 절차이다.

```python
c.NotebookApp.token = ''
```

### 3. Nativefier를 이용하여 Desktop Application 빌드
- Nativefier github 주소: [https://github.com/jiahaog/nativefier](https://github.com/jiahaog/nativefier)
- `nativefier`(웹 페이지를 브라우저에서 열지 않고 앱으로 생성해 주는 패키지)를 이용하여 Desktop App을 만드는 과정이다.
- 터미널에 아래 4개의 행을 차례로 입력한다. nodejs 패키지가 이미 설치되어 있을 경우 첫 줄은 생략하면 된다.

```
# in case you didn't install node:
conda install -c conda-forge nodejs

npm install nativefier -g
cd ~/Applications
nativefier -n "Jupyter Lab" -i ~/Desktop/jupyter.icns "http://localhost:8888"
```

위 명령어는 `~/Applicatons` 폴더에 'Jupyter Lab'이라는 이름(**-n**)으로, ~/Desktop/jupyter.icns 아이콘 파일로(**-i**) 앱을 생성하는 과정이다.<br>
아이콘 파일들은 anaconda3 폴더 안의 `/pkgs/notebook-6.0.3-py37_0/info/recipe/`에 있다. Windows는 jupyter.ico, Linux는 jupyter.png 파일을 사용하면 된다. Mac OS의 경우는 jupyter.ico 파일을 .icns 파일로 변환해야 한다.

명령어를 실행하면 이 작업 전에 터미널에 `http://localhost:8888`을 실행하던 것과 똑같이 브라우저에서 jupyter lab이 실행된다. 테스트 용도이므로, 이 터미널과 해당 브라우저 창은 닫아주고 진행하면 된다.
참고로 App의 이름을 Jupyter Lab으로 지정했지만, 터미널에서 jupyter lab, jupyter notebook 중 어느 명령을 실행했는지에 따라 아래 둘 중 하나의 명령이 실행된다.
- `jupyter lab --no-browser --notebook-dir=~/`
- `jupyter notebook --no-browser --notebook-dir=~/`

필자는 Jupyter Lab을 선호하는데, Jupyter Notebook을 선호한다면 위에서 `nativefier "http://localhost:8888"` 대신 `nativefier "http://localhost:8888/tree"` 를 실행하면 된다.

### 4. Jupyter Lab을 서비스로 실행

**i)** 아래 코드를 `~/Library/LaunchAgents/com.jupyter.lab.plist`라는 파일로 저장한다. `your_username`을 여러분의 맥 사용자 이름에 맞게 고쳐주면 된다. `--notebook-dir`은 Jupyter App 시작 디렉토리에 해당한다. 본인이 원하는 경로로 지정해 주면 된다.

(2020.03.24 수정사항) `/Users/your_username/anaconda3/bin/jupyter` 부분은 anaconda3를 [Anaconda 설치 페이지](https://www.anaconda.com/distribution/)에서 Graphical Installer로 설치했을 때 자동으로 지정되는 jupyter의 경로인데, 다시 설치해 보니 경로가 `/opt/anaconda3/bin/jupyter`으로 **바뀌어 있다!!**<br>
터미널에서 `which jupyter`를 입력하면 jupyter의 경로가 나오는데, 이에 맞춰서 알맞게 입력하길 바란다. 설치할 때 이 경로를 지정하고 싶다면, Graphical Installer가 아닌 Command Line Installer로 설치하면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>local.job</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/your_username/anaconda3/bin/jupyter</string>
		<string>lab</string>
		<string>--no-browser</string>
		<string>--notebook-dir=/Users/your_username/</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StandardErrorPath</key>
	<string>/tmp/local.job.err</string>
	<key>StandardOutPath</key>
	<string>/tmp/local.job.out</string>
</dict>
</plist>
```

**ii)** 터미널에 아래와 같이 입력한다.
```
launchctl load ~/Library/LaunchAgents/com.jupyter.lab.plist
```
여기까지 완료하면, 앱 실행시 `jupyter lab --no-browser --notebook-dir=~/` 명령이 실행되는 것이다.

이 서비스를 다시 시작(**restart**)하고 싶다면, 터미널에 아래와 같이 입력하면 된다.
```
launchctl unload ~/Library/LaunchAgents/com.jupyter.lab.plist
launchctl load ~/Library/LaunchAgents/com.jupyter.lab.plist
```

위 과정을 한 줄의 명령으로 실행하는 함수를 만들어 사용할 수 있다. 터미널에 `vim ~./bash_profile`을 입력하여 bash_profile 파일에 아래 코드를 추가하면 된다. 변경 사항은 Mac을 재시동해야 적용된다.

```
function lctl {
    COMMAND=$1
    PLIST_FILE=$2
    if [ "$COMMAND" = "reload" ] && [ -n "$PLIST_FILE" ]
      then
        echo "reloading ${PLIST_FILE}.."
        launchctl unload ${PLIST_FILE}
        launchctl load ${PLIST_FILE}
      else
        echo "either command not specified or plist file is not defined"
    fi
}
```

재시동 후 터미널에 아래와 같이 입력하면, 서비스가 다시 시작된다. **Jupyter 실행 결과도 모두 초기화되니 주의해서 사용하기 바란다.**

```
lctl reload ~/Library/LaunchAgents/com.jupyter.lab.plist
```

이 앱을 몇 달간 사용해본 결과, 앱을 종료해도 `http://localhost:8888` 서버가 종료되지 않는다. 이렇게 장기간 서버를 켜 놓을 경우 가끔씩 서버 에러가 발생하여 컴퓨터를 재시동 해야 하는 경우가 있었다. 그래서 주기적으로나 매일 작업 종료 후 재시작 과정을 반복하는 것을 추천한다.

---
- 참고 문서
  - [Running Jupyter Lab as a Desktop Application](http://christopherroach.com/articles/jupyterlab-desktop-app/)
  - Mac OS X: [How to run Jupyterlab as a desktop app on Mac OS X](https://gist.github.com/xiaolai/697ec3ea1607994440abf574c0f017e5)
  - Linux: [Running JupyterLab as a desktop application on Linux](https://blog.aldomann.com/jupyterlab-desktop-on-linux/)
  - Windows: [Running JupyterLab as a Desktop Application in Windows 10
  ](https://stackoverflow.com/questions/51036132/running-jupyterlab-as-a-desktop-application-in-windows-10)
  - [Nativefier](https://github.com/jiahaog/nativefier/)
  - [Nativefier Icons](https://github.com/jiahaog/nativefier-icons)
