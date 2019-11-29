---
layout: post
title: Jupyter Lab을 브라우저가 아닌 Desktop App에서 사용하기
tags: [jupyter, jupyter-lab, jupyter lab, chrome, 주피터랩, 주피터, 앱, no-browser]
author-id: matthew
excerpt_separator: <!--more-->
---
Jupyter Lab은 기본적으로 Web browser 위에서 구동된다. 하지만 이러면 개발 작업과 웹서핑 작업하는 앱이 겹쳐서 불편한 점이 있다. 그래서 브라우저가 아닌 Desktop App에서 사용하는 방법이 있다.<!--more-->

(Mac 기준) 터미널을 켜고 아래와 같이 입력한다.
```
$ jupyter lab --no-browser
```

그러면 아래와 같은 메시지가 나온다.
```
[C 21:21:23.257 LabApp]

    To access the notebook, open this file in a browser:
        file:///Users/qpdev/Library/Jupyter/runtime/nbserver-16530-open.html
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

하지만 매번 이렇게 여는 것은 번거롭다. 그래서 Default로 앱에서 작동하게 하는 방법이 있다. 아래 python 파일을 연다.
* 참고로 맥 터미널에서 특정 디렉토리를 여는 것은 `open {디렉토리 주소}`로 하면 된다. 맥 처음 사용할 때에는 윈도우와 같이 탐색기에서 주소로 못 가서 좀 애먹었다.. (되는데 내가 아직 모를지도?)

```
~/.jupyter/jupyter_notebook_config.py
```

그리고 아무 위치에 아래와 같이 추가한다. App 생성시 브라우저 기반으로 동작하는 것인데, 어느 브라우저를 대상으로 할지 설정하는 절차이다.
```python
c.LabApp.browser = '/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --app=%s'
```

다시 터미널을 열고 아래와 같이 입력한다.
```
$ jupyter lab --generate-config
```

그리고 이제 터미널에서 `jupyter lab`을 실행하면, 앱으로 실행된다.
다만 완전히 독립된 주피터 앱이 아니고 크롬으로서 작동하는 방식이다. 그래서 Dock(작업 표시줄)에서 구분이 되지 않는 단점이 있으며 터미널에서 `jupyter lab` 명령어를 매번 쳐야 한다.
그래서 앱을 생성하여 실행하는 방법이 있다. 필자는 Mac OS X를 사용하므로 맥의 경우를 소개하는데, Linux나 Windows의 경우 이 글 맨 밑에 있는 링크를 참고하면 된다.

### 1. Mac용 Anaconda 설치
터미널에 아래와 같이 입력한다. 이미 설치되어 있다면 다음으로 넘어가면 된다. 다만 아래 명령어는 2018년 12월 업데이트된 버전으로 보인다(최신 버전의 Anaconda를 원한다면 수정 필요).

```
wget https://repo.anaconda.com/archive/Anaconda3-2018.12-MacOSX-x86_64.sh
bash /Download/Anaconda3-2018.12-MacOSX-x86_64.sh
```

### 2. jupyter lab configure file 생성
터미널에 아래와 같이 입력한다.

```
jupyter-lab --generate-config
```
그리고 앞에서 만든 `~/.jupyter/jupyter_notebook_config.py` 파일을 수정한다. 아래의 행을 추가한다. Desktop App을 만들고, 실행시 token 입력을 생략하기 위한 절차이다.

```python
c.NotebookApp.token = ''
```

### 3. Nativefier를 이용하여 Build Desktop Application
- Nativefier github 주소: [https://github.com/jiahaog/nativefier](https://github.com/jiahaog/nativefier)
- 터미널에 아래와 같이 입력한다. npm(node.js project manager)이 이미 설치되어 있을 경우 첫 줄은 생략하면 된다.

```
# in case you didn't install node:
# conda install -c conda-forge nodejs

npm install nativefier -g
cd ~/Applications
nativefier "http://localhost:8888"
```
위 명령어는 `~/Applicatons` 폴더에 'Jupyter Notebook'이라는 앱을 생성하게 된다. 실행하면 이 작업 전에 터미널에 `http://localhost:8888`을 실행하던 것과 똑같이 브라우저에서 jupyter lab이 실행된다. 테스트 용도이므로, 이 터미널과 해당 브라우저 창은 닫아주고 진행하면 된다.
참고로 App의 이름은 Notebook이지만, 터미널에서 jupyter lab, jupyter notebook 중 어느 명령을 실행했는지에 따라 아래 둘 중 하나의 명령이 실행된다.
- `jupyter lab --no-browser --notebook-dir=~/`
- `jupyter notebook --no-browser --notebook-dir=~/`

필자는 Jupyter Lab을 선호하는데, Jupyter Notebook을 선호한다면 위에서 `nativefier "http://localhost:8888"` 대신 `nativefier "http://localhost:8888/tree"` 를 실행하면 된다.

### 4. Jupyter Lab을 서비스로 실행

i) 아래 코드를 `~/Library/LaunchAgents/com.jupyter.lab.plist`라는 파일로 저장한다. `your_username`을 여러분의 맥 사용자 이름에 맞게 고쳐주면 된다. `--notebook-dir`은 Jupyter App 시작 디렉토리에 해당한다. 본인이 원하는 경로로 지정해 주면 된다.
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

ii) 터미널에 아래와 같이 입력한다.
```
launchctl load ~/Library/LaunchAgents/com.jupyter.lab.plist
```
여기까지 완료하면, 앱 실행시 `jupyter lab --no-browser --notebook-dir=~/` 명령이 실행될 것이다!!

이 서비스를 다시 시작(**restart**)하고 싶다면, 터미널에 아래와 같이 입력하면 된다.
```
launchctl unload ~/Library/LaunchAgents/com.jupyter.lab.plist
launchctl load ~/Library/LaunchAgents/com.jupyter.lab.plist
```

이 앱을 몇 달간 사용해본 결과, 앱을 종료해도 `http://localhost:8888` 서버가 종료되지 않는다. 이렇게 장기간 서버를 켜 놓을 경우 가끔씩 서버 에러가 발생하여 컴퓨터를 재시동 해야 하는 경우가 있었다.
`launchctl unload`, `launchctl load`를 하면 서버를 재시동하는 것이다. 주기적으로나 매일 작업 종료 후 이 과정을 반복하는 것을 추천한다.


---
- 참고 문서
  - [Running Jupyter Lab as a Desktop Application](http://christopherroach.com/articles/jupyterlab-desktop-app/)
  - Mac OS X: [How to run Jupyterlab as a desktop app on Mac OS X](https://gist.github.com/xiaolai/697ec3ea1607994440abf574c0f017e5)
  - Linux: [Running JupyterLab as a desktop application on Linux](https://blog.aldomann.com/jupyterlab-desktop-on-linux/)
  - Windows: [Running JupyterLab as a Desktop Application in Windows 10
  ](https://stackoverflow.com/questions/51036132/running-jupyterlab-as-a-desktop-application-in-windows-10)
