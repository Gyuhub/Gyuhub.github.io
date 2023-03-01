---
layout: post
title: Learning-Techniques-3
category: deep learning
post-series: Deep learning from scratch
post-order: 18
---

이전 [post](https://gyuhub.github.io/posts/study/machine%20learning/deep%20learning/learning-techniques-2)에서는 **가중치 초깃값**과 **배치 정규화**에 대해서 배웠습니다. 이번 post에서는 신경망 학습의 **오버피팅**을 억제하는 기술에 대해서 알아보겠습니다.

---

# 오버피팅

기계학습에서는 **오버피팅**이 문제가 되는 일이 많습니다. 오버피팅이란 신경망이 **훈련 데이터**에만 지나치게 적응되어 그 외의 데이터에는 제대로 대응하지 못하는 상태를 말합니다. 기계학습은 범용적인 성능을 지향합니다. 훈련 데이터에는 포함되지 않는, **아직 보지 못한 데이터**가 주어져도 바르게 식별해내는 모델이 바람직합니다. 복잡하고 표현력이 높은 모델을 만들 수는 있지만, 그만큼 오버피팅을 억제하는 기술이 중요해지는 것입니다.

오버피팅은 주로 다음의 두 경우에 일어납니다.
* 매개변수가 **많고** 표현력이 **높은** 모델
* 훈련 데이터가 **적은** 경우

위 두 조건을 충족하면 어떤 일이 발생할까요? 실제로 두 조건을 충족하여 오버피팅을 **일부러** 일으켜보겠습니다. 원래 60,000개의 훈련 데이터로 이루어진 MNIST 데이터셋에서 **500개**만 사용해서 훈련 데이터를 줄이고, **7층** 네트워크를 사용해서 네트워크의 복잡성을 높이겠습니다. 각 층의 뉴런은 100개, 활성화 함수는 ReLU를 사용하겠습니다. 결과는 다음과 같습니다.

<figure>
    <img src="/posts/study/machine learning/deep learning/images/learning_techniques_28.png"
         title="Comparison of accuracy between train and test dataset when overfitting occurred"
         alt="Image of comparison of accuracy between train and test dataset when overfitting occurred"
         class="img_center"
         style="width: 60%"/>
    <figcaption>오버피팅이 발생하는 조건을 충족했을때 훈련 데이터와 시험 데이터의 정확도 비교</figcaption>
</figure>

[Fig. 1.]을 보시면 훈련 데이터의 경우 약 75 데폭을 지나는 무렵부터 거의 100%의 정확도를 기록했습니다. 반면에 시험 데이터의 경우에는 시간이 흘러도 80% 이상의 정확도는 넘어서지 못하는 모습을 보입니다. 이처럼 **훈련 데이터**에만 적응해버리는 것이 바로 **오버피팅**입니다.

## 가중치 감소

오버피팅을 억제하기 위한 방법으로 예로부터 많이 이용해온 **가중치 감소**(weight decay)라는 것이 있습니다. 이는 학습 과정에서 **큰** 가중치에 대해서는 그에 상응하는 **큰 페널티**를 부과하여 오버피팅을 억제하는 방법입니다. 원래 오버피팅은 가중치 매개변수의 *값이 커서* 발생하는 경우가 많기 때문입니다.

신경망 학습의 목적은 <ins>손실 함수의 값을 줄이는 것</ins>입니다. 이때, 예를 들어 가중치의 **제곱 노름**(L2 노름, L2 norm)을 손실 함수에 더합니다. 그렇다면 가중치가 커지는 것을 억제할 수 있겠죠. 가중치를 $\boldsymbol{W}$라 하면 L2 노름에 따른 가중치 감소는 $\frac{1}{2}\lambda\boldsymbol{W}^2$이 되고, 이 $\frac{1}{2}\lambda\boldsymbol{W}^2$을 손실 함수에 더합니다. 여기에서 $\lambda$는 정규화의 세기를 조절하는 **하이퍼파라미터**입니다. $\lambda$를 **크게** 설정할수록 **큰 가중치**에 대한 페널티가 커집니다. $\frac{1}{2}\lambda\boldsymbol{W}^2$에서 $\frac{1}{2}$은 $\frac{1}{2}\lambda\boldsymbol{W}^2$의 미분 결과인 $\lambda\boldsymbol{W}$를 조정하는 역할의 상수입니다.

가중치 감소는 모든 가중치 각각의 손실 함수에 $\frac{1}{2}\lambda\boldsymbol{W}^2$을 더합니다. 따라서 가중치의 기울기를 구하는 계산에서는 그동안의 오차역전파법에 따른 결과에 정규화 항을 미분한 $\lambda\boldsymbol{W}$를 더합니다.

> 💡 L2 노름은 **각 원소의 제곱**을 더한 것에 해당합니다. 가중치 $W=\begin{pmatrix} w_1 & w_2 & \cdots & w_n \end{pmatrix}$ 이 있다면, L2 노름에서는 $\sqrt{w_1^2+w_2^2+\cdots+w_n^2}$으로 계산할 수 있습니다. L2 노름 외에 L1 노름과 L$\infty$ 노름도 있습니다. L1 노름은 절댓값의 합, 즉 $\left \vert w_1 \right \vert+\left \vert w_2 \right \vert+\cdots+\left \vert w_n \right \vert$에 해당합니다. L$\infty$ 노름은 Max 노름이라고도 하며, 각 원소의 절댓값 중 가장 큰 것에 해당합니다. 정규화 항으로 L2 노름, L1 노름, L$\infty$ 노름 중 어떤 것도 사용할 수 있습니다.

그러면 가중치 감소를 한번 적용해보겠습니다. [Fig. 1.]에서는 $\lambda=0.1$의 가중치 감소를 적용합니다. 결과는 다음과 같습니다.

<figure>
    <img src="/posts/study/machine learning/deep learning/images/learning_techniques_29.png"
         title="Comparison of accuracy between train and test dataset when applied weight decay method"
         alt="Image of comparison of accuracy between train and test dataset when applied weight decay method"
         class="img_center"
         style="width: 60%"/>
    <figcaption>가중치 감소를 적용한 훈련 데이터와 시험 데이터의 정확도 비교</figcaption>
</figure>

가중치 감소를 <ins>적용하지 않은</ins> 신경망의 <span style="color: blue">훈련 데이터</span>와 <span style="color: green">시험 데이터</span>의 정확도 차이보다 가중치 감소를 <ins>적용한</ins> 신경망의 <span style="color: purple">훈련 데이터</span>와 <span style="color: pink">시험 데이터</span>의 정확도 차이가 **더 작은** 것을 확인할 수 있습니다. 다시 말해 가중치 감소를 통해서 **오버피팅**을 **억제**하는 것에 성공했다는 말입니다. 하지만, 앞선 경우와 달리 훈련 데이터에 대한 정확도가 **100%**에 도달하지 못한 점도 주목해야 하겠습니다.

---

## 드롭아웃

오버피팅을 억제하는 방식으로 손실 함수에 가중치의 L2 노름을 더하는 **가중치 감소** 방법을 설명했습니다. 가중치 감소는 **구현도 간단하고** 어느 정도 **지나친 학습**을 억제하는데 도움이 된다는 것도 확인했습니다. 하지만 신경망이 복잡해질수록 가중치 감소만으로는 오버피팅에 대응하기 어려워집니다. 이때 사용하는 기법이 바로 **드롭아웃**(Dropout)입니다.

**드롭아웃**은 뉴런을 **임의로 삭제**하면서 학습하는 방법입니다. 훈련 때 **은닉층**의 뉴런을 **무작위로** 골라 삭제합니다. 삭제된 뉴런은 아래 그림과 같이 신호를 전달하지 않게 됩니다. **훈련** 때는 데이터를 흘릴 때마다 삭제할 뉴런을 **무작위로** 선택하고, **시험** 때는 **모든** 뉴런에 신호를 전달합니다. 단, **시험 때는** 각 뉴런의 **출력**에 훈련 때 **삭제 안 한 비율**을 곱하여 출력합니다.

<figure>
    <img src="/posts/study/machine learning/deep learning/images/learning_techniques_30.png"
         title="Comparison of networks within dropout"
         alt="Image of comparison of networks within dropout"
         class="img_center"
         style="width: 60%"/>
    <figcaption>드롭아웃의 개념. 일반적인 신경망(좌)과 드롭아웃을 적용한 신경망(우)</figcaption>
</figure>

그러면 드롭아웃을 한번 적용해보겠습니다. [Fig. 4.]에서는 **0.12**의 드롭아웃 비율을 적용합니다. 결과는 다음과 같습니다.

<figure>
    <img src="/posts/study/machine learning/deep learning/images/learning_techniques_31.png"
         title="Comparison of accuracy between train and test dataset when applied dropout method"
         alt="Image of comparison of accuracy between train and test dataset when applied dropout method"
         class="img_center"
         style="width: 60%"/>
    <figcaption>드롭아웃을 적용한 훈련 데이터와 시험 데이터의 정확도 비교</figcaption>
</figure>

[Fig. 4.]와 같이 드롭아웃을 적용하니 훈련 데이터와 시험 데이터에 대한 정확도 차이가 **줄었습니다**. 또한, **가중치 감소**와 마찬가지로 훈련 데이터에 대한 정확도가 **100%**에 도달하지도 않게 되었습니다. 이처럼 드롭아웃을 이용하면 표현력을 높이면서도 오버피팅을 억제할 수 있습니다.

> 📈 기계학습에서는 **앙상블 학습**(ensemble learning)을 애용합니다. **앙상블 학습**은 개별적으로 학습시킨 여러 모델의 출력을 평균을 내거나 투표 등의 방법으로 추론하는 방식입니다. 신경망의 맥락에서 얘기하면, 가령 같은 (혹은 비슷한) 구조의 네트워크를 5개 준비하여 따로따로 학습시키고, 시험 때는 그 5개의 출력을 평균 내어 답하는 것입니다. 앙상블 학습을 수행하면 신경망의 정확도가 몇% 정도 **개선된다는** 것이 실험적으로 알려져 있습니다. 

> 앙상블 학습과 **드롭아웃**은 밀접합니다. 드롭아웃이 학습 때 뉴런을 무작위로 삭제하는 행위를 <ins>매번 다른 모델을 학습시키는 것</ins>으로 해석할 수 있기 때문입니다. 그리고 추론 때는 뉴련의 출력에 삭제한 비율을 곱함으로써 앙상블 학습에서 <ins>여러 모델의 평균을 내는 것</ins>과 같은 효과를 얻는 것입니다. 즉, 드롭아웃은 앙상블 학습과 같은 효과를 **하나의 네트워크**로 구현했다고도 생각할 수 있습니다.

---

# 적절한 하이퍼파라미터 값 탐색

신경망에는 **하이퍼파라미터**(hyper parameter)가 다수 등장합니다. 하이퍼파라미터란 예를 들어 <ins>각 층의 뉴런 수, 배치의 크기, 매개변수 갱신 시의 학습률과 가중치 감소</ins> 등입니다. 이러한 하이퍼파라미터의 값을 적절히 설정하지 않으면 모델의 성능이 크게 떨어지기도 합니다. 그만큼 하이퍼파라미터의 값은 매우 **중요**하지만 그 값을 결정하기까지는 일반적으로 많은 시행착오를 겪습니다. 따라서 이번에는 적절한 하이퍼파라미터의 값을 최대한 효율적으로 탐색하는 방법에 대해서 알아보겠습니다.

## 검증 데이터

지금까지는 주어진 데이터셋을 학습을 위한 **훈련 데이터**와 범용 성능 평가를 위한 **시험 데이터**로 분리해서 이용했습니다. 이를 통해 훈련 데이터에만 지나치게 적응되어 있지 않은지(오버피팅이 발생한 건 아닌지), 그리고 범용 성능은 어느 정도인지 같은 것을 평가할 수 있었습니다. 하이퍼파라미터가 적절한 지 다양한 값으로 설정하고 검증할 텐데, 여기서 주의할 점은 하이퍼파라미터의 성능을 평가할 때는 <ins>시험 데이터를 사용해서는 안 된다든 것</ins>입니다.

같은 **성능 평가**인데 하이퍼파라미터가 대상일 때는 시험 데이터를 사용해서는 안 되는 이유가 무엇일까요? 그것은 시험 데이터를 사용하여 하이퍼파라미터를 조정하면 하이퍼파라미터 값이 시험 데이터에 **오버피팅**되기 때문입니다. 그렇게 되면 다른 데이터에는 적응하지 못하니 범용 성능이 떨어지는 모델이 될지도 모릅니다. 그래서 하이퍼파라미터를 조정할 때는 **하이퍼파라미터 전용 확인 데이터**가 필요합니다. 하이퍼파라미터 조정용 데이터를 일반적으로 **검증 데이터**(validation data)라고 부릅니다.

> ✨ 데이터셋을 용도에 따라서 분류하면 아래와 같습니다.<br>
> * 훈련 데이터: **매개변수**(가중치와 편향) 학습
> * 검증 데이터: **하이퍼파라미터** 성능 평가
> * 시험 데이터: 신경망의 **범용 성능** 평가

데이터셋에 따라서는 훈련, 검증, 시험 데이터를 미리 분리해둔 것도 있습니다. 하지만 그렇지 않은 경우도 있습니다. MNIST 데이터셋은 훈련 데이터와 시험 데이터로만 분리해뒀습니다. 이런 경우에는 사용자가 직접 데이터를 분리해서 사용하면 됩니다. 이어서 검증 데이터를 사용해서 하이퍼파라미터를 최적화하는 기법에 대해서 살펴보겠습니다.

## 하이퍼파라미터 최적화

하이퍼파라미터를 최적화할 때의 핵심은 하이퍼파라미터의 "**최적 값**"이 존재하는 **범위**를 조금씩 <ins>줄여간다는</ins> 것입니다. 범위를 조금씩 줄이려면 우선 1. 대략적인 범위를 설정하고 2. 그 범위에서 무작위로 하이퍼파라미터 값을 골라낸(샘플링) 후, 3. 그 값으로 정확도를 평가합니다. 정확도를 잘 살피면서 이 작업을 여러 번 반복하며 하이퍼파라미터의 "최적 값"을 범위를 좁혀가는 것입니다.

> 📜 신경망의 하이퍼파라미터 최적화에서는 **그리드 서치**(grid search)같은 **규칙적인** 탐색보다는 **무작위로 샘플링해** 탐색하는 편이 **좋은** 결과를 낸다고 알려져 있습니다[^fn-hyper-parameter-optimization]. 이는 최종 정확도에 미치는 영향력이 하이퍼파라미터마다 **다르기** 때문입니다.

하이퍼파라미터의 범위는 '**대략적으로**' 지정하는 것이 효과적입니다. 실제로도 0.001에서 1,000 사이 $(10^{-3} \sim 10^{3})$와 같이 **10의 거듭제곱** 단위로 범위를 지정합니다. 이를 **로그 스케일**(log scale)로 지정한다고 합니다. 하이퍼파라미터를 최적화할 때는 딥러닝 학습에는 **오랜 시간**(며칠이나 몇 주 이상)이 걸린다는 점을 기억해야 합니다. 따라서 나쁠 듯한 값은 *일찍 포기하는* 게 좋습니다. 그래서 학습을 위한 에폭을 *작게* 하여, 1회 평가에 걸리는 시간을 **단축**하는 것이 효과적입니다.

이상이 하이퍼파라미터의 최적화입니다. 이를 정리하면 다음과 같습니다.

* 0단계
  * 하이퍼파라미터 값의 **범위**를 설정합니다.
* 1단계
  * 설정된 범위에서 하이퍼파라미터의 값을 **무작위로** 추출합니다.
* 2단계
  * 1단계에서 샘플링한 하이퍼파라미터 값을 사용하여 **학습**하고, **검증 데이터**로 정확도를 평가합니다(단, 에폭은 작게 설정합니다).
* 3단계
  * 1단계와 2단계를 특정 횟수(100회 등) 반복하며, 그 정확도의 결과를 보고 하이퍼파라미터의 범위를 **좁힙니다**.

이상을 반복하여 하이퍼파라미터의 범위를 좁혀가고, 어느 정도 좁아지면 그 압축한 범위에서 값을 하나 **골라냅니다**. 이것이 하이퍼파라미터를 최적화하는 하나의 방법입니다.

> 🔆 여기에서 설명한 하이퍼파라미터 최적화 방법은 실용적인 방법입니다. 더 세련된 기법을 원한다면 **베이즈 최적화**(Bayesian optimization)[^fn-bayesian-optimization]를 소개할 수 있겠습니다. 베이즈 최적화는 **베이즈 정리**(Bayes' theorem)를 중심으로 한 수학 이론을 구사하여 더 **엄밀하고 효율적으로** 최적화를 수행합니다.

그러면 하이퍼파라미터 최적화를 한번 적용해보겠습니다. [Fig. 5.]에서는 가중치 감소 계수를 $10^{-8} \sim 10^{-3}$, 학습률을 $10^{-6} \sim 10^{-1}$ 범위부터 시작했습니다. 결과는 다음과 같습니다.

<figure>
    <img src="/posts/study/machine learning/deep learning/images/learning_techniques_32.png"
         title="Comparison of accuracy between train and validation dataset within hyper parameter optimization"
         alt="Image of comparison of accuracy between train and validation dataset within hyper parameter optimization"
         class="img_center"
         style="width: 75%"/>
    <figcaption>하이퍼파라미터 최적화를 통한 훈련 데이터와 검증 데이터의 정확도 비교</figcaption>
</figure>

[Fig. 5.]에서는 검증 데이터의 학습 추이를 정확도가 높은 순서로 "Best 6"까지 나열했습니다. 이를 바탕으로 하이퍼파라미터의 값(학습률과 가중치 감소 계수)을 살펴보겠습니다. 결과는 다음과 같습니다.

```bash
Best 1 (0.89): lr:0.05567252269169351, weight decay:1.4486411822468745e-07
Best 2 (0.88): lr:0.025982059274246065, weight decay:2.399166472542664e-08
Best 3 (0.88): lr:0.03266666951643395, weight decay:0.00039864344919228147
Best 4 (0.88): lr:0.08622005359644884, weight decay:7.461699683307732e-08
Best 5 (0.87): lr:0.09742737935246511, weight decay:1.7644865261190753e-05
Best 6 (0.87): lr:0.04728076439891178, weight decay:3.874235915438321e-07
```

위 결과를 보면 학습이 잘 진행될 때의 학습률은 $0.01 \sim 0.1$, 가중치 감소 계수는 $10^{-8} \sim 10^{-4}$ 정도라는 것을 알 수 있습니다. 이처럼 학습 결과가 좋은 값들의 범위를 관찰하고 범위를 좁혀갑니다. 그런 다음 그 축소된 범위에서 똑같은 작업을 다시 반복하는 겁니다. 이렇게 적절한 값이 위치한 범위를 좁혀가다가 특정 단계에서 최종 하이퍼파라미터의 값을 하나 선택합니다.

지금까지 신경망 학습에 중요한 기술 몇 가지에 대한 기초와 이론, 구현까지 정리해봤습니다. 다음 post에서는 **합성곱 신경망**(Convolutional Neural Network, CNN)에 대해서 다뤄보겠습니다.

## 신경망 학습 관련 기술들 요약
- 매개변수 갱신 방법에는 확률적 경사 하강법(SGD) 외에도 모멘텀, AdaGrad, Adam 등이 있다.
- 가중치 초깃값을 정하는 방법은 올바른 학습을 하는 데 매우 중요하다.
- 가중치의 초깃값으로는 "Xavier 초깃값"과 "He 초깃값"이 효과적이다.
- 배치 정규화를 이용하면 학습을 빠르게 진행할 수 있으며, 초깃값에 영향을 덜 받게 된다.
- 오버피팅을 억제하는 정규화 기술로는 가중치 감소와 드롭아웃이 있다.
- 하이퍼파라미터 값 탐색은 최적 값이 존재할 법한 범위를 점차 좁히면서 하는 것이 효과적이다.

---

[^fn-hyper-parameter-optimization]: James Bergstra and Yoshua Bengio(2012): Random Search for Hyper-Parameter Optimization. Journal of Machine Learning Research 13, Feb(2012), 281-305.

[^fn-bayesian-optimization]: Jasper Snoek, Hugo Larochelle, and Ryan P. Adams(2012): Practical Bayesian Optimization of Machine Learning Algorithms. In F. Pereia, C. J. C. Burges, L. Bottou, & K. Q. Weinberger, eds. Advances in Neural Information Processing Systems 25. Curran Associates, Inc., 2951 - 2959.
