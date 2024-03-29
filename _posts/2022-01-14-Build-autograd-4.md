---
layout: post
title: "Autograd 만들기 4"
category: 만들기
---

# 서론

이전에 이어서 계속해서 만들어보자.

두 번 이상 행렬곱을 할 때에도 제대로 계산그래프가 만들어지는지를 확인하는 것에서 시작하자.

테스트 케이스를 돌려보고, 손으로 검산해봤다. 대충 잘 되는 것 같다.

# 추가

지금은 행렬곱만 제대로 처리할 수 있다. 다른 함수도 제대로 처리할 수 있게 해줘야 하지 않을까.

Func의 forward와 backward의 시그니처를 생각해보자.

지금은 backward는 args만을 받지만, propagate되어 오는 값을 받게 만드는게 더 코드가 깔끔해지지 않을까.

즉 backward는 `backward(propa, *args)`의 시그니쳐를 가지게 하는 것이 맞을 것 같다.

# 코드 분리

지금은 `tensor.py` 하나에서 모든 함수를 정의한다. 조금 더 깔끔하게 하기 위해 파일들을 분리하자.

Func들을 정의하는 `func.py`, 계산그래프가 있는 `calc_graph.py`, 그리고 `tensor.py` 이렇게 세 개면 될 것이다.

`ufunc_to_fucn`는 CalcGraph에서 쓰이는 Helper 함수인데... Func 안으로 넣는게 나을까?

그냥 함수 혼자서 놀게 하려니까 약간 부웅 뜨는 것 같은 기분이다.

## 거의 팩토리

지금 코드는 거의 팩토리 패턴을 따르고 있다.

`ufunc_to_func` 함수를 팩토리 클래스로 어떻게 분리해주면 대충 팩토리 패턴과 같은 모양이 되는 것 같다.

그러면 이를 `FuncSelecter`나 `FuncFactory`같은 클래스로 잘 감싸주면 괜찮지 않을까.

원래는 추상 클래스를 만들고 이를 상속하고 해야 하는데 넘어가자.

## Match - Case

클래스 안으로 옮기는 김에 최신 기능을 써보기로 했다.

match-case문인데, 다른 언어의 switch-case에 해당하는 문법이다.

사실 매우 간단한 예라서 match-case를 쓸 것까지도 없지만, 그래도 그 편이 간단해 보이니까 if-else 대신 match-case를 쓰자.

## 다른 함수들

일단... Element-wise 연산들을 만들어줄 필요가 있다.

### 미분?

또 이게 문제다.

좀 더 문제를 단순화시켜서 생각하자.

사실 행렬과 행렬의 계산에 대해서는 생각할 필요가 없었다.

그래프의 노드를 따라 흐르는 것은 벡터이고, 각종 연산들은 벡터를 다른 벡터로 변환하는 역할을 한다.

그렇기 때문에 우리가 구해야 할 것은 야코비안이다.

그렇다면 함수에 들어오는 args들도 벡터고 나가는 값도 벡터라고 생각을 하자.

어떻게 차원을 그렇게 고정을 시키는 방법도 고려를 하자.

#### 그건 그렇다고 칠 수 있는 방법?

다음 문단을 '그건 그렇다고 치고'로 시작할 생각이다. 그러면 그건 그렇다고 칠 수 있는 방법을 생각해야 할 것이다.

엊그제 글을 기억한다면 '행렬곱의 미분(?)' 비슷한걸 했었던걸 기억할 것이다.

정확히는 행렬의 '한 행'에 대한 미분이었다.

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7D-%20&%20A_%7B1%7D%20&%20-%20%5C%5C%5Cvdots%20&%20%5Cvdots%20&%20%5Cvdots%20%5C%5C-%20&%20A_%7Bn%7D%20&%20-%20%5C%5C%5Cend%7Bbmatrix%7D%20B%20=%5Cbegin%7Bbmatrix%7D-%20&%20A_%7B1%7DB%20&%20-%20%5C%5C%5Cvdots%20&%20%5Cvdots%20&%20%5Cvdots%20%5C%5C-%20&%20A_%7Bn%7DB%20&%20-%20%5C%5C%5Cend%7Bbmatrix%7D%20)

위 공식을 보자. 행렬과 행렬의 곱은 그 행렬을 이루는 행벡터와 행렬의 곱을 쌓아둔 것과 같다.

그렇기 때문에 '그' 행렬곱의 미분이 '각 행벡터의 야코비안 계산'이 되어 나름 정당화가 가능한 것이다.

### 야코비안 : element-wise add, subtract

뭐 그건 그렇다고 치고, 일단 뭘 계산해야 할지를 정확히 알았으니 아무튼 야코비안이다.

덧셈에 대해 야코비안을 어떻게 할지를 생각해보자.

스칼라 + 스칼라의 덧셈을 계산하면 1이다. 마찬가지로 1로 가득한 행렬이 야코비안이 될 것이다.

어짜피 element-wise add는 두 입력값과 출력값이 모두 shape가 같으므로 propagate되어 들어온 값을 그대로 전파해주면 될 것이다.

element-wise subtract도 비슷하지만, +1인지 -1인지를 잘 신경써야 한다.

### 야코비안 : element-wise multiplication, division

곱셈과 나눗셈은 어떨까?

곱셈의 야코비안 자체는 크게 까다로울 것 없다.

벡터 v1과 v2를 곱한다고 할 때, v1에 대한 야코비안은 v2를 대각행렬 모양으로 만든 것이다.

문제는 이거를 어떻게 처리할 것이냐... 인데,

계산 그래프의 기본으로 돌아가보자.

결국에는 연쇄 법칙에서 나온 법칙이니까, 다시 거기부터 생각해보면 뭐라도 나올 것이다.

u = g(x)라고 하고, y = f(u)라고 할 때, (x, u, y는 벡터)

![eq](https://wikimedia.org/api/rest_v1/media/math/render/svg/865553d07f871801f11a44e6892903a40ce1e3fb)

라고 한다. 여기서 ∇y는 전파되어 오는 값이고...

이걸 바탕으로 대충 계산해보면 그냥 단순한 element-wise multiplication이 된다는 것을 알 수 있다.

## Broadcast 문제

뭔가 잘 안돌아간다 싶더니 Broadcast와 관한 문제가 생겼다.

어떤 문제가 생기는지 건너뛰고 해결해야 할 것을 생각해보면

연산을 적용했을 때 어떻게 broadcast가 되는지를 추적해야 한다.

사용할 수 있는 정보는 함수의 시그니쳐 정보 정도인 것 같다.

일단은 완전히 broadcast되거나 전혀 broadcast되지 않는다고 가정하자.

또 문제가 하나 생겼는데, 이러면 broadcast 자체를 하나의 연산으로 생각하는게 좋을 것 같다.

broadcast는 차원이 늘어난다. 차원이 줄어들 때 grad의 변화는... 일단 임의로 평균값으로 하자.

이번엔 여기까지 하고, 다음에 이런 저런 완료되지 않은 문제들을 해결해보자.
