---
layout: post
title: "Autograd 만들기 9"
category: 만들기
---

# 서론

함수들을 구현하자.

# 새로 구현한 것들

* 제곱
* 자연로그
* 로그 2
* 로그 10
* sin, cos, tan
* sinh, cosh, tanh
* exp, exp2

# Concat과 Squeeze

얘내들은 수학 함수는 아니지만 머신러닝을 하다 보면 가끔 쓰인다.

일단 수학 함수들은 다들 구현됐고 얘내 구현이 나왔는데... 문제가 하나 있다.

얘내들은 추가 매개변수를 받는다.

Broadcast와 비슷한 방법으로 어떻게 처리해주면 될 것 같다.

다만 지금 match-case가 ufunc로 이루어지고 있는걸 ufunc와 매개변수 쌍의 튜플로 이루어지게 하면 되지 않을까.

-하고 단순하게 생각했는데, 또 문제가 생겼다.

이거, concat은 수학 함수같은게 아니라서 `__array_wrap__`(지금까지는 여기서 어떤 함수가 불렸는지를 알아왔다.)을 부르지를 않는다.

찾아보니 `__array_function__`을 사용하면 된다고 한다. 참고하자.

# `FuncFactory` 코드 정리

해당 문서를 보다가, `FuncFactory`의 개선점을 발견했다.

지금 방법은 함수를 구현하고, 구현한 함수와 원래 함수 쌍을 FuncFactory의 코드 내에다가 넣어주는 식인데...

이걸 함수 데코레이터를 이용해서 깔끔하게 만들 수 있다.

잘 정리해주자.

```python
class FuncFactory:
    IMPL_FUNC = {}
    @staticmethod
    def generate(ufunc: np.ufunc) -> Func:
        if ufunc in FuncFactory.IMPL_FUNC:
            return FuncFactory.IMPL_FUNC[ufunc]
        warning(ufunc)
        return FuncNil
```

길었던 FuncFactory가 이렇게 줄어들었다.

`IMPL_FUNC`는 implements라는 데코레이터가 채워준다.

# 다음 시간

Concat과 Squeeze에 대해서 `__array_function__`을 구현해주자.
