---
layout: post
title: "Autograd 만들기 7"
category: 만들기
---

#서론

저번에 만들던 Autograd를 계속 만들자.

일단 어디까지 만들었는지를 다시 한번 복기해보자.

가장 중요한 부품들은 다 완성됐고, 함수들의 backward들만 구현해주면 거의 완성이라고 불러도 좋겠다.

구현한 함수들은 다음과 같다.

* 행렬곱
* 원소별 합
* 원소별 차
* 브로드캐스트

이제 뭘 더 만들어야 할까?

# Todo 리스트

* 브로드캐스트 backward 다시 생각해보기
* Transpose
* 원소별 곱
* 원소별 나누기
* 제곱
* concat
* squeeze
* 각종 함수들
  * 로그
  * 삼각함수
  * exponential
  * 기타 제곱
* ...

일단 당장에 생각나는 것들은 이정도다. 뭐가 더 필요할지는 앞으로 만들어가면서 알게 되지 않을까?

# Broadcast를 다시 생각해보자.

지금은 일단 임의로 Broadcast는 평균을 취하게 해뒀다. 조금 더 그럴싸한 방법은 없을까?

인터넷을 조금 더 뒤져보고 (구글에 Gradient of broadcast라고 검색해봤다.) 나니까 어느 정도 감이 잡힌다.

짤막하게 정리하면 (지금은 평균을 계산하고 있는데) 합을 계산하는 것이다.

그 이유는 다음과 같다.

![broadcast-grad1](/images/broadcast_grad1.png)

지금 Broadcast는 대충 이런 느낌이다. 이걸 어떻게 수학적으로 표기하는 방법은 없을까?

![broadcast-grad2](/images/broadcast_grad2.png)

대충 이런 식으로, 적당한 행렬을 곱하고 (@는 행렬곱을 의미한다.) 적당히 모양에 맞게 Transpose해주면 된다.

이 때 적당한 행렬은 전부 1로만 이루어진 행렬이 되는데...

명확한 증명은 어렵지만 직관적으로 생각해보면 (이전에 했던 행렬곱의 미분 부분을 참고해보자. 정확히는 행별로 미분을 하는 것이었지만.) Propagate되어 온 값을 더하게 된다.

이전에 mean으로 되어 있던 부분을 sum으로 바꾸기만 하면 된다.

# 기타 함수 구현

곱하기, 나누기는 간단하니까 빠르게 구현해주자.

딱 하나 문제가 있다면 사실 나누기는 그렇게 간단하지 않다는 것이다.

그래도 고등학교에서 배운 수준으로 커버가 되니까 한번 해보자.

정확한 공식은 소위 '몫의 미분법'이다.

![eq](https://latex.codecogs.com/svg.image?%5C%7B%5Cfrac%20%7Bf(x)%7D%7Bg(x)%7D%5C%7D'%20=%20%5Cfrac%7Bf'(x)g(x)-f(x)g'(x)%7D%7B%5C%7Bg(x)%5C%7D%5E2%7D)

사실 이 공식이 필요한게 아니라 다른 공식이 필요하긴 한데... f(x)하고 g(x)가 여기는 x라는 공통된 놈으로 묶여(?) 있지만, 실제로는 그렇지 않다고 생각하고 계산할거다.

![eq](https://latex.codecogs.com/svg.image?%5C%7B%5Cfrac%20%7B1%7D%7Bg(x)%7D%5C%7D'=-%5Cfrac%7Bg'(x)%7D%7B%5C%7Bg(x)%5C%7D%5E2%7D)

그러면 이렇게 될 것이다.

g'(x)는 간단하게 1로 처리해주면 된다. 혹은 z = a/b라고 할 때 z를 b로 편미분하면 -a/b^2이 나온다고 생각하면 된다.

Transpose는 앞으로 가나 뒤로 가나 똑같지 않을까?

## Transpose 문제

그런데 이게 그렇게 단순하지가 않다. Transpose라는게.

이게 함수로 취급되질 않다 보니까...

`__array_wrap__`에도 잡히지를 않는다.

인터넷에 찾아보면 해결 방법이 나오지 않을까?

다음에 해결하도록 하자.

# 다음 시간

Transpose에 대한 해결책을 만들고, 위의 공식 (몫의 미분법)이 제대로 작동하는지 테스트해보자.
