---
layout: post
title: Jupyter Lab을 Chrome App에서 사용하기
tags: [jupyter, jupyter-lab, chrome]
author-id: matthew
excerpt_separator: <!--more-->
---
Jupyter Lab은 기본적으로 Chrome browser 위에서 구동된다. 하지만 이러면 다른 웹사이트 창과 겹칠 수도 있고 (Windows에선 그런 적은 없지만) Mac에선 서버 오류가 계속 난다.<!--more-->

그래서 브라우저가 아닌 Chrome App에서 사용하는 방법이 있다. (Mac 기준)
터미널을 켜고 아래와 같이 입력한다.
```
$ jupyter lab --no-browser
```

그러면 아래와 같이 메시지가 나온다.
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

그러면 Jupyter Lab이 브라우저 위에서가 아닌 독립된 앱에서 작동된다. (캡쳐본 추가하기!!)
하지만 매번 이렇게 여는 것은 번거롭다. 그래서 Default로 앱에서 작동하게 하는 방법이 있다. 아래 python 파일을 연다.
* 참고로 맥 터미널에서 특정 디렉토리를 여는 것은 `open {디렉토리 주소}`로 하면 된다. 맥 처음 사용할 때에는 윈도우와 같이 탐색기에서 주소로 못 가서 좀 애먹었다.. (되는데 내가 아직 모를지도?)
```
~/.jupyter/jupyter_notebook_config.py
```

그리고 아무 위치에 아래와 같이 입력한다.
```python
c.LabApp.browser = '/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --app=%s'
```

다시 터미널을 열고 아래와 같이 입력한다.
```
$ jupyter lab --generate-config
```

그리고 이제 터미널에서 `jupyter lab`을 실행하면, 앱으로 실행된다.
다만 앱 아이콘이 주피터가 아니고 크롬으로 나와서, Dock에서 구분이 되지 않는 단점은 있다. 최적의 ipython IDE를 찾기 위한 노력은 계속된다..
- 여기에 추가로 앱을 생성하여 실행하는 방법이 있다.
- [Running JupyterLab as a desktop application on Linux](https://blog.aldomann.com/jupyterlab-desktop-on-linux/) 리눅스 환경에서 작업했는데, 맥에서 하는 방법도 알아보자.


- 참고 문서
  - [Running Jupyter Lab as a Desktop Application](http://christopherroach.com/articles/jupyterlab-desktop-app/)
