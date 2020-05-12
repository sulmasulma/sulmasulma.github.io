---
layout: post
title: python collections 라이브러리 다루기
tags: [Algorithm, collections, stack, queue]
categories: [Algorithm]
excerpt_separator: <!--more-->
---
python 알고리즘을 풀 때 유용한 라이브러리 중 하나인 `collections`의 용도를 소개한다.<!--more--> 코딩 테스트에서도 사용 가능한 [python 표준 라이브러리](https://docs.python.org/ko/3/library/index.html) 중 하나로, `stack`과 `queue` 자료구조를 구현할 수 있다.<br>
`stack`과 `queue` 자료구조에 대해서는 [Stacks and Queues in Python](https://levelup.gitconnected.com/stacks-and-queues-in-python-b2e8b4dbd876)에서 알기 쉽게 설명해 놓았다.

### 1. Counter
`list`, `tuple` 등을 `Counter Dictionary`로 바꾸어 주는 클래스이다. 인자로 받은 배열의 각 값이 몇 개 있는지 반환한다. `pandas`의 `Series.value_counts()`와 유사하다.

```py
from collections import Counter

lst = [1, 2, 3, 3, 2, 1, 1, 1, 2, 2, 3, 1, 2, 1, 1]
counter = Counter(lst)
print(counter)
# >>> Counter({1: 7, 2: 5, 3: 3})
```

- lst에 1이 7개, 2가 5개, 3이 3개 있다. 각 값과 개수들이 `값: 개수` 형태로 출력된다.

또한 Counter에는 가장 개수가 많은 n개 값을 `list` 형태로 반환하는 `.most_common()` 메소드가 있다.

```py
print(counter.most_common(2))
# >>> [(1, 7), (2, 5)]
```

여기서는 각 값과 개수들이 (값, 개수) 형태로 출력된다.
<br>
<br>

### 2. defaultdict
`defaultdict`는 python dictionary와 유사하지만, 존재하지 않는 key에 접근해도 에러가 출력되지 않는다.

```py
from collections import defaultdict

names_dict = defaultdict(int)
names_dict["Bob"] = 1
names_dict["Katie"] = 2
sara_number = names_dict["Sara"]
print(names_dict)
# >>> defaultdict(<class 'int'>, {'Bob': 1, 'Katie': 2, 'Sara': 0})
```

- `defaultdict` 객체의 기본 형식으로 `int`를 지정한다.
- key와 그에 맞는 value를 지정한 "Bob", "Katie"에는 값이 나오지만, key를 지정하지 않은 "Sara"에 대해서는 `int`의 기본값인 0이 할당된다.

위 코드에서 `defaultdict`의 기본 형식으로 `int` 대신 `list`를 지정하면, 즉 `names_dict = defaultdict(list)`로 바꿔 주면 "Sara"의 값은 아래와 같이 빈 list가 할당된다.

```py
defaultdict(<class 'list'>, {'Bob': 1, 'Katie': 2, 'Sara': []})
```

- 마찬가지로 기본 형식으로 `dict`로 지정하면 "Sara"의 값은 {}가 된다.
<br>
<br>

### 3. deque
`queue`는 한 쪽에선 입력, 반대 쪽에선 출력이 되는 First-In-First-Out (FIFO) 자료구조이다. `list`를 가지고 queue를 구현할 수도 있지만, `collections` 라이브러리의 `deque`를 이용하여 더 효율적이고 편리하게 구현할 수도 있다.

<br>
<br>

### 4. namedtuple

<br>
<br>



---
출처
- [Introducing high-performance datatypes in Python with the collections library](https://levelup.gitconnected.com/introducing-high-performance-datatypes-in-python-with-the-collections-library-3d8c334827a5)
- [Stacks and Queues in Python
](https://levelup.gitconnected.com/stacks-and-queues-in-python-b2e8b4dbd876)
