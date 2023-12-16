### 자료구조 stack, python 의 deque 에 대해서

- 스택 : 후입선출의 원칙. 가장 최근에 스택에 추가된 항목이 장장 먼저 제거된다.
  python 에서는 List 를 이용해 구현 가능.
  리스트의 `append()` 메소드를 이용해 스택에 추가(push) 하고,
  `pop()` 메소드를 이용해 가장 최에에 추가된 데이터를 스택에서 제거(pop) 한다.

`appendleft()`는 deque(double-ended queue) 라는 자료구조의 메소드이다. deque는 양쪽 끝에서 항목의 추가 및 제거를 허용하는 자료구조로 python 의 collections 모듈에서 제공된다. `appendleft()` 메소드는 deque 왼쪽끝에 항목을 추가하는 기능이다.

- 스택은 말그대로 그릇이 쌓여있는 모습이다. 위에서부터 쌓고, 위에서 부터 뺄 수 있다.

![이미지](https://p.ipic.vip/6p40ha.png)
