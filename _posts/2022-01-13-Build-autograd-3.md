---
layout: post
title: "Autograd 만들기 3"
category: 만들기
---

# 서론

이전에 이어서 계속해서 만들어보자.

자동으로 계산 그래프를 만들고, 자동으로 미분을 할 수 있다면 성공이다.

# 어떻게 `__call__`해야 할까?

계산 그래프를 어떻게 만들었다고 하자. 그러면 이제 실제로 계산하고 각 텐서마다 값을 기억해 둬야 한다.

텐서마다 값을 기억하게 하는 방법은 간단하다. `np.ndarray`를 상속받았으니 self를 사용하면 될 것이다.

문제는 제일 앞에 해당하는 텐서를 찾아서, 거기서부터 차근차근 계산해나가는 것이다.

지금 구조 상으로는 Tensor가 가지고 있는 것은 계산 그래프이고, 계산 그래프는 계산 그래프만을 참조로 가지고 있기 때문에 계산 그래프에서 Tensor를 호출할 방법이 없다.

계산 그래프의 생성자에 Tensor를 넣어줘야지 맞겠다.

TF 1.0때의 Placeholder에 해당하는 무언가가 필요할 것이다. Tensor의 일종으로 만들거나 Func의 일종으로 만들어야 할 것이다.

Leaf 노드를 Func의 일종으로 만드는 것은 왜인지 좀 아닌 것 같은 기분이 든다. CalcGraph의 일종으로 만들어야 할까.

함수가 instantiate되어 사용되는게 맞나? 스태틱으로 만들어 두는게 맞지 않을까?

# 오늘의 결과

일단 행렬곱에 대해서만이지만 어떻게든 forward/backward가 작동하게 만들었다.

아직 두 번 이상 행렬곱을 할 때에도 제대로 되는지는 확인해보지 않았다.

다음에는 그걸 확인하는 것에서 시작하자.
