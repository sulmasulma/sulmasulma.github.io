---
layout: post
title: 마크다운 링크 새 탭에서 열기
tags: [markdown, atag]
categories: [Github Pages]
excerpt_separator: <!--more-->
---
링크를 현재 창이 아닌 새 탭에서 열리게 하는 방법을 정리해 보았다.<!--more-->

a 태그를 이용한 링크는 기본적으로 원래 창에서 열리도록 되어 있다. 이 경우 html 코드는 다음과 같다.

```html
<a href="https://sulmasulma.github.io/">
```

글을 쓰면서 링크를 중간 중간 넣게 되는데, 글과 링크를 같이 보려면 원래 창이 아닌 새 탭에서 열리게 해야 한다. 이 경우 html 코드는 다음과 같다.

```html
<a href="https://sulmasulma.github.io/" target="_blank">
```

<br>

Github 블로그 글은 마크다운으로 작성하는데, html 코드를 사용할 수는 있지만 기본적으로는 간략화된 문법을 사용한다.

```md
[현재 탭에서 열기](https://sulmasulma.github.io/)
```

- [현재 탭에서 열기](https://sulmasulma.github.io/github%20pages/2020/07/02/atag-target.html)

**마크다운에서 링크를 새 탭에서 열리게 하려면, 다음과 같이 하면 된다.**

```md
[새 탭에서 열기](https://sulmasulma.github.io/){:target="_blank"}
```

- [새 탭에서 열기](https://sulmasulma.github.io/github%20pages/2020/07/02/atag-target.html){:target="_blank"}
