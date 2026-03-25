---
layout: post
title:  "Linear Regression"
date:   2025-10-20T20:08:52+09:00
author: DINHO
categories:
  - Python
---

오늘은 Python으로 Linear Regression을 복습해볼까 합니다.인공지능을 공부하는 사람이라면 모를 수가 없는데요. 학부 시절 풀었던 과제를 바탕으로 포스팅하겠습니다. 
머신러닝을 공부하는 사람이라면 여기서부터 공부 시작이라고 할 수 있습니다!

이 내용은 광운대학교 전기공학과 인공지능응용 수업 과제입니다. 혹시라도 후배님들이 이 글을 보게 된다면 비밀로 하고 과제를 진행해주세요 

#  Linear Regression (선형 회귀) 정리

Linear Regression은 머신러닝에서 가장 기본이 되는 모델 중 하나로,  
입력 변수와 출력 변수 사이의 **선형 관계** 를 모델링하는 방법입니다.

---

## 1.  핵심 개념

Linear Regression은 다음과 같은 형태의 직선을 찾는 과정입니다.

\[
y = wx + b
\]

- \( w \): 기울기 (weight)  
- \( b \): 절편 (bias)

즉, **데이터를 가장 잘 설명하는 직선 하나를 찾는 문제**입니다.

---

## 2. 왜 사용하는가

Linear Regression의 목적은 크게 두 가지입니다.

- 새로운 입력에 대한 **값 예측**
- 데이터의 **경향성(trend)** 파악

### 예시
- 집 크기 → 집 가격
- 공부 시간 → 시험 점수

---

## 3. 학습 방식 (중요)

모델은 다음을 최소화하도록 학습됩니다.

\[
J(w,b) = \frac{1}{n} \sum_{i=1}^{n} (y_i - (wx_i + b))^2
\]

이를 **Mean Squared Error (MSE)** 라고 합니다.

### 학습 과정 요약

1. 예측값 계산  
   \[
   \hat{y} = wx + b
   \]

2. 실제값과의 차이 계산  
   \[
   y - \hat{y}
   \]

3. 오차 제곱 후 평균

4. 이 값을 최소화하도록 \( w, b \) 업데이트

---

## 4. 다변수 Linear Regression

입력이 여러 개일 경우 다음과 같이 확장됩니다.

\[
y = w_1 x_1 + w_2 x_2 + \cdots + w_n x_n + b
\]

또는 벡터 형태로:

\[
y = \mathbf{w}^T \mathbf{x} + b
\]

---

## 5. 직관적인 이해

- 데이터 점들이 존재함
- 그 점들을 가장 잘 통과하는 **직선(또는 평면)**을 찾는 문제

👉 즉, "오차가 가장 작은 직선 찾기"

---

## 6. 한계점

Linear Regression은 간단하지만 다음과 같은 한계가 있습니다.

- 비선형 관계를 표현하기 어려움
- 이상치(outlier)에 민감
- 복잡한 데이터 패턴 학습 불가

---

## 7. 정리

- Linear Regression = **직선으로 데이터 설명**
- 목표 = **오차(MSE) 최소화**
- 특징 = 단순하지만 매우 중요한 기본 모델

---


### 실습

* 행렬 방정식 풀기

  - 다음 행렬 방정식을 'Linear Regression'을 이용해 풀어보자

    + 적당한 learning rate를 찾아 1000 epoch 정도 계산해본다

    + 'Pseudo Inverse'를 이용한 풀이와 비교해본다

  - Hint: y = wx 꼴로 변환해본다

    + Ax=B에서는 x가 미지수이지만, y=wx에서는 w가 미지수임에 주의!

    + linear model에서 b를 없애기 위해서 nn.Linear() 사용법을 검색해보자
    
$$Ax=B$$

$$A = \begin{bmatrix}0 & 1 \\ 1 & 1 \\ 2 & 1 \\ 3 & 1 \end{bmatrix} $$

$$B = \begin{bmatrix}-1 \\ 0.2 \\ 0.9 \\ 2.1 \end{bmatrix} $$



```Python
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as func
import torch.optim as opt

# 행렬 A와 벡터 B 정의
A = torch.FloatTensor([[0, 1], [1, 1], [2, 1], [3, 1]])
B = torch.FloatTensor([-1, 0.2, 0.9, 2.1]).reshape(-1, 1)

# Model 초기화 (입력 dim, 출력 dim)
model = nn.Linear(2, 1)

# 손실 함수 및 옵티마이저 정의
cost = nn.MSELoss()
optimizer = opt.SGD(model.parameters(), lr=0.1)

# 학습 과정
epochs = 1000
for epoch in range(epochs):
    # 예측
    A_pred = model(A)

    # 손실 계산
    loss = cost(A_pred, B)

    # 역전파 및 최적화
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    # 10 에포크마다 손실 출력
    if (epoch+1) % 100 == 0:
        print(f'Epoch [{epoch+1}/{epochs}], Loss: {loss.item():.4f}')

# 최종 가중치 출력
with torch.no_grad():
    print(f'Learned weights: {model.weight}')

#Pseudo Inverse

# A와 B 정의
A_np = np.array([[0, 1], [1, 1], [2, 1], [3, 1]])
B_np = np.array([-1, 0.2, 0.9, 2.1])

# Pseudo Inverse 계산
w_pseudo_inverse = np.linalg.pinv(A_np).dot(B_np)
print(f'Weights using Pseudo Inverse: {w_pseudo_inverse}')

```
linear regression을 통해 얻은 값은 tensor([[ 1.0000, -0.9550]], requires_grad=True) 로 얻었는데요, 실제 계산하면 [ 1.   -0.95]가 나옵니다. 방대한 데이터에서 정확한 값을 찾기 힘들거나, 없는 데이터가 들어갔을 때 예상값의 근사치(외삽, Extrapolation)을 할 때 사용되는 방법입니다.

간단하지만 중요한 기초 내용입니다!