---
layout: post
title: "AlphaCode"
category: 흥미
---

# AlphaCode

그 전에도 코딩하는 문제를 푸는 머신 러닝은 있었다. GPT-3라던가.

딥마인드에서도 그런 문제를 푸는 무언가를 만들었나 보다. 이름하여 [AlphaCode](https://deepmind.com/blog/article/Competitive-programming-with-AlphaCode).

아무래도 얘내는 Alpha를 정말 좋아하는 모양이다.

대충 블로그 글을 읽어보고, 간단한 정리와 소감을 적어보자.

# 정리

기본적으로는 일반적인 언어 모델과 같다.

모델 자체는 일반적인 Transformer를 쓴다. 애매하다 싶으면 가져다 놓으면 성능이 보장되는 엄청 좋은 모델이다.

문제는 'Competitive programming'이다. 이게 뭔가 하니 백준같은거다. 즉 문제가 긴 글로 설명이 주어지고, 거기에 간단한 예제가 보여지며, 이에 맞게 코딩을 해야 하는 것이다.

다음과 같이 구성되어 있다.

![img](https://kstatic.googleusercontent.com/files/744ac5325ebfe5ea78651e3c498643d989125caf53abf31d5ece15691247c5f48bd92c95aa9dd7bfdfe4f4026ac2ad69c51cf728935f7debdf2ee2695a2ad05a)

기본적으로 GitHub의 코드를 가지고 사전 학습을 한다.

이후 잘 선택된 문제-해답 쌍을 통해 Fine-Tuning을 해서 모델을 준비한다.

이후 질문을 받으면, 거기서 많은 해답을 내놓은 후, 이를 한번 필터링/클러스터링하여 후보를 준비하고, 이를 테스트해서 정확한 점수를 구한다.

대충 훑어본 [논문](https://storage.googleapis.com/deepmind-media/AlphaCode/competition_level_code_generation_with_alphacode.pdf) (아직 출판 안됨)에서 가장 눈에 띄는 것 중 하나는 Loss의 문제인 것 같다.

이런 문제를 One-Of-Many 문제라고 부르는 모양인데, 답을 여럿 내놓고 그 중 하나만 맞으면 맞는 것으로 치는 문제이다.

보통 학습할 때 Loss를 Train set에만 적용하는 것이 아닌 Validation set에도 적용해서 현재 학습의 진행 정도를 추산하는데, Train set의 Loss는 감소하지만 Validation set의 Loss는 증가한다면 과적합 (Overfitting)의 징후로 본다.

하지만 실제로 Validation set의 정확도 역시 증가하고 있었기에, 과적합의 징후는 보이지만 과적합은 일어나지 않고 있었다.

왜 Validation set의 Loss가 학습 진행 정도를 제대로 반영하지 못한 것일까?

One-of-Many 문제에 나타날 수 있는 문제라고 하는데, 말했다시피 여럿 중 하나만 맞으면 맞는 것으로 치는 문제이기 때문에 다른 '해답'들이 Loss를 올리는 것이다.

다음과 같은 예를 들어보자.

단순하게 정렬하는 문제가 있다고 해 보자. 해답 코드로 제시된 것은 QuickSort로 구현된 코드이다. Loss는 이 QuickSort 코드와 얼마나 비슷한지를 본다.

내놓은 여러 해답 중 하나는 QuickSort일 것이다. (비슷한 문제에 대해 학습할 때에도 QuickSort에 대해 학습했을 것이다.) 그 해답에 대한 Loss는 낮다.

하지만 다른 해답에는 BubbleSort라던가 Selection Sort 등 다른 정렬 알고리즘도 있을텐데, 이는 Loss를 높이는 역할을 한다.

특히 학습이 진행되면 진행될 수록 해당 '다른 알고리즘'도 제 모습을 갖출텐데, 그럴 수록 Loss는 더욱 커질 것이다.

이렇듯 Loss가 과적합의 표지로 사용될 수 없는 것이 특징이라고 한다.

결과는 대충 평균적인 인간 프로그래머 (53%)정도의 성적을 낸다고 한다.

# 소감

확실히 많이 발전했다는 것이 느껴진다.

이전에는 '머신 러닝이 프로그래밍을 한다'는 개념 자체가 말이 안된다고 생각했었는데, 점점 말이 되어간다.

이젠 대충 백준같은 것도 절반쯤 풀 수 있을 정도라고 하니, 놀라울 따름이다.

또 논문을 훑어 보면서 새로운 것을 알 수 있었다.
