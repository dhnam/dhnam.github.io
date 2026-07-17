---
layout: post
title: "Autograd 만들기 2"
category: 만들기
---

# 서론

이전에 이어서 계속 만들어갈 것이다.

# 시행착오 기록

공식 유저 가이드대로 했는데 다른 결과가 나온다?

공식 가이드는 이렇다.

`__array_ufunc__`는 조금 까다로운 것 같아서 쉬운 `__array_wrap__`을 먼저 해보기로 했다.

```python
import numpy as np

class MySubClass(np.ndarray):

    def __new__(cls, input_array, info=None):
        obj = np.asarray(input_array).view(cls)
        obj.info = info
        return obj

    def __array_finalize__(self, obj):
        print('In __array_finalize__:')
        print('   self is %s' % repr(self))
        print('   obj is %s' % repr(obj))
        if obj is None: return
        self.info = getattr(obj, 'info', None)

    def __array_wrap__(self, out_arr, context=None):
        print('In __array_wrap__:')
        print('   self is %s' % repr(self))
        print('   arr is %s' % repr(out_arr))
        # then just call the parent
        return super().__array_wrap__(self, out_arr, context)
```

이러면 예상하는대로, 적당한 계산 결과와 프롬프트가 뜨는 결과가 나온다.

내가 짠 코드는 이렇다.

```python
class Tensor(np.ndarray):

    def __new__(cls, array):
        print("__new__ called for Tensor")
        obj = np.asarray(array).view(cls)
        return obj

    def __array_wrap__(self, out_arr, context=None):
        print(f"__array_wrap__ Called\n{self=}\n{out_arr=}\n{context=}")
        return super().__array_wrap__(self, out_arr, context)

    def __array_finalize__(self, obj):
        print("__array_finalize__")
        print(f"self type: {type(self)}")
        print(f"obj type: {type(obj)}")
```

그랬는데 계산 결과가 제대로 나오지 않는다.

정확히는 a와 b를 계산한다고 하면 어떻게 계산하던 a만이 나온다.

일단은 위 코드에서 `super().__array_wrap__(self, out_arr, context)`에서 self를 빼서 해결했다.

Tensor는 내부에 CalcGraph를 가지고 있어야 한다.

## 행렬곱의 미분

코드에 이것 저것 끼워맞추다 보니 알아야 할 필요가 생긴 것이 있다.

행렬 연산을 미분하면 어떻게 될 것인가 하는 일이다.

### 야코비안 (자코비안)

f(스칼라) = 스칼라 형태의 공식을 미분하면 나오는 값은 스칼라 하나이다.

f(벡터) = 스칼라 형태의 공식을 미분하면 그라디언트가 나온다. (벡터이다.)

f(벡터) = 벡터 형태의 공식을 미분하면 그라디언트보다 차원이 높은 무언가가 나오지 않을까?

f(벡터) = 벡터 형태의 공식을 미분해서 나오는 행렬을 '야코비 행렬' 이라고 부른다.

## 다시 돌아와서

그렇다면 행렬곱의 미분은 어떻게 될까?

f(행렬) = 행렬 형태...라면 좀 더 차원이 높아져야 하지 않을까? 싶다.

하지만 행렬곱의 미분은 다행히도 이것 보다는 조금 간단해지는 것 같다.

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7DA_%7B1,1%7D%20&%20%5Ccdots%20&%20A_%7B1,%20n%7D%20%5C%5C%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5CA_%7Bm,%201%7D%20&%20%5Ccdots%20&%20A_%7Bm,%20n%7D%20%5C%5C%5Cend%7Bbmatrix%7D%5Cbegin%7Bbmatrix%7DB_%7B1,1%7D%20&%20%5Ccdots%20&%20B_%7B1,%20k%7D%20%5C%5C%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5CB_%7Bn,%201%7D%20&%20%5Ccdots%20&%20A_%7Bn,%20k%7D%20%5C%5C%5Cend%7Bbmatrix%7D)

이런 행렬곱을 생각해보자.

각각 m x n차원과 n x k차원의 행렬이고, 결과는 m x k 차원의 행렬이 나온다.

A에서 임의의 i번째 행을 꺼내보자.

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7DA_%7Bi,%201%7D%20&%20%5Ccdots%20&%20A_%7Bi,%20n%7D%20%5C%5C%5Cend%7Bbmatrix%7D%5Cbegin%7Bbmatrix%7DB_%7B1,1%7D%20&%20%5Ccdots%20&%20B_%7B1,%20k%7D%20%5C%5C%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5CB_%7Bn,%201%7D%20&%20%5Ccdots%20&%20A_%7Bn,%20k%7D%20%5C%5C%5Cend%7Bbmatrix%7D)

1 x n차원과 n x k차원의 행렬곱이고, 1 x k차원의 행렬이 결과로 나온다.

이 때 이 연산을 앞쪽 벡터 (1 x n차원 행렬)에 대해 미분하면, 야코비안은 어떻게 나올까?

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7DA_%7Bi,%201%7D%20&%20%5Ccdots%20&%20A_%7Bi,%20n%7D%20%5C%5C%5Cend%7Bbmatrix%7D%5Cbegin%7Bbmatrix%7DB_%7B1,1%7D%20&%20%5Ccdots%20&%20B_%7B1,%20k%7D%20%5C%5C%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5CB_%7Bn,%201%7D%20&%20%5Ccdots%20&%20A_%7Bn,%20k%7D%20%5C%5C%5Cend%7Bbmatrix%7D%20=%20%5Cbegin%7Bbmatrix%7D(A_%7Bi,%201%7DB_%7B1,%201%7D%20&plus;%20A_%7Bi,%202%7DB_%7B2,%201%7D%20&plus;%20%5Ccdots%20&plus;%20A_%7Bi,n%7DB_%7Bn,1%7D)%20&%20%5Ccdots%20&%20(A_%7Bi,1%7DB_%7B1,k%7D%20&plus;%20A_%7Bi,2%7DB_%7B2,k%7D%20&plus;%20%5Ccdots%20&plus;%20A_%7Bi,n%7DB_%7Bn,k%7D)%20%5C%5C%5Cend%7Bbmatrix%7D)

보기 좋게 계산 결과도 써놓자.

야코비 행렬은 다음과 같다. f(X) = Y라고 할 때...

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7D%5Cfrac%7B%5Cpartial%20y_%7B1%7D%7D%7B%5Cpartial%20x_%7B1%7D%7D%20&%20%5Ccdots%20&%20%5Cfrac%7B%5Cpartial%20y_%7B1%7D%7D%7B%5Cpartial%20x_%7Bn%7D%7D%20%5C%5C%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5C%5Cfrac%7B%5Cpartial%20y_%7Bm%7D%7D%7B%5Cpartial%20x_%7B1%7D%7D%20&%20%5Ccdots%20&%20%5Cfrac%7B%5Cpartial%20y_%7Bm%7D%7D%7B%5Cpartial%20x_%7Bn%7D%7D%20%5C%5C%5Cend%7Bbmatrix%7D)

즉 Y의 각 항목에 대한 그라디언트 (열벡터)를 쌓아놓은 형태다.

이를 바탕으로 계산해보면, 야코비안의 첫 행은 다음과 같다.

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7DB_%7B1,%201%7D%20&%20B_%7B2,%201%7D%20&%20%5Ccdots%20&%20B_%7Bn,%201%7D%20%5C%5C%5Cvdots%20&%20%5Cvdots%20&%20%5Cvdots%20&%20%5Cvdots%5Cend%7Bbmatrix%7D)

...많이 익숙한 모양이 아닌가?

나머지에 대해서도 쭉 계산해보면

![eq](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7DB_%7B1,%201%7D%20&%20B_%7B2,%201%7D%20&%20%5Ccdots%20&%20B_%7Bn,%201%7D%20%5C%5CB_%7B1,%202%7D%20&%20B_%7B2,%202%7D%20&%20%5Ccdots%20&%20B_%7Bn,%202%7D%20%5C%5C%5Cvdots%20&%20%5Cvdots%20&%20%5Cddots%20&%20%5Cvdots%20%5C%5CB_%7B1,%20k%7D%20&%20B_%7B2,%20k%7D%20&%20%5Ccdots%20&%20B_%7Bn,%20k%7D%5Cend%7Bbmatrix%7D%20=%20B%5E%7BT%7D)

그렇다. B 행렬의 전치행렬이 임의의 행에 대한 야코비안이 되는 것이다.

그렇기에 행렬곱을 미분하면 전치행렬이 된다고 봐도 좋을 것이다. 차원도 딱 맞고...


