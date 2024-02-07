---
layout: post
title:  "Auffusion & AudioLDM2 리뷰1"
date:   2024-02-07T14:08:52+09:00
author: DINHO
categories:
  - 논문-리뷰
cover:  "/assets/post/auffusion_image.png"
---

최근 오디오 생성형 AI 모델에 대한 연구가 활발히 진행중입니다. 그 중에서도 가장 최신 모델인 [Auffusion](https://auffusion.github.io/)과 [AudioLDM2](https://audioldm.github.io/audioldm2/)에 대한 논문 리뷰를 해보겠습니다. 각 모델에 대한 자세한 정보는 링크를 타고 들어가시면 볼 수 있습니다. 

두 모델 모두 Diffusion, HiFi-GAN, Transformer, VAE 등 공통적으로 알아야할 내용들이 있습니다. 그래서 이번 게시글에서는 먼저 공통적으로 알아야할 내용들에 대해 다루어보겠습니다. 이후에 2편에서 본격적으로 두 모델의 구조와 차이에 대해서 알아보겠습니다.

# 