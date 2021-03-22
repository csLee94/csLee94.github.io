---
layout: post
title:  "AI 기초 개요"
subtitle:   ai 기초 개요
categories: datascience
tags: AI
comments: true
use_math: true
---

## 1. AI란?

인공지능이란 기계가 주어진 문제를 사고를 통해 해결하고, 경험을 통해 능력을 향상하는 학습 능력을 갖추도록 하는 기술이라고 정의할 수 있다. 

![img](https://drive.google.com/uc?id=1u_lOBhfe2cAGulK9z2Ri91CrXgeS05ch)

인공지능 이전에 기계에게 결정을 내리도록 하는데 알고리즘을 사용하였다. 소프트웨어가 수신하는 각 유형의 입력값들에 대한 출력을 정의하는 **규칙**을 설정하였고, 기계는 이 규칙을 토대로 입력값들에 대한 출력을 결정하였다. 

반면에 인공지능은 학습된 경험에 의해 결정한다. 입력과 출력 사이에 f(x)를 추론하고, 이를 토대로 새로운 입력값들에 대해서도 학습된 f(x)를 통해 결과를 추론한다. 다시 말해, 인공지능은 **예측**을 한다.  

>인공지능이란 특별한 임무 수행에 인간 대체, 인지능력의 제고, 자연스러운 인간의 의사소통, 복잡한 콘텐츠의 이해, 결론을 도출하는 과정 등 **인간이 수행하는 것을 모방하는 기술**이다. *(Gartner, 2016)*

인공지능이 인간의 일을 얼마나 수행하느냐에 따라서 다음과 같이 분류할 수 있다. 

- ANI (Artificial Narrow Intelligence): 자율성이 없는 특정 능력에만 특화
- AGI (Artificial General Intelligence): Ex.Machina 등에서 나오는 AI로 학습, 추리,적응,논증 기능을 갖춘 자율성이 있다.
- 혹은 약/강/초 인공지능으로 분류하기도 한다.

그러나 현재 산업에서 다루고 있는 인공지능은 모두 ANI에 국한된다. 

## 2. AI는 어떻게 학습하는가?

![img](https://drive.google.com/uc?id=1x6KVtFQXcCWkore-p1vCKr7slbKxRzM0)

1. Machine Learning 

    >A computer program is said to learn from experience E with respect to some task T and some performance measure P, if its performance on t, as measured by P, improves with experience E *(Tom Mitchell, 1988)*

    컴퓨터가 어떤 작업(T)을 하는데 있어서 경험(E)으로부터 학습하여 성능에 대한 측정(P)를 향상시키는 것을 기계학습이라고 정의하고 있습니다. 초기에 기계가 학습하는 방식은 통계에 기반하여 이루어졌다. 다량의 데이터 속에서 '통계적' 유의성을 통해 관계, 패턴등을 파악학고 이를 통해 더 높은 성능을 구현하는 것이 머신러닝이었다.

    >사실 기계학습 알고리즘은 통계 기반 뿐만 아니라 신호 기반, 물체 인식, 패턴 기반 등 조금 더 다양하지만 지금은 편의상 통계 기반으로 뭉뚱그려 표현한다. 

    ![img](https://drive.google.com/uc?id=1PvmfWT1VO3gR7c66Dvphe6uGOlhnHzAg)

    예를 들어 몸무게와 키에 대한 데이터 분포가 위와 같다. 데이터 분포를 토대로  \(y = ax + b\) 과 같은 수식으로 데이터 가운데 직선을 그릴 수 있다. 다만 분포가 완벽한 형태를 띄지 않기 때문에 각 데이터들에 대한 $$e$$ (오차)가 생길 것이고, 컴퓨터는 $$e$$ 의 총합이 최소가 되는 직선을 찾는다. 

    또한 데이터가 쌓일수록 [경험이 증가할수록] 직선의 형태가 변화하며 더 나은 직선을 그리게 된다. 혹은 추가 변수를 통해 직선의 설명력을 향상시킬 수도 있다.

    이렇게 만들어진 직선 $$f(x)$$ 를 통해 기계는 새로운 값 키(ex. 190cm)를 입력받았을 때 약 90kg이라고 '예측'하게 된다.

    > **기계학습과 통계학의 차이**<br>
    통계학이 가설을 검증하는 일과 좀 더 밀접하게 관련돼 있는 반면, 
    기계학습은 가설을 통해 일반화한 프로세스를 공식화하는 작업에 좀 더 연관돼 있다. 

 2. Deep Learning 

    기존 Machine Learning의 학습 방식이 아닌 사람이 학습하는 방식을 차용한 방식이다. 

    2.1 Perceptron

    ![img](https://drive.google.com/uc?id=1G9E-PurPAEma2hgF26C4IyFqVREOonhS)

    사람이나 동물의 신경계를 본 따 만든 학습 알고리즘으로 . Input, Weights, Output, Activation Function으로 나누어 볼 수 있다. Input 된 다수의 신호를 받아 각각 고유한 weight 값을 곱한 값을 활성화함수를 통해 0과 1로 출력한다. 

    >*1943년 Warren McCulloch* : 신경망 연구 중 맥컬론-피츠 모델 발표 (수학적 모델 제시)<br>
    *1958년 Frank Rosenblatt* : 기존 신경망 모델을 발전시켜 실전 문제 접목


    $$
    x = \begin{pmatrix}
    x_1 \\
    x_2 \\
    x_3 \\
    x_4 \\
    ... \\
    x_d \\
    \end{pmatrix}
    \text{ }
    \text{ }
    \text{ }
    \text{ }

    w =\begin{pmatrix}
    w_1 \\
    w_2 \\
    w_3 \\
    w_4 \\
    ... \\
    w_d
    \end{pmatrix}
    $$

    처음에는 임의로 설정된 weight로 시작하여, 퍼셉트론 모형의 분류가 잘못되었을 때, 각 가중치(weight)를 개선해 나간다. 학습하는 과정에서 각각의 입력값( $$x$$ )과 그에 대한 가중치( $$w$$ )를 행렬곱으로 계산하며 많은 연산이 필요하게 되고 이것이 GPU가 필요한 이유가 된다. 

    또한, 가중치가 반영된 입력값을 가지고 bias(편향)을 통해 Output layer에서 활성화 정도를 조절할 수 있다. 출력층에 설정된 임계점( $$\theta$$ )을 넘지 못하면 활성화되지 않게 되는데 활성화 정도를 높이기 위해 편향을 통해 조절할 수 있다. 퍼셉트론을 간단한 수식으로 나타내면 다음과 같다.

    $$\hat{y} = 
    b+\sum_{i=0}^{n} x_i * w_i
    $$

    그러나 퍼셉트론은 [XOR 문제](https://ardino-lab.com/%EB%8B%A8%EC%B8%B5-%ED%8D%BC%EC%85%89%ED%8A%B8%EB%A1%A0%EC%9D%98-%EB%AC%B8%EC%A0%9C%EC%99%80-%ED%95%9C%EA%B3%84/)를 해결하지 못한다는 문제가 있다. 

    2.2 다층 퍼셉트론과 역전파, 사전훈련법이 결합된 Deep Learning

    이 후 퍼셉트론은 Hidden layer를 추가하는 다층 퍼셉트론, 결과를 보고 연결된 가중치를 역으로 조절해나가는 Backpropagation(역전파)를 통해 XOR 문제를 해결하였다. 그러나 Hidden layer를 무작정 많이 쌓다 보니 모델의 복잡도가 높아졌다. 이에 임의로 지정된 초기 $$w$$값을 튜닝할 수 있는 사전훈련법을 도입하여 Deep Learning이라고 명명했다. 

    ![img](https://drive.google.com/uc?id=1JXmth-ipSxscKA-iI-wFn0nU_4id56tP)

    ![img](https://drive.google.com/uc?id=16vOkb45uTpt82kNxcPXdSA-5cbRLMR5V)

    이론 상 퍼셉트론의 층을 거듭 쌓아가면 비선형 표현 뿐만 아니라 컴퓨터가 수행하는 모든 처리도 표현 가능하게 된다. 

3. Machine Learning vs Deep Learning

    3.1 학습하는 방식 

    Machine Learning으로 문제를 해결하려고 할 때는 변수(feature)를 정의해줘야 한다. 예를 들어 사람 얼굴을 인식한다고 할 때, [눈, 코, 입, 각 부위 별 길이 및 비율] 등 사람 얼굴을 잘 표현할 수 있는 변수들을 찾고, 검증을 거쳐야 한다. 

    ![img](https://drive.google.com/uc?id=1yc-8v9XNTpHw-4r-GiHQTO1AH7JpDiJf)

    반면 Deep Learning은 사람 얼굴에 관련된 유의미한 feature를 연결된 신경망을 거치면서 스스로 찾는다. CNN(Convolution Neural Network의 경우 초기 layer에서 convolution을 통해 이미지의 edge나 line같은 low-level-feature를 학습하고 그 다음 이미지의 high-level-feature 표현을 학습한다. 

    ![img](https://drive.google.com/uc?id=1sJiMn4IODIWX1U6xoVF8nLrLW1d7L6hx)

    조금 더 쉬운 예로 손글씨를 인식하는 Deep Learning 모델을 가정해보자. 이미지를 28*28로 쪼개어 784개의 Input layer를 만들고 각각 16개의 Hidden layer를 2개 배치한다. 숫자는 0~9까지로 총 10개의 output layer가 만들어진다.  input된 데이터는 각각 연결된 노드를 통해 w를 곱해 계산되며 최종적으로 0~8까지의 출력층에서는 활성화되지 않고, 9노드만 활성화되어 이 모델은 9라고 판단하게 된다. 각 layer들을 거칠 때 신경망은 아래 그림과 같이 스스로 feature를 탐색하여 학습한다. 

    ![img](https://drive.google.com/uc?id=1c-UhcQg76jxoMxEEpS32NUbAKXlsFZTM)

    3.2 결과에 대한 해석

    Machine Learning 알고리즘은 결과에 대한 해석이 명료하다. 특히 의사결정나무는 결과가 도출되기까지 각 노드의 엔트로피와 같은 명확한 규칙을 알 수 있다. 또한 Regression의 결과값인 $$R^2$$ 역시 내가 선택한 변수들의 설명력으로써 해석할 수 있다. 

    그러나 Deep Learning의 결과값은 해석할 수 없다. 수학적으로 어느 노드가 얼만큼 활성화되었는지는 알 수 있지만, 그 노드가 어떤 역할을 하는지, 하나의 레이어가 정확하게 무엇을 하는지 알지 못한다. 따라서 결과값에 이용된 근거(변수; feature)를 정확하게 알 수 없고 따라서 Deep Learning의 결과는 일반적으로 해석할 수 없다.

    현재 딥러닝이 산업에서 즉각적인 적용이 어려운 주요한 이유 중 하나다. 성능은 이미 사람과 유사하거나 혹은, 그 이상인 경우도 많다. 특히 Computer Vision 분야에서는 사실상 사람의 능력을 상회한다는 게 중론이며, 일란성 쌍둥이까지 구분해낸다고 한다. 그러나 정확하게 어떻게 이를 구별해내는지 알기 어렵기 때문에 Deep Learning의 실무 적용에 고민하게 되며, 특히 결과에 대한 '해석'이 중요한 산업일수록 기존 Machine Learning의 Regression 모델이 활용된다.

    3.3 통계학적 제약

    대부분의 Machine Learning 알고리즘 성능은 feature가 어느 정도로 정확히 식별되고 추출되는가에 달려 있다. 그렇기 때문에 연관성이 높은 변수를 식별해야하고 패턴을 보다 잘 보이게 하는 Feature engineering이 매우 중요하다. Regression 모델을 만들면서 feature에 대한 EDA, 정규화(normalization)/표준화 Scaling 등 feature에 대한 전처리가 중요하게 다뤄진다. 또한 정규분포, p-value, 다중공선성 등 통계학적 전제 조건들을 성립시켜야 제대로 된 성능을 낸다. 

    반면 Deep Learning은 '비교적' 통계적 제약으로 부터 자유롭다. Deep Learning 중 DNN(Deep Neural Network)이 일정 부분 전처리를 자체적으로 진행하기 때문이다. 물론 Input에 대한 정규화(normalization)가 필요할 때도 있지만, missing values도 전처리하지 않는 경우도 있다.  

    >Deep Learning(DL) vs Deep Neural Network(DNN)
    DL은 자연이 매우 구성적이라는데서 착안했다. 컴퓨터 비전을 예로 들면 
    > - 픽셀이 결합하여 가장자리를 형성한다. 
    > - 모서리가 결합되어 부품을 형성한다. 
    > - 부품이 결합되어 완전한 개체를 형성한다. <br> 
    
    >이러한 계층적 표현(hierarchical representation)에 대한 학습을 말한다. 그에 반해 DNN은 모델이 데이터를 적절하게 처리할 수 있게끔 전처리하는 역할을 한다.

    3.4 Machine Learning과 Deep Learning의 차이 예시

    ![img](https://drive.google.com/uc?id=1EPRjDZ-ORcSKPy55KRdNFW6d5UepQlK_)

    사진에 나와 있는 것은 왜 강아지이고 고양이인가? 그렇다면 어떤 기준을 가지고 둘을 '분류'할 수 있겠는가? 대부분 사람들에게 물어보면 아마도 '그냥'이라고 대답할 것이다. 뭐라 설명할 수 없는 경험적으로 알고 있는 것. 

    이 때문에 오랜 시간 Machine Learning에서는 개와 고양이를 제대로 분류하는 모델을 만들어 내지 못했다. 둘을 구분할 수 있는 feature들을 적절하게 집어내지 못했기 때문이다. 반면 Deep Learning은 이 둘을 구별해낼 수 있다. 어떻게 구별하는지 '정확하게' 정의할 수 없다. 우리가 '그냥'이라고 대답하는 것처럼.

4. AI의 바탕

    수학은 철학을 증명하기 위해서 발전해왔다고 한다. AI도 마찬가지. 통계 기반이든, 신호 기반이든, 패턴 기반이든, 신경계 기반이든, 증명하고자 하는 철학(사상)이 전제가 된다. 물론 이 철학에 따라 증명하고자 하는 방법도 달라지게 된다. 

    다만 Neural Network의 경우 우리가 정확하게 '왜 잘 돌아가는지' 알지 못한다. 증기기관이 상용화 되고 난 후에 열역학 법칙이 정리된 것처럼. 

0. 참고 사이트 

    [남세동의 딥러닝 이야기](https://www.youtube.com/watch?v=kMGEpIYPCiM) <br>
    [Deep Learning VS Machine Learning](https://brunch.co.kr/@itschloe1/8) <br>
    [Neural Network](https://brunch.co.kr/@gdhan/6) <br>
    [Deep Learning Feature 추출](https://warm-uk.tistory.com/53) <br>
    [Neural Network_video](https://www.youtube.com/watch?v=aircAruvnKk) <br>
    [DNN vs DL](https://www.quora.com/Is-Convolutional-Neural-Network-basically-data-preprocessing-via-kernel-plus-Neural-Networks-Isnt-Deep-Learning-just-neural-networks-with-some-pre-processing-for-automated-feature-selections)