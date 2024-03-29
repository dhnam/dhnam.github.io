---
layout: post
title: "Autograd 만들기 11"
category: 만들기
---

# 서론

Squeeze를 구현하자. 일단 그 정도면 Autograd는 다 만들었다고 할 수 있을 것 같다.

# Squeeze 구현

어려운건 어제 다 해뒀다. 그냥 형태에 맞춰서 구현해주기만 하면 된다.

# 코드 개선?

사실 이 정도면 만들 코드는 다 만들었고 '끝!' 해도 되기는 하는데 (그리고 내일부터는 다른걸 만들고)

솔직히 조금 더 뭔가 건드릴 수 있는 부분은 없을까 싶다.

당장 생각나는건 --maker계열 함수 세가지를 일종의 틀로 만들어서 찍어내는 방법인데...

'틀로 만들어서 찍어낸다'라고 하면 보통 클래스를 만들고, 그 인스턴스들을 만드는거인데 이 쪽은 안될 것 같다.

어떻게 하던 결국 내부 구현 함수는 만들어줘야 하기 때문이다.

사실 다른 Func계열에도 쓴 방법인데... (이걸 더 이상 클래스라고 부를 수 있는지는 제쳐두고) `__new__`를 적절히 손봐주면 클래스를 그냥 함수 마냥 쓸 수가 있다.

파이썬의 객체지향은 syntactic sugar처럼 작동하는 경향이 있어서 뭔가 엄밀한 의미를 가진다기 보다는 '--하고 써 있는건 사실 --를 불러서 작동한다'하는 경향이 있다.

`__new__` 역시 예외가 아니라서, 클래스 이름 뒤에 `()`를 붙여서 클래스를 인스턴스화 할 때 불려서 인스턴스를 만든다.

(정확히는 메타클래스의 `__call__`이 불리고, 그 안에서 클래스의 `__new__`가 불리는 식이다. 정확히 `__init__`까지 `__call__`안에서 불린다.)

그래서 `__new__`(나 메타클래스의 `__call__`)을 잘 덮어씌워주면 클래스를 완전히 함수처럼 부를 수가 있어진다.

생각해보니까 전부 `FuncMeta`를 메타클래스로 쓰는데, `__new__`를 어떻게 하는 대신 메타클래스의 `__call__`을 건드려주는게 보기 좋지 않을까?

아무튼 그건 그렇고, 지금의 `_class_maker` 함수들을 시그니처를 그대로 유지한 채로 클래스로 바꾸고, 메타클래스를 가지고 잘 놀아주면 좀 더 구조적인 무언가를 만들 수 있을 것이다.

편의상 ArrayFunc와 Func의 차이를 없애자.

그리고 나서 `FuncClassMaker`라는 새로운 클래스를 만들자. 여기에 구현해야 할 함수들을 명시해둔다.

그 후 모든 `ClassMaker`계열에 동일하게 쓰이는 부분은 `FuncClassMaker`에 final로 짜둔다.

새로 구현해야 할 함수는 다음과 같다.

* `__init__`: 이건 굳이 abstract로 해두지 말자.
* `args_to_class_name`: 클래스의 이름을 만든다.
* `args_to_func_name`: Func의 이름을 만든다.
* `make_forward`: 주어진 매개변수로 `forward` 함수를 만든다.
* `make_backward`: 주어진 매개변수로 `backward` 함수를 만든다.

그렇게 하고 나면 - 이 사이에도 또 시행착오가 꽤 있었지만 - 나름 보기 좋은 클래스로 구현이 된다.

# 맺음말

나중에 무언가 문제를 발견하지 않는 한, Autograd 구현은 이걸로 종료일 것 같다.

뭔가 문제가 있거나, 아니면 코드를 더 깔끔하게 만들거나, 아마 둘 중 하나가 아니면 이게 Autograd 만들기 시리즈의 끝일 것 같다.

다음엔... Loss 함수들을 공부하면서 만들어볼까 생각중이다.
