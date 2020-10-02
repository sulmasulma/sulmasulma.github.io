---
layout: post
title: 스택/큐
tags: [Algorithm, stack, queue]
categories: [Algorithm]
excerpt_separator: <!--more-->
---
스택(Stack)은 한 쪽으로만 입출력이 가능한 FILO (First In Last Out), 큐(Queue)는 한 쪽에선 입력, 한 쪽에선 출력되는 FIFO (First In First Out) 구조로 데이터를 저장하는 자료구조이다. 스택과 큐 알고리즘에 대한 예를 정리해 보았다.<!--more-->

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
- 뒤에서부터 추가한 답을 `answer` 리스트에 추가한 것이므로, `answer`를 반대로 처리하여 앞에서부터의 값이 오도록 했다.

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
> 인쇄 대기열에 있는 문서들을 정해진 규칙대로 인쇄하는 문제이다. 규칙은 아래와 같다. 현재 대기목록에 있는 문서의 중요도가 순서대로 담긴 배열 priorities와 내가 인쇄를 요청한 문서가 현재 대기목록의 어떤 위치에 있는지를 알려주는 location이 매개변수로 주어질 때, 내가 인쇄를 요청한 문서가 몇 번째로 인쇄되는지 return 하도록 solution 함수를 작성한다.

1. 인쇄 대기목록의 가장 앞에 있는 문서(J)를 대기목록에서 꺼냅니다.
2. 나머지 인쇄 대기목록에서 J보다 중요도가 높은 문서가 한 개라도 존재하면 J를 대기목록의 가장 마지막에 넣습니다.
3. 그렇지 않으면 J를 인쇄합니다.

- [문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42587)
- 예: 4개의 문서(A, B, C, D)가 순서대로 인쇄 대기목록에 있고 중요도가 [2, 1, 3, 2] 라면 C D A B 순서대로 인쇄된다. location이 2로 주어지면, C의 인쇄 순서인 1이 출력된다.

#### 예시 1
```py
def solution(p, location):
    orders = [] # 인쇄 완료한 문서의 인덱스들이 차례로 담김
    temp = list(range(len(p))) # 인덱스들의 순서를 저장하는 리스트
    while p:
        p2 = p.copy() # 원래 리스트에서 순서대로 확인하기 위해 복사본 저장
        for i in range(len(p)):
            print(p)
            for j in range(i+1, len(p)):
                if p2[j] > p2[i]:
                    # 뒤에 중요도가 큰 문서가 있으면 맨 뒤로 보냄
                    p.append(p2[i])
                    p.pop(0)
                    temp.append(temp[0]) # 먼저 빼면 값이 사라지므로, 뒤로 먼저 보내고 앞에거를 뺌
                    temp.pop(0)
                    break
            else:
                # 대기열(p2) 맨 앞에 있는 문서를 인쇄
                p.pop(0)
                orders.append(temp[0])
                temp.pop(0)
                break

    return orders.index(location) + 1
```
- 인쇄할 문서를 p에서 차례대로 꺼낸다.
- 앞에 있는 문서부터 확인하는데, 첫 번째 문서가 중요도가 가장 높지 않을 경우 맨 뒤로 보내야 한다. 첫 번째 문서를 p에서 빼고(`p.pop`) 다음 인덱스를 확인하면, 처음 p에서 인덱스 번호가 1이었던 문서가 아니고 2였던 문서를 확인하게 된다.
- 그래서 `p`의 복사본인 `p2`를 생성하여 차례대로 훑도록 한다.
- `temp`에는 초기값으로 각 문서의 인덱스를 저장하고, 인쇄 순서에 따라 변경되는 인덱스를 담는다.
- `orders`에 각 문서들의 인쇄 순서들이 저장된다.

#### 예시 2
```py
def solution(priorities, location):
    queue =  [(i,p) for i,p in enumerate(priorities)]
    answer = 0
    while True:
        cur = queue.pop(0)
        # 자기보다 중요도 큰 문서가 있으면 queue의 맨 뒤로 보냄
        if any(cur[1] < q[1] for q in queue):
            queue.append(cur)
        # 인쇄할 문서
        else:
            answer += 1
            if cur[0] == location:
                return answer
```
- 위 예시보다 훨씬 간단한 코드이다.
- 각 문서의 인덱스와 중요도는 `enumerate`로 저장한다.
- `cur = queue.pop(0)`에서 비교할 문서를 저장하고, `any`를 이용하여 자기보다 중요도가 큰 문서가 있을 경우 queue의 맨 뒤로 보내고, 그렇지 않을 경우(`else`) 인쇄한다.
- `cur[0]`(인쇄할 문서의 인덱스)을 `location`과 비교하여, 이 과정을 `location` 순서에 해당하는 문서가 올 때까지 반복한다. `return`을 실행하면 루프가 종료되기 때문이다.
- **any로 비교하고 return으로 종료하는 것이 핵심이라고 생각한다.**


---
출처: [프로그래머스 스택/큐 문제](https://programmers.co.kr/learn/courses/30/parts/12081)
