---
layout: post
title:  "Multi-Scale Sub-band CQT Discriminator리뷰"
date:   2025-07-01T13:22:52+09:00
author: DINHO
categories:
  - 인공지능-분야-공부
  - 논문-리뷰
cover:  "/assets/post/msct.png"
---

너무 오랜만에 포스팅입니다. 그 동안 논문 작성, 실험, 새로운 연구 주제 선정 등 많은 일들을 하다보니 시간이 이렇게 흘렀습니다. 

그 과정에서 제 모델을 [깃허브](https://github.com/dinhoitt/BemaGANv2) 에 공개했고, Huggingface에서 연락도 오고, 다른 연구자가 문의도 하고 정말 제가 연구자가 된 것 같은 기분이 들었습니다.😊

그 과정에서 다른 연구자가 제 깃허브를 보고 제안한 방법이 있는데, 그 관련 논문 리뷰를 해보겠습니다.

오늘 리뷰해 볼 논문은 Yicheng Gu et al. "Multi-Scale Sub-Band Constant-Q Transform Discriminator for High-Fidelity Vocoder"(2023) 입니다. 

# Introduction

이 논문은 중간 표현(대표적으로 멜스펙트로그램)으로부터 파형을 복원하는 GAN 기반 vocoder의 성능 향상에 중점을 둡니다.

기존의 GAN기반 vocoder의 연구는 생성기나 판별기에 중점을 두는데, 이 논문은 판별기에 중점을 둡니다.

GAN 기반의 vocoder에서 대부분의 연구는 STFT 스펙트로그램을 이용하여 학습합니다.

서로 다른 주파수 대역에 다른 attention을 요구하는 노래, 목소리 등의 신호를 처리할 때는 이것이 불충분할 수 있습니다.

연구자들은 여기에 착안하여 Constant-Q Transform(CQT)를 제안합니다. CQT가 STFT보다 다른 주파수 대역에 대해 더 유연한 해상도를 갖기 때문입니다.

# Multi-Scale Sub-band CQT Discriminator

이 논문에서는 어떤 GAN 기반 보코더에서든 통합될 수 있는 MS-SB-CQT 판별기 구조를 제안합니다.

서로 다른 스케일의 CQT 스펙트로그램에 작동하는 동일한 구조의 서브 판별기들로 구성 됩니다.

각 서브 판별기는 먼저 CQT의 실수부와 허수부를 제안된 Sub-Band Processing(SBP) 모듈에 개별적으로 보내서 잠재 표현을 얻습니다.

두 표현은 결합되어 컨볼루션 레이어로 전송되고, 출력된 아웃풋은 loss 계산에 사용됩니다.

<div align="center">
	<img src="/assets/post/msct.png" alt="Introduction of Melfusion">
	<p><em>[그림 1] 모델 구조</em></p>
</div>

이제 그림의 내용 하나 하나 이야기해보겠습니다.

# The Strengths of Constant-Q Transform

CQT가 왜 좋은지 연구자들의 주장을 이야기해보겠습니다.

가장 핵심은 각 주파수 bin의 중심 주파수 대역 폭의 비율(Q-factor)를 일정하게 유지한다는 것입니다. 이를 통해 주파수가 높아질수록 해상도를 줄이고, 낮아질수록 해상도를 높일 수 있습니다.(인간 귀의 특성 반영 가능)

$$Q = \frac{f_k}{\Delta f_k}$$

또한 주파수를 옥타브 단위로 나누어서 정보의 압축을 효율적으로 합니다.

<div align="center">
	<img src="/assets/post/msct.png" alt="Introduction of Melfusion">
	<p><em>[그림 2] 스펙트로그램과 CQT 스펙트럼 차이</em></p>
</div>

# Multi-Scale Sub-Discriminators

더 다양한 시간-주파수 해상도 하에서 정보를 포착하기 위해 MSD 아이디어를 활용합니다. 

서로 다른 해상도 trade-off를 가진 CQT에 하위 판별기들을 채택하고, 옥타브 당 bin 수에 해상도가 결정되므로 B가 24, 36, 48인 세 개의 하위 판별기 적용합니다.

$$Q_k \overset{\mathrm{ref.}}{=} \frac{f_k}{\Delta f_k}
= \left( 2^{\frac{1}{B}} - 1 \right)^{-1}$$

$$f_k = f_1 \cdot 2^{\frac{k-1}{B}}$$



# Sub-Band Processing Module

앞에서 언급한 SBP에서 이야기해보도록 하겠습니다.

이 모듈은 유연한 시간-주파수 해상도를 가져오지만, 고정되지 않은 윈도우 길이는 문제를 야기합니다.

다른 주파수 bin에 있는 커널들은 시간적으로 동기화되지 않습니다.

이 문제를 완화하기 위해 옥타브 내에서 시간적으로 동기화된 일련의 커널을 설계합니다.

하지만 시간적으로만 동기화할 뿐 옥타브 간 문제는 해결하지 못하였습니다. 실제로 연구진들은 실험 중 오히려 이러한 편향을 가진 CQT 스펙트로그램을 사용하는 것이 보코더 품질을 해치는 경우를 발견하기도 했습니다.

CQT 스펙트로그램의 실수부 또는 허수부는 옥타브에 따라서 Sub-Band로 분할됩니다. 그 후 각 밴드는 해당 컨볼루션 레이어로 보내져 표현을 얻습니다. 마지막으로 모든 밴드 표현들을 결합하여 잠재 표현을 얻습니다. (그림 1 참고)

# Experiments

이 논문에서는 아래와 같이 네 실험을 진행했습니다. 그 중 세 가지에 대해서만 리뷰하도록 하겠습니다. 실험 데이터셋은 음성과 노래 음성 모두 포함하고 있으며,  데이터 세트 구성, 코드 구현 세부사항 등은 논문에 자세하게 언급되어 있습니다.

EQ1: 제안된 MS-SB-CQT Discriminator는 얼마나 효과적인가?

EQ2: MS-SB-CQTD와 MS-STFTD를 함께 사용하는 것이 보코더를 더 향상시키는가?

EQ3: MS-SB-CQTD는 다른 GAN 기반 보코더에서도 일반화된 성능을 보이는가?

EQ4: 제안된 SBP 모듈을 채택하는 것이 필수적인가?(이 부분은 따로 언급하지 않겠습니다.)

1. EQ1 & EQ2

	- HiFi-GAN (+C)와 HiFi-GAN (+S) 모두 HiFi-GAN보다 더 나은 성능을 보여줌 

	- HiFi-GAN (+C)는 HiFi-GAN (+S)보다 더 나은 성능을 보이며 MOS에서 상당한 향상을 보임

	- HiFi-GAN (+S+C)는 객관적 및 주관적 평가 모두에서 최고의 성능을 보였으며

	이는 다른 판별기들이 서로에게 상호 보완적인 정보를 제공하여 공동 학습의 효과를 확인시켜 줌

<div align="center">
	<img src="/assets/post/msct3.png" alt="Introduction of Melfusion">
	<p><em>[그림 3] 실험 결과 및 표</em></p>
</div>

2. EQ3

	- MelGAN과 NSF-HiFiGAN의 성능은 MS-SB-CQT 및 MS-STFT 판별기와의 공동 학습을 통해 크게 향상됨 

	- 특히, MelGAN은 저주파 부분에 과적합되고 중고주파 구성 요소를 무시하여 노이즈를 발생 

	- MS-STFTD 및 MS-SB-CQTD를 추가한 후, 스펙트로그램의 전역 정보를 더 잘 모델링하여 더 나은 결과를 보임

	- 비록 저주파 관련 지표는 악화되었지만, 선호도 테스트는 전반적인 품질이 현저하게 증가했음을 보여줌

	- NSF-HiFiGAN은 고품질의 노래 음성을 합성할 수 있지만, 여전히 주파수 디테일이 부족 

	- MS-STFT 및 MS-SB-CQT 판별기를 추가하면 이 문제가 해결됨

<div align="center">
	<img src="/assets/post/msct4.png" alt="Introduction of Melfusion">
	<p><em>[그림 4] 실험 결과 및 표</em></p>
</div>

오늘 포스팅 여기서 마무리하도록 하겠습니다. 감사합니다.