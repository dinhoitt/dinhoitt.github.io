---
layout: post
title:  "Transformer, Attention Is All You Need 이란?"
date:   2024-08-19T20:08:52+09:00
author: DINHO
categories:
  - 인공지능-분야-공부
  - 논문-리뷰
cover:  "/assets/post/transformer.png"
---
오랜만에 논문 리뷰입니다. 지난 번 논문 리뷰 때 [MusicGen](https://dinhoitt.github.io/%EC%9D%B8%EA%B3%B5%EC%A7%80%EB%8A%A5-%EB%B6%84%EC%95%BC-%EA%B3%B5%EB%B6%80/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0/2024/08/05/MusicGen%EC%9D%B4%EB%9E%80.html) 이야기를 했었는데요! MusicGen Decoder의 기반이 되기도 하고, 인공지능을 공부하는 사람이라면 절대로 몰라선 안 되는!!! Transformer, Attention Is All You Need(2017) 논문 리뷰를 해보겠습니다.

1. __연구 배경__

    기존 RNN(Recurrent Neural Network)과 CNN(Convolutional neural network) 기반 모델은 긴 문맥을 학습하기 어렵고, 병렬처리가 어렵다는 단점이 있습니다. Transformer 모델은 이러한 문제를 해결하기 위해 고안되었습니다.

    - __순환 구조 제거__ : RNN의 순차적인 계산 방식을 없애고 병렬 처리가 가능하게 함으로써 학습 속도를 높임

    - __Self-Attention__ : 입력의 각 단어가 다른 모든 단어와 직접 연결되어 긴 거리의 종속 관계를 쉽게 학습할 수 있도록 함.

2. __Model Arcitecture__

    <img src="/assets/post/transformer.png">
    _그림1 - 트랜스포머 모델 구조_

    Transformer는 인코더-디코더 구조를 따르며, 일반적인 Sequence to Sequence 작업에 적합하게 설계되었습니다. 그림의 왼쪽과 오른쪽 부분에 표시된 것처럼 인코더와 디코더 모두에 stacked self-attention and point-wise, fully connected layers를 사용하는 아키텍처를 따릅니다.

    ### 2.1 Encoder and Decoder Stacks ###

    - Encoder : 인코더는 동일한 레이어의 스택(Stack of identical layer)으로 구성되어 있습니다(일반적으로 N=6). 각 레이어는 두 서브레이어(Sub-layer)가 있습니다. 첫 번째는 __multi-head self-attention(멀티헤드 셀프어텐션)__ 이고 두 번째는 __position-wise fully connected feed-forward network__ 입니다. 이 때 두 서브 레이어에 잔차 연결(residual connection)을 사용한 다음, 레이어 정규화를 수행합니다. 즉, $Sublayer Output = LayerNorm(x + Sublayer(x))$ 함수 식으로 표현할 수 있습니다. 이 때 x는 서브 레이어의 입력, $Sublayer(x)$ 는 해당 서브레이어의 출력이고, 이후에 레이어 정규화를 진행합니다. 이러한 잔차 연결을 용이하게 하기 위해 모든 모델의 서브레이어와 임베딩 레이어는 512 차원의 output을 생상합니다.

    - Decoder : 디코더 역시 동일한 레이어의 스택으로 구성되어 있습니다. 다만 디코더에서는 인코더와 달리 세 개의 서브레이어가 있습니다. 새로 추가된 서브레이어는 인코더 스택의 아웃풋에 대한 멀티헤드 어텐션을 수행합고, 이외의 구조는 인코더와 같습니다. 이 때 디코더 스텍에서 셀프어텐션 서브레이어가 이전과 현재 위치만 참조하고, 이후의 위치를 참조하지 않도록 위치를 수정했습니다. 이 이유는 디코더가 __순차적 예측(autoregressive prediction)__ 을 수행하기 때문입니다. 즉, 디코더는 한 번에 하나씩 다음 단어를 예측하는 방식으로 작동하며, 이전에 예측한 단어들만을 참고해서 연재 단어를 예측할 수 있어야 합니다. 다시 말하면, 각 위치가 미래 단어들을 참조하지 못하도록 해야 합니다. 이를 구현하기 위해 __마스킹__ 기법을 사용합니다. 

    ### 2.2 Attention ###

    <img src="/assets/post/attention.png">
    _그림2 - (왼쪽) Scaled Dot-Product Attention. (오른쪽) Multi-Head Attention은 병렬로 실행되는 여러 어텐션 레이어로 구성_

    어텐션 함수는 Query와 key-value 세트를 출력에 매핑하는 것으로 설명할 수 있으며, 여기서 Q, K, V는 모두 벡터입니다. 출력은 V의 가중 합(weighted sum)으로 계산되며, 각 값에 할당된 가중치는 해당 K와 Q의 호환성 함수(Compatibility function)에 의해 계산됩니다.

    - Scaled Dot-Product Attention : 입력은 K와 V의 차원 $d_{K}, d_{V}$ 로 구성됩니다. 수식은 아래와 같습니다. 

    $$
    \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V
    $$

    
