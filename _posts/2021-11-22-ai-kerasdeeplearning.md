---
layout: post
title:  "케라스 창시자에게 배우는 딥러닝 Study(1) - 용어 정리"
subtitle:   케라스 창시자에게 배우는 딥러닝 Study
categories: datascience
tags: AI DeepLearning keras review 
comments: true
---



> 이 글은 **"케라스 창시자에게 배우는 딥러닝"** (프랑소와 숄레, 박해선 옮김; 길벗출판사)를 읽고 공부하며 남기는 Post입니다.
![img](https://drive.google.com/uc?id=1knT2Fs7LfbZ7iLrAzSK2wvNpcDV4zb5D)

---

<br>


> ![img](https://drive.google.com/uc?id=11nNrXW-B-BXBGc5hZtaT0K2DEo-3r14A)
> 위 그림에서 각 단계를 기준으로 용어에 대한 정의를 정리

<br>

# 1단계

### 데이터의 표현
- **텐서(Tensor)**: 다차원 배열을 나타내는 단위
    > 주) *'텐서플로를 비롯하여 딥러닝 라이브러리들은 종종 다차원 배열을 텐서라고 부릅니다. 이 책에서는 넘파이 배열도 텐서라고 하지만, 사실 파이썬 커뮤니티에서 넘파이 배열을 텐서라고 부르지는 않습니다.*
- **스칼라(0D 텐서)**: 하나의 숫자만을 담고 있는 텐서 (ndim==0)
- **벡터(1D 텐서)**: 숫자의 배열을 벡터, 또는 1D 텐서라고 부릅니다. (ndim==1)
    > 5D 텐서와 5D 벡터를 혼동하지 말자! **5D 텐서**는 5개의 축을 가지는 텐서, **5D 벡터**는 5개의 원소를 가지는 벡터
- **행렬(2D 텐서)**: 백터의 배열로 행렬(Matrix)또는 2D 텐서 (ndim==2)
- **3D 텐서**: 행렬의 배열로 정육면체로 이해할 수 있는 3개의 축을 가진 텐서.
    > 3D 텐서의 배열은 4D 텐서고, 동영상 딥러닝의 경우 5D 텐서까지 다루기도 한다. 

- **텐서의 실제 사례**: 데이터 배열의 형태 앞에 "sample" 수가 붙는다.

    |사례|구조|텐서 차원|
    |---|---|---|
    |벡터 데이터|(samples, features)|2D 텐서|
    |시계열 데이터 혹은 시퀀스 데이터|(samples, timesteps, features)|3D 텐서|
    |이미지 데이터|(samples, height, wight, channels)|4D 텐서|
    |동영상 데이터|(samples, frames, height, width, channels)|5D 텐서|

### 신경망의 엔진
```python
output = relu(dot(W, input) + b)
# dot는 행렬 곱을 계산하는 numpy function
```

- **가중치(weight) 혹은 커널(kernel)**: 위 식 상 W에 해당하는 값 
- **훈련되는 파라미터 혹은 편향(bias)**: 위 식 상 b에 해당하는 값 
- 커널(kernel): 커널은 여러 가지 의미로 사용되는데, 대표적으로 서포트 백터 머신의 커널함수와 합성곱 신경망의 필터가 있습니다. 

### 훈련
- **에포크(Epoch)**: 전체 훈련 데이터에 수행되는 각 반복 횟수. 전체 Dataset에 대한 전체 과정(Forward pass + backward pass)가 완료되면 한 번의 Epoch가 완료됐다고 볼 수 있다.
- **배치 사이즈(batch-size)**: 대부분의 경우에는 메모리의 한계와 속도 저하 때문에 한 번의 Epoch에 모든 데이터를 한꺼번에 집어 넣을 수 없습니다. 그래서 데이터를 나누어 학습하는데, 나눠진 데이터 샘플 표본을 batch-size라고 합니다.
    > 예를 들어, Samples 수가 **6만개인 Training Set**을 **batch_size=128, epoch=5**로 학습을 시작하면 469번의 배치가 만들어져 학습하게 됩니다. 마지막 배치의 샘플 수는 96개 이며, 하나의 배치를 학습하는 횟수를 **Iteration**이라고 합니다. <br> 각 iteration 마다 네트워크가 배치에서 손실에 대한 가중치의 그래디언트를 계산하고, 업데이트합니다. 이 경우, 5번의 epoch 동안 네트워크는 2,345(469*5)번의 그래디언트 업데이트를 수행할 것입니다.

<br>

# 2단계
- **손실함수**: 훈련 데이터에서 신경망의 성능을 측정하는 방법으로 네트워크가 옳은 방향으로 학습될 수 있도록 도와줍니다.
- **그래디언트(Gradient)**: 연산 중 비효율화를 위해서 신경망에 사용되는 모든 연산이 **미분 가능**하다는 장점을 사용해 네트워크 가중치에 대한 손실의 **그래디언트**를 계산합니다. 손실함수에 대해 미분, 즉 **순간변화율**을 통해 손실함수 f(x)의 값의 변화를 추적합니다.
- **확률적 경사 하강법(stochastic gradient descent; SGD)**: 여기서 '확률적(stochastic)'은 각 배치 데이터가 무작위로 선택된다는 표현입니다. 
- **미니 배치 SGD와 배치 SGD**: SGD 알고리즘에서 "step" 값을 적절히 고르는 것이 중요합니다. 이 값이 너무 작으면 손실함수곡선을 따라 내려가는데 너무 많은 반복이 필요하고, 지역의 최솟값에 갇힐 수 있습니다. 반면 너무 크면 완전히 임의의 위치로 이동시킬 수 있습니다. <br> **미니 배치 SGD**는 무작위로 선택된 배치를 통해 업데이트하며, 가용한 모든 데이터를 사용해 반복하는 걸 **배치 SGD**라고 합니다. 이 경우 더 정확하게 업데이트되지만, 더 많은 비용이 듭니다.


<br>

# 3단계
- **옵티마이저**: 입력된 데이터와 손실 함수를 기반으로 네트워크를 업데이트하는 메커니즘입니다. <br> 현재 그래디언트 값만 보지 않고 이전에 업데이트된 가중치를 여러 가지 다른 방식으로 고려합니다. 대표적으로 모멘텀을 사용하는 **Adagrad**, **RMSProp** 등이 있습니다.
- **모멘텀(Momentum)**: 기존 SGD에 있는 2개의 문제점인 수렴 속도와 지역 최솟값을 해결합니다. 모멘텀은 물리학에서 영감을 받아 그라디언트 알고리즘에 '운동량'을 추가합니다. '운동량'이 충분하다면 지역 최솟값에 같히지 않고 전역 최솟값에 도달할 수 있을 것입니다. 현재의 **기울기**(현재의 가속도) 뿐만 아니라 과거의 가속도를 함께 고려합니다. 
    > ![img](https://drive.google.com/uc?id=1v8r8lXqTJ921wKwYh8wO4rdO20pDXJki)
    > 모멘텀은 현재 기울기 값(현재 가속도)뿐만 아니라 과거의 가속도를 함께 고려해 각 단계에서 공을 움직입니다. 실제로는 현재 그래디언트 값뿐만 아니라, 이전에 업데이트한 파라미터에 기초해 파라미터 w를 업데이트 합니다.
    
- **역전파(Backpropagation)**: **후진 모드 자동 미분(reverse-mode automatic differentiation)**이라고도 부릅니다. 손실 값에 각 파라미터가 기여한 정도를 계산하기 위해 **연쇄 법칙**을 적용해 최상위 층에서 하위층까지 거꾸로 진행됩니다.
    > 요즘에는 텐서플로처럼 **기호 미분**이 가능한 프레임워크를 사용해 신경망을 구현합니다. 역방향 패스는 그래디언트 함수를 호출하는 것으로 단순화될 수 있으므로, 정확한 역전파 공식을 유도하거나 직접 구현할 필요가 없어졌습니다.



# Summary

아래 샘플 코드를 통해 간단하게 해석하겠습니다. 
```python
from keras.datasets import mnist
from keras import models
from keras import layers

# 학습 데이터 생성
(train_images, train_labels), (test_images, test_labels) = mnist.load_data()

train_images = train_images.reshape((60000, 28*28))
train_images = train_images.astype('float32')/255

test_images = test_images.reshape((10000, 28*28))
test_iamges = test_images.astype('float32')/255

# 1. 신경망 생성
network = models.Sequential()
network.add(layers.Dense(512, activation='relu', input_shape=(28*28,)))
network.add(layers.Dense(10, activation='softmax'))

# 2. 네트워크 컴파일
network.compile(optimizer = 'rmsprop', loss='categorical_crossentropy', metrics=['accuracy'])

# 3. 훈련반복
network.fit(train_images, train_labels, epochs=5, batch_size=128)
```

### 신경망 생성
2개의 Dense 층이 연결되어 있고 각 층은 가중치 텐서를 포함해 입력 데이터에 대한 몇 개의 간단한 텐서 연산을 적용합니다. 층의 속성인 가주이 텐서는 네트워크가 정보를 저장하는 곳입니다.

### 네트워크 컴파일
'categorical_crossentropy'는 `손실 함수`이며, 가중치 텐서를 학습하기 위한 피드백 신호로 사용되고 훈련하는 동안 최소화 됩니다. 경사하강법을 적용하는 구체적인 방식은 `rmsprop 옵티마이저`에 의해 결정 됩니다. 이 때 훈련 과정에서 모니터링하는 지표는 `accuracy`(정확도; 정확히 분류된 이미지 비율)입니다.

### 훈련반복
총 60,000개의 샘플을 128개씩 나누어 469회 학습하며, 이를 총 5회 반복합니다. 이 때 네트워크는 총 2,345(469*5)번의 그래디언트 업데이트를 수행할 것입니다.








<br><br>

# Reference
> - ["쵸코쿠기의 연습장" 블로그](https://jjeongil.tistory.com/999): 모멘텀에 대한 추가 설명 및 그림
> - ["개발자의 개발 정보와 리뷰" 블로그](https://m.blog.naver.com/qbxlvnf11/221449297033): Epoch, batch size, iteration에 대한 추가 설명