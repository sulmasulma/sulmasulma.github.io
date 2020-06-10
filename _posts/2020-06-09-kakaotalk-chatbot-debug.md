---
layout: post
title: 카카오톡 챗봇 개선 과정
tags: [kakaotalk, serverless, Data Engineering, AWS, chatbot, debug]
categories: [Data]
excerpt_separator: <!--more-->
---
카카오톡 챗봇에서 크고 작은 개선 사항들을 정리해 보았다.<!--more-->

### 1. 아티스트 이름 번역

[Spotify](https://open.spotify.com/search)에서 한국 아티스트는 한국어로 검색해도 나오지만, 외국 아티스트는 (아직까지 발견한 바로는) 비틀즈 말고는 한국어로 검색했을 때 나오지 않는다. Search API에서 `콜드플레이`를 검색하면 아무 결과도 나오지 않는다.

이는 사용자 입장에서 매우 불편한 부분이다. 구글 번역기를 손쉽게 코드로 사용할 수 있는 방법을 찾아 보다가, `googletrans`라는 아주 유용한 라이브러리를 발견하게 되었다. `pip install googletrans`로 설치하여 사용하면 된다.

```py
from googletrans import Translator
translator = Translator()

print(translator.translate('콜드플레이', dest="en").text)
# Coldplay
print(translator.translate('퀸', dest="en").text)
# queen
print(translator.translate('트래비스', dest="en").text)
# Travis
print(translator.translate('포스트말론', dest="en").text) #
# Post Marlon
```

포스트 말론의 결과인 **Post Marlon** 의 경우 정확한 아티스트 이름인 **Post Malone** 과 약간 차이가 있지만, Search API에서 Post Marlon으로 조회하면 Post Malone이 나온다.

이 정도면 문제 없는 수준이다!

<br>
<br>

---
#### 참고 문서
- [Python - Google translate(구글 번역) API 사용 방법](https://codechacha.com/ko/python-google-translate/)
