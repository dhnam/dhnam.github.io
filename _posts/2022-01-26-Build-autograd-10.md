---
layout: post
title: "Autograd 만들기 10"
category: 만들기
---

# 서론

이전에 말한대로 `__array_function__`의 구현부터 시작하자.

# `__array_function__`과 함수 구현들

`__array_function__`으로 구현될 함수들은 따로 새 파일을 만들어서 빼주자.

그리고 `__array_function__` 그 자체의 구현은 [여기](https://numpy.org/doc/stable/reference/arrays.classes.html)를 참조하자.

일단 `__array_function__`에 어떤 값이 들어오는지를 알아보기로 했다.

[[1, 2], [3, 4]]와 [[5, 6], [7, 8]]을 axis 1로 concatinate하는 코드를 짰고, `__array_function__`의 매개변수 (`func`, `types`, `args`, `kwargs`)를 출력해보았다.

다음과 같은 결과가 나온다.

```
<function concatenate at 0x000002388B5B3BE0> (<class '__main__.Tensor'>, <class 'numpy.ndarray'>) ([Tensor([[1, 2],
        [3, 4]]), array([[5, 6],
       [7, 8]])],) {'axis': 1}
<function copy at 0x000002388B7DFBE0> (<class '__main__.Tensor'>,) (Tensor([[1, 2, 5, 6],
        [3, 4, 7, 8]]),) {}
```

concatenate 내부에서 copy도 부른다는 부가적인 정보도 알 수 있었다.

함수 내부 구현은 [아까 거기](https://numpy.org/doc/stable/reference/arrays.classes.html)를 참고하면 될 것이다.

`__array_function__`으로 불리는 함수들을 구현할 클래스는 `ArrayFunc`라는 이름으로 만들어주자. 메타클래스는 FuncMeta다.

# CalcGraph 확장

이것 저것 생각할게 많은데, 그 중에 하나는 아무래도 CalcGraph인 것 같다.

지금 계산 그래프는 kwargs를 처리하지 못한다.

그나마 비슷한게 필요한 Broadcast는 동적으로 클래스를 생성해서 처리하고 있지만.

일단 CalcGraph의 `__init__`으로 kwargs를 받게 한 다음 (`**kwargs`) CalcGraph 내부에 저장해두자.

# FuncConcat: 클래스 동적 생성

예전에 Broadcast에서 했던 것 처럼 아무래도 FuncConcat도 동적으로 클래스를 만들어야 할 것 같다.

backward에서 어떻게 concat됐는지에 대한 정보가 있어야 제대로 split해줄 수 있기 때문이다.

생각해보니 Squeeze도 마찬가지.

# Copy 문제

Copy를 하고 나면 계산 그래프가 완전히 날아간다.

이는 `__array_finalize__`를 손보는 것으로 해결할 수 있었다.

# 다음 시간

여기 적은 것 말고도 이것 저것 생각보다 할게 많았다.

다음엔 Squeeze를 만들자.
