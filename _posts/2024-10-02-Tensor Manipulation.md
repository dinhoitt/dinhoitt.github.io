---
layout: post
title:  "Transformer란?"
date:   2024-10-02T20:08:52+09:00
author: DINHO
categories:
  - Python
cover:  "/assets/post/transformer.png"
---

오랜만에 포스팅입니다!! 개학하고 아주 정신이 없네요😅 

오늘은 Pytorch를 이용하여 간단한 행렬 방정식푸는 python 코드를 간단하게 소개해볼까 합니다. 

이 내용은 광운대학교 전기공학과 인공지능응용 수업 과제입니다. 혹시라도 후배님들이 이 글을 보게 된다면 비밀로 하고 과제를 진행해주세요 ㅎㅎ

## 행렬 방정식 풀기 (단, Pytorch만을 사용한다) ##

  - 다음 행렬 방정식을 'Pseudo inverse matrix'를 이용해 풀어보자

  - $A^{T}A$의 역행렬이 존재한다고 가정

$$Ax=B$$
$$A = \begin{bmatrix}0 & 1 \\ 1 & 1 \\ 2 & 1 \\ 3 & 1 \end{bmatrix} $$
$$B = \begin{bmatrix}-1 \\ 0.2 \\ 0.9 \\ 2.1 \end{bmatrix} $$

```Python
import torch

# 행렬 A와 벡터 B 정의
A = torch.FloatTensor([[0, 1], [1, 1], [2, 1], [3, 1]])
B = torch.FloatTensor([-1, 0.2, 0.9, 2.1]).reshape(-1, 1)

# Pseudo 역행렬 계산: (A^T A)^{-1} A^T
A_pseudo_inv = torch.inverse(A.T @ A) @ A.T

# 방정식 Ax = B에서 x 계산
x = A_pseudo_inv @ B

print(x)

```

간단한 코드라 쉽게 이해할 수 있을 겁니다.