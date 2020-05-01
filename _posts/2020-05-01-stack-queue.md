---
layout: post
title: 스택/큐
tags: [Algorithm, Stack, Queue]
categories: [Algorithm]
excerpt_separator: <!--more-->
---
스택은 FIFO (First In First Out), 큐는 FILO (First In Last Out) 구조로 데이터를 저장하는 자료구조이다. 스택과 큐 알고리즘에 대한 예를 정리해 보았다.<!--more-->

### 1. 탑
> 수평 직선에 탑 N대를 세웠습니다. 모든 탑의 꼭대기에는 신호를 송/수신하는 장치를 설치했습니다. 발사한 신호는 신호를 보낸 탑보다 높은 탑에서만 수신합니다. 또한, 한 번 수신된 신호는 다른 탑으로 송신되지 않습니다. heights는 길이 2 이상 100 이하인 정수 배열이며, 신호를 수신하는 탑이 없으면 0으로 표시합니다.

- [문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42588)

#### 예시 1
```python
def solution(heights):
    answer = []
    for i in reversed(range(len(heights))):
        for j in reversed(range(i)):
            if heights[j] > heights[i]:
                answer.append(j + 1)
                break
            # 신호를 수신하는 탑이 없으면, 즉 자기보다 높은 탑이 없으면 0 추가
            if j == 0:
                answer.append(0)
    # 가장 왼쪽 탑은 신호를 받을 탑이 없음
    answer.append(0)
    return list(reversed(answer))
```
- 배열에 `reversed` 함수를 사용하여 앞부터가 아닌 뒤부터 읽도록 했다. `i`에서 뒤로 읽어 뒷쪽 탑부터, `j`에서 뒤로 읽어 바로 앞 탑부터의 높이와 비교하여 자기보다 높은 탑이면 **index + 1** 을 답이 되는 리스트에 넣는다. 2번째 탑의 경우 1번 인덱스에 있기 때문이다.

#### 예시 2
```py
def solution(heights):
    answer = [0] * len(heights)
    # 마지막 순서 ~ 2번째 탑(1번째 인덱스)까지만 하면 됨.
    for i in range(len(heights) - 1, 0, -1):
        for j in range(i - 1, -1, -1):
            if heights[j] > heights[i]:
                answer[i] = j+1
                break
    return answer
```
- `reversed` 함수 대신 `range`의 세 번째 인자로 -1을 넣어 순서를 반대로 한 경우이다.
- 먼저 모든 원소가 0이고 탑 개수만큼의 길이를 가진 list를 만든다. 이 경우 첫 번째 탑의 신호를 받을 탑이 없어 항상 결과가 0이 나오므로, 마지막 탑 ~ 2번째 탑(1번 인덱스)까지만 하면 된다. 그래서 첫 번째 for loop의 `range`에서 두 번째 인자가 0이 된다. 0 전의 인덱스, 즉 1번 인덱스까지만 돌기 때문이다.
<br>
<br>

### 2. 프린터
> d

- [문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42587)


---
출처: [프로그래머스 스택/큐 문제](https://programmers.co.kr/learn/courses/30/parts/12081)
