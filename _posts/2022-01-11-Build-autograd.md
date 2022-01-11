---
layout: post
title: "Autograd 만들기 1"
category: 만들기
---

# 서론
![저번 글]({% post_url 2022-01-10-Autograd %})에서 Autograd에 대해서 알아봤다.

그걸 직접 만들어 볼 수는 없을까?

약간 실습 같은 느낌으로, Autograd를 시작으로 한번 머신 러닝 라이브러리 비슷한 무언가를 만들어보자.

# 준비과정

일단 최근 VSCode가 마음에 든다. VSCode에서 만들 예정이다.

그 전에 역시 깃허브 레포지토리를 하나 만들어둘까.

VSCode의 확장으로는 Python을 받아준다. 앞으로 뭔가가 더 받을 수도 있겠지.

(Python Environment Manager도 받았다. 도움이 되지 않을까?)

Conda를 이용해서 새 가상 환경을 만들어준다. 여기에는 numpy를 설치해준다.

그냥 그게 좋을 것 같으니까 `conda install python=3.10`을 써서 파이썬도 최신버전으로 받아주자.

대충 준비는 다 됐다.

# Tensor

가장 기본이 되는건 Tensor일 것이다.

일단 Tensor와 `np.ndarray`의 관계에 대해서 고심을 해봐야 할텐데...

Tensor is-a np.ndarray일까? 아니면 Tensor has-a np.ndarray일까?

아무리 생각해봐도 is-a 관계가 자연스러울 것 같다.

ndarray의 서브클래스를 만드는 방법에 대해서는 NumPy에 [공식 문서가 있다.](https://numpy.org/doc/stable/user/basics.subclassing.html) 참고해서 만들어보자.

## 계산 그래프 만들기?

솔직히 여기서 한번 막힌다.

어떻게 어떻게 ndarray의 서브클래스를 만들었다고 하자. 이제 어떻게 계산 그래프를 만들 것인가?

가장 눈에 가는건 `__array_ufunc__`다. 모든 NumPy 함수 콜은 여기로 향하니까.

`Tensor(np.ndarray)`의 `__array_ufunc__`는 실제로 계산하지 않고 계산 그래프를 만들고, 이후 forward나 backward 신호에 맞춰서 내부 콜을 보낸다?

그렇다면 텐서에는 지금까지의 그래프를 저장하는 변수가 필요할 것이다.

계산 그래프 타입 역시 필요할 것이고...

계산 그래프 타입은 입력 계산 그래프들과 함수로 이루어져야 할 것이다.

![calcgraph-class](/images/calcgraph_class.png)

대충 계산그래프 클래스는 이런 느낌이 아닐까.

결국에는 문제가 생긴다.

지금 생각으로는 계산 그래프에 저장되는 함수는 `Func`라는 새로운 클래스의 서브 클래스로 만들고 싶은데,

어떤 식으로든 NumPy의 연산을 Func의 서브 클래스로 바꾸는 방법이 필요해진다.

외부 함수를 둬서 해당 역할을 맡게 하는 것도 한 방법으로 보인다.

