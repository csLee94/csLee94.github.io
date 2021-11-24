---
layout: post
title:  "케라스 창시자에게 배우는 딥러닝 Study(1) - 용어 정리"
subtitle:   케라스 창시자에게 배우는 딥러닝 Study
categories: datascience
tags: AI DeepLearning keras review 
comments: true
---


# 목차
1. [Related Post](#related-post)


<br>

> 이 글은 **"케라스 창시자에게 배우는 딥러닝"** (프랑소와 숄레, 박해선 옮김; 길벗출판사)를 읽고 공부하며 남기는 Post입니다.
![img](https://drive.google.com/uc?id=1knT2Fs7LfbZ7iLrAzSK2wvNpcDV4zb5D)

---

<br>

# Related Post
- 1부: 딥러닝의 기초
- 2부: 실전 딥러닝

<br>

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
    |시계열 데이터 혹은 시퀀스 데이터| (samples, timesteps, features)|3D 텐서|
    |이미지 데이터| (samples, height, wight, channels)|4D 텐서|
    |동영상 데이터| (samples, frames, height, width, channels)|5D 텐서|

### 신경망의 엔진
```python
output = relu(dot(W, input) + b)
# dot는 행렬 곱을 계산하는 numpy function
```

- **가중치(weight) 혹은 커널(kernel)**: 위 식 상 W에 해당하는 값 
- **훈련되는 파라미터 혹은 편향(bias)**: 위 식 상 b에 해당하는 값 
- 커널(kernel): 커널은 여러 가지 의미로 사용되는데, 대표적으로 서포트 백터 머신의 커널함수와 합성곱 신경망의 필터가 있습니다. 

<br>

# 2단계
- **손실함수**: 훈련 데이터에서 신경망의 성능을 측정하는 방법으로 네트워크가 옳은 방향으로 학습될 수 있도록 도와줍니다.

<br>

# 3단계
- **옵티마이저**: 입력된 데이터와 손실 함수를 기반으로 네트워크를 업데이트하는 메커니즘입니다.