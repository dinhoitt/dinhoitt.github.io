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

두 모델 모두 Diffusion, HiFi-GAN, Transformer, VAE 등 공통적으로 알아야할 내용들이 있습니다. 그래서 이번 게시글에서는 먼저 공통적으로 알아야할 내용들에 대해 다루어보겠습니다. 이후에 2편에서 본격적으로 두 모델의 구조와 차이에 대해서 알아보겠습니다.

# Diffusion Overview

디퓨젼(Diffusion)이란 데이터를 만들어내는 Deep Generative Model 중 하나입니다.

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*xc2Y6jwIUhfEGxJLytU1RA.png">
이미지 출처: https://miro.medium.com/v2/resize:fit:720/format:webp/1*xc2Y6jwIUhfEGxJLytU1RA.png

그림에서와 같이 디퓨젼에는 Forward process와 Reverse process가 있습니다. Forward process에서는 데이터에 _Fixed(고정된)_ Gaussian Noise가 더해집니다. Reverse process에서는 데이터를 _Learned(학습된)_ Gaussian Noise로 뺍니다. 즉, 디퓨젼의 목적은 Forward -> Reverse 단계를 거친 '결과 이미지'를 '입력 이미지'의 확률 분포와 유사하게 만드는 것입니다. 이미지 픽셀값에 첨가된 Noise 값을 계산할 수 있다면 Noise 이미지 $$x_T$$ 로부터 진짜 이미지 $$x0$$로 되돌리는 것이 가능하겠죠? 이 말은 곧 입력으로 랜덤하게 생성한 Noise 이미지 $$x_T$$가 들어가면 Reverse Diffusion process를 거쳐 완전한 이미지 $$x_0$$가 만들어진다는 의미입니다.

그렇다면 과정들을 자세하게 알아보도록 하겠습니다.

### Forward process

Forward process에서는 노이즈의 분포를 알아내야 합니다. 그 이유는 Reverse process에서 학습할 때 Forward process의 정보를 활용하기 때문이죠. Forward process는 데이터 $$x_0$$ 에서 잠재 변수(latent variable) $$x_T$$ 까지의 고정 마르코프 체인(Fixed Markov Chain)으로 정의됩니다.