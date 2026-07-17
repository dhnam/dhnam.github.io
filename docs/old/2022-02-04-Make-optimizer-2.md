---
layout: post
title: "Optimzer 만들기 2"
category: 만들기
---

# 서론

깊이 우선 탐색을 이용한 graph의 Iterator를 만들어보기로 했다.

서론을 적기 전에 미리 만들었다.

이제부터 본격적으로 Optimizer를 구현해보자.

# Gradient Descent (다시)

이제 그래프의 각 요소를 순회할 수 있게 Iterator도 만들었겠다, 단순한 GD Optimizer의 구현은 쉽다.

생각해보면 모든 Optimizer는 다음과 같은 순서로 작동한다.

1. Gradient를 계산한다.
2. 매 노드마다 최적화 연산을 한다.
3. (선택 사항) Gradient를 초기화한다.

1번과 3번은 미리 구현해두고, 2번에서도 for문 안에 있는 것만 구현하게 할 수 있지 않을까?

step을 final로 만들고, 실제 로직을 담당하는 optimize라는 함수를 만들어서 구현해주자.

그렇게 구현은 대충 했는데... 과연 이게 잘 작동할까?

# Playground

일종의 Playground를 만들어보고 싶다. (playground.py)

테스트 코드는 예전에 만들었던 코드를 사용하자. (학교 과제 기록을 뒤져보면 남아 있을 것이다.)

대충 토이 데이터셋을 만들고 이걸 사용하는 식의 코드다.

레이어 수는 둘로 줄이자. (생각해보니까 아직 ReLU를 만들어두질 않아서 그냥 sigmoid로 할 생각이다.)

Loss는 일단 단순하게 MSE로스를 만들어보자.

# `array_func`에 Avg 등등 추가하기

생각해보니까 필요하다. 특히 평균이 MSE를 만들 때 필요하다.

Average 함수에 weights라는 매개변수가 있는데, Backward를 만들 때 이걸 고려해주는게 좋지 않을까.

그래도 일단은 필요도 없고 하니까 NotImplementedError를 띄우게 해두자.

# 다음 시간

일단 지금 생각대로 되지 않는다... Substract에 세 항목이 들어간다거나, `zero_grad`가 무한 뺑뺑이를 돈다거나, 이것저것

다음에 마저 처리하도록 하자.
