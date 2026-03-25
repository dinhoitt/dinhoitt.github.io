---
layout: post
title:  "Numpy and Matplotlib with Python"
date:   2025-10-20T20:08:52+09:00
author: DINHO
categories:
  - Python
---

오늘은 Python Numpy와 Matplotlib 기초를 복습해볼까 합니다. 두 라이브러리는 인공지능을 공부하는 사람이라면 모를 수가 없는데요. 학부 시절 풀었던 과제를 바탕으로 포스팅하겠습니다.

예전에 [SSIM](https://dinhoitt.github.io/python/%EC%8B%A0%ED%98%B8-%EC%B2%98%EB%A6%AC-%EC%9D%B4%EB%A1%A0/2024/07/03/SSIM.html) 파이썬 코드를 공유하면서도 아마 보셨던 라이브러리일 겁니다.

이 내용은 광운대학교 전기공학과 인공지능응용 수업 과제입니다. 혹시라도 후배님들이 이 글을 보게 된다면 비밀로 하고 과제를 진행해주세요 ㅎㅎ

# Numpy

### 실습

* 행렬 방정식 풀기

  - 다음 행렬 방정식을 'Pseudo inverse matrix'를 이용해 풀어보자

  - $A^{T}A$의 역행렬이 존재한다고 가정

$$Ax=B$$

$$A = \begin{bmatrix}0 & 1 \\ 1 & 1 \\ 2 & 1 \\ 3 & 1 \end{bmatrix} $$

$$B = \begin{bmatrix}-1 \\ 0.2 \\ 0.9 \\ 2.1 \end{bmatrix} $$

```Python
import numpy as np # 보통 이렇게 많이 줄입니다.

# 행렬 정의
matrixA = np.array([[0,1],[1,1],[2,1],[3,1]])
matrixB = np.array([[-1],[0.2],[0.9],[2.1]])

# (A^T A)^(-1) A^T B 계산
pseudo_inverse_solution = np.dot(np.linalg.inv(np.dot(matrixA.T, matrixA)), np.dot(matrixA.T, matrixB))

print(pseudo_inverse_solution)

```


# Matplotlib

* 2x2 그래프 그리기

  - $y=x$

  - $y=x^2$

  - $y=sin(x)$

  - $y=cos(x)$

```python
import matplotlib.pyplot as plt

# x range for the graphs
x = np.linspace(-10, 10, 400)

# Create the 2x2 subplot
fig, axs = plt.subplots(2, 2, figsize=(10, 8))

# y = x
axs[0, 0].plot(x, x)
axs[0, 0].set_title('y = x')

# y = x^2
axs[0, 1].plot(x, x**2)
axs[0, 1].set_title('y = x^2')

# y = sin(x)
axs[1, 0].plot(x, np.sin(x))
axs[1, 0].set_title('y = sin(x)')

# y = cos(x)
axs[1, 1].plot(x, np.cos(x))
axs[1, 1].set_title('y = cos(x)')

# Display the plot
plt.tight_layout()
plt.show()


```

간단한 코드라 쉽게 이해할 수 있을 겁니다.