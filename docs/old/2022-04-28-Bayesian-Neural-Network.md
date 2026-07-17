---
layout: post
title: "Bayesian Neural Network"
category: 공부하기
---

# 서론

베이지언 신경망과 NNGP에 대해 더 알아보고 싶다.

# 기본 컨셉

베이지언 신경망은 NN의 모든 weight가 전부 결정적인 것이 아닌 확률 분포를 따른다고 생각한다.

모든 weight가 각각 정규분포를 따른다는 것이다.

NNGP는 - VAE할 때 나왔던 그거다. NN의 결과로 평균과 표준편차를 계산해낸다.

# VI로 BNN

[이쪽](http://krasserm.github.io/2019/03/14/bayesian-neural-networks/)의 코드에 구현이 있다.

VAE할 때 나왔던 Reparametrization trick을 매 layer마다 쓰고 있는 모양이다.

각 layer의 weight와 bias가 각각 평균과 표준편차를 가지고 있고

이를 Reparametrization trick을 이용해서 매 학습 때마다 추출해서 쓰는 모양이다.

학습 자체는 엄청 평범하게 한 것 같다.

# MC Dropout으로 BNN

드롭아웃으로??

일단 드롭아웃이라는건 굉장히 흔한 신경망 학습 기법이다. (MC는 몬테카를로의 줄임말인데... 임의로 끊는다는 뜻이다.)

학습하는 동안 신경망 일부를 완전히 끊어두는 것이다. 매 학습 때마다 다른 부분을 끊는다.

그렇게 하면 과적합을 막을 수 있고 기타등등 해서 좋은 결과를 낼 수 있다고 한다.

당연히 학습 때 쓰는 기법이기 때문에 학습 후에는 dropout을 해제한다.

그런데 [어떤 논문](https://arxiv.org/abs/1506.02142)에서 말하길 이 Dropout을 학습 후에도 유지한다면 베이지안 신경망과 비슷한 효과를 낼 수 있다고 한다.

대충 어떤 느낌인가 하면-

Dropout이 켜져 있는 상태에서 모델이 확신을 가지는 부분은 어떻게 모델 중간을 날리더라도 나름 비슷비슷한 결과가 나오겠지만

확실치 않은 부분에서는 Dropout을 어떻게 날리는지에 따라 나오는 값이 천차만별일 것이다.

그렇기 때문에 Dropout을 켜고 여러 번 돌려본 다음 (확률적으로 다르게 나올 테니까) 나온 결과 값들을 확인해보면

이 모델이 얼마나 확신을 가지는지를 알 수 있을 것이다.

# 참고 사이트

링크를 건 것 외에 본문을 적을 때 참고한 사이트는 [여기](https://gaussian37.github.io/dl-concept-bayesian_neural_network/)이다.

# 다음 시간

다음엔 GPNN을 볼 생각이다.

