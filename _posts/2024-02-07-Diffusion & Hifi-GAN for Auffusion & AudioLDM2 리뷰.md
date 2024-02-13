---
layout: post
title:  "Diffusion & Hifi-GAN for Auffusion & AudioLDM2 리뷰"
date:   2024-02-07T14:08:52+09:00
author: DINHO
categories:
  - 논문-리뷰
  - 인공지능-공부
cover:  "/assets/post/auffusion_image.png"
---

최근 오디오 생성형 AI 모델에 대한 연구가 활발히 진행중입니다. 그 중에서도 가장 최신 모델인 [Auffusion](https://auffusion.github.io/)과 [AudioLDM2](https://audioldm.github.io/audioldm2/)에 대한 논문 리뷰를 해보겠습니다. 각 모델에 대한 자세한 정보는 링크를 타고 들어가시면 볼 수 있습니다. 

두 모델 모두 Diffusion, HiFi-GAN, Transformer, VAE 등 공통적으로 알아야할 내용들이 있습니다. 그래서 이번 게시글에서는 먼저 공통적으로 알아야할 내용들에 대해 다루어보겠습니다. 후속편에서 본격적으로 두 모델의 구조와 차이에 대해서 알아보겠습니다.

# Diffusion Overview

디퓨젼(Diffusion)이란 데이터를 만들어내는 Deep Generative Model 중 하나입니다.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*xc2Y6jwIUhfEGxJLytU1RA.png">
이미지 출처: https://miro.medium.com/v2/resize:fit:720/format:webp/1*xc2Y6jwIUhfEGxJLytU1RA.png

그림에서와 같이 디퓨젼에는 Forward process와 Reverse process가 있습니다. Forward process에서는 데이터에 _Fixed(고정된)_ Gaussian Noise가 더해집니다. Reverse process에서는 데이터를 _Learned(학습된)_ Gaussian Noise로 뺍니다. 즉, 디퓨젼의 목적은 Forward -> Reverse 단계를 거친 '결과 이미지'를 '입력 이미지'의 확률 분포와 유사하게 만드는 것입니다. 이미지 픽셀값에 첨가된 Noise 값을 계산할 수 있다면 Noise 이미지 $$x_T$$ 로부터 진짜 이미지 $$x0$$로 되돌리는 것이 가능하겠죠? 이 말은 곧 입력으로 랜덤하게 생성한 Noise 이미지 $$x_T$$가 들어가면 Reverse Diffusion process를 거쳐 완전한 이미지 $$x_0$$가 만들어진다는 의미입니다.

그렇다면 과정들을 자세하게 알아보도록 하겠습니다.

### Forward process

Forward process에서는 노이즈의 분포를 알아내야 합니다. 그 이유는 Reverse process에서 학습할 때 Forward process의 정보를 활용하기 때문이죠. Forward process는 데이터 $$x_0$$ 에서 잠재 변수(latent variable) $$x_T$$ 까지의 고정 마르코프 체인(Fixed Markov Chain)으로 정의됩니다. 이를 수식으로 표현하면 다음과 같습니다.

$$q(x_1,x_2, ... , x_T \vert x_0) = \prod_{t=1}^{T} q(x_t \vert x_{t_1})$$

각 부분을 살펴보면

$$q(x_1,x_2, ... , x_T \vert x_0)$$ : 이 수식은 원본 데이터 $$x_0$$ 가 주어졌을 때, 노이즈를 추가하여 $$x_1$$ 부터 $$x_T$$ 까지 연속적인 상태에 도달할 조건부 확률을 나타냅니다. 

$$\prod_{t=1}^{T}$$ : 이 수식은 다들 아실 텐데요!! t=1 부터 T까지의 모든 항목을 곱하라는 의미입니다. 시그마의 곱셈버전이라고 생각하시면 편합니다.

$$q(x_t \vert x_{t_1})$$ : 이 수식은 각 단계 t에서의 상태 $$x_t$$ 가 이전 상태 $$x_{t-1}$$ 에 기반할 때의 확률 분포를 나타냅니다.

요약하자면 이 수식은 노이즈가 점차적으로 추가되는 과정을 수학적으로 모델링한 것입니다. 각 단계에서의 변화는 조건부 확률 분포를 사용하여 표현되며, 전체 Forward process는 이러한 변화들의 연속적인 곱으로 나타납니다.

$$q(x_t \vert x_{t_1})$$ 이 수식은 다음과 같이 알아볼 수 있습니다.

$$q(x_t \vert x_{t-1}) := \mathcal{N}(x_t; \sqrt{1 - \beta_t}x_{t-1}, \beta_t I)$$ 

각 부분을 살펴보면

$$\mathcal{N}(x_t; \sqrt{1 - \beta_t}x_{t-1}, \beta_t I)$$ : 여기서는 평균이 $$\sqrt{1 - \beta_t}x_{t-1}$$ , 공분산이 $$\beta_t I$$ 인 다변수 정규 분포를 의미합니다. 여기서 $$I$$는 단위 행렬(identity matrix)을 의미하며, 이는 모든 변수들이 서로 독립적이고 동일한 분산을 가짐을 의미합니다.

$$\sqrt{1 - \beta_t}x_{t-1}$$ : 여기서 $$\sqrt{1 - \beta_t}$$ 는 이전 상태 $$x_{t−1}$$ 에 노이즈를 추가하기 전에 적용되는 스케일링 팩터(scaling factor)입니다. 이는 Diffusion 과정에서 각 단계에서 이전 상태 $$x_{t−1}$$ 에 노이즈를 추가ㅏ하기 전에 적용되는 배율을 의미합니다. 스케일링 펙터의 역할에는 정보의 보존, 노이즈 추가의 조절, Reverse process 준비 등이 있습니다. 자세히 설명하자면:

- 정보의 보존: 스케일링 팩터는 원본 데이터 $$x_{t−1}$$ 에서 일부 정보를 유지하면서 새로운 노이즈를 추가합니다. 이때, $$\beta_t$$ 가 0에 가까울수록 더 많은 원본 정보를 유지하며, 1에 가까울수록 더 많은 정보가 노이즈로 인해 손실됩니다.

- 노이즈 추가의 조절: Diffusion 모델은 데이터에 점차적으로 노이즈를 추가하여 데이터 분포로부터 멀어지게 만드는 과정입니다. 스케일링 팩터는 각 단계에서 노이즈를 추가할 때 원본 데이터의 양을 조절하여 이 과정을 세밀하게 제어합니다.

- Reverse process 준비: 스케일링 팩터는 forward process에서 데이터에 노이즈를 추가하는 방식을 정의함으로써, 나중에 reverse process를 수행할 때 데이터를 복원하는 방법을 모델이 학습할 수 있도록 준비합니다.

전체 절차는 미리 결정된 노이즈 스케줄 "β1,...,βT"에 따라 초기 잠재 데이터 $$x0$$ 를 노이즈 잠재 변수 $$x_T$$ 로 변환합니다. 여기서 βt는 작은 양의 상수입니다. $$q(x_t | x_{t−1})$$ 은 $$x_{t−1}$$ 의 분포에 작은 가우스 노이즈가 추가되는 함수를 나타냅니다. 이 수식은 모델이 이전 상태 $$x_{t−1}$$ 로부터 새로운 상태 $$x_T$$ 를 생성할 때, 원본 데이터의 일부 정보를 유지하면서 어느 정도의 노이즈를 추가할지를 정의합니다. Diffusion 과정에서는 이처럼 점차적으로 노이즈를 증가시키며, 원본 데이터로부터 멀어지는 과정을 시뮬레이션합니다.
