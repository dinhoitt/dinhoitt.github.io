---
layout: post
title:  "Vision-Language-Action(VLA)란?"
date:   2026-07-02T15:08:52+09:00
author: DINHO
categories:
  - 인공지능-분야-공부
  - 논문-리뷰
cover:  "/assets/post/vla.png"
---

오늘은 **Vision-Language-Action(VLA)** 에 대해서 이야기해보겠습니다. 최근 로보틱스 분야를 공부하다 보면 VLM(Vision-Language Model)과 함께 정말 자주 보이는 단어인데요!! 간단하게 이야기하면 VLA는 **보고(Vision), 명령을 이해하고(Language), 실제 행동(Action)까지 생성하는 모델**입니다.

이번 포스팅은 Sapkota, Ranjan, et al. **"Vision-Language-Action (VLA) Models: Concepts, Progress, Applications and Challenges"** 논문을 중심으로 정리해보겠습니다. 이 논문은 최근 3년 동안 발표된 80개 이상의 VLA 모델을 정리한 리뷰 논문이라 VLA를 처음 공부할 때 전체적인 흐름을 잡기에 좋습니다.

논문 내용이 굉장히 많기 때문에 모든 모델을 하나씩 소개하기보다는 제가 중요하다고 생각한 **VLA의 개념, 발전 과정, 학습 방식, 아키텍처, 한계와 앞으로의 방향** 을 중심으로 이야기해보겠습니다. 😄

# Introduction

VLA가 왜 등장했는지부터 먼저 보겠습니다.

VLA 이전의 인공지능은 크게 **Vision, Language, Action이 서로 독립적으로 발전** 했습니다.

- Vision Model: 이미지를 보고 객체를 검출하거나 분류할 수 있음

- Language Model: 텍스트를 이해하고 생성할 수 있음

- Action Model: 로봇을 움직이고 특정 행동을 수행할 수 있음

각각의 분야만 놓고 보면 굉장히 많은 발전이 있었지만, 문제는 이 세 가지를 **하나의 시스템 안에서 자연스럽게 연결하는 것** 이 어려웠다는 점입니다.

예를 들어 비전 모델은 사과를 보고 "사과"라고 인식할 수 있습니다. 언어 모델은 "사과를 집어라"라는 문장을 이해할 수 있습니다. 로봇 제어 시스템은 미리 정의된 방식으로 그리퍼를 움직일 수도 있습니다.

그런데 실제 사람처럼

**사과를 본다 → 명령을 이해한다 → 어디에 있는지 판단한다 → 팔을 움직인다 → 사과를 잡는다**

라는 전체 과정을 하나의 유연한 시스템으로 만드는 것은 다른 문제입니다.

<div align="center">
	<img src="/assets/post/vla2.png" alt="Evolution from isolated modalities to unified VLA models">
	<p><em>[그림 1] Vision, Language, Action에서 VLA로의 발전</em></p>
</div>

[그림 1]이 이 내용을 굉장히 잘 보여줍니다. 기존 시스템은 Vision Model, Language Model, Action Model이 각각 역할을 수행하고 그 사이를 별도의 파이프라인으로 연결했습니다.

이런 구조는 특정 환경에서는 잘 동작할 수 있지만 새로운 객체, 새로운 명령, 새로운 환경이 등장했을 때 쉽게 무너질 수 있습니다. 특히 로봇의 행동 시스템은 수작업으로 만든 정책(hand-crafted policy)이나 강화학습에 크게 의존해왔기 때문에 **학습하거나 설계한 상황을 넘어서는 일반화** 가 쉽지 않았습니다.

VLM(Vision-Language Model)이 등장하면서 이미지와 언어를 함께 이해하는 능력은 크게 발전했습니다. 하지만 VLM도 기본적으로는 **보고 이해하는 모델**입니다. 실제 로봇의 관절이나 그리퍼를 움직이는 행동까지 직접 연결하는 데에는 여전히 간극이 있었습니다.

이러한 문제를 해결하기 위해 VLA가 등장했습니다.

논문에서는 VLA가 2021~2022년경 개념화되었고, 이후 Google DeepMind의 **RT-2(Robotic Transformer 2)** 와 같은 연구를 통해 본격적으로 발전했다고 설명합니다.

한 문장으로 요약하면 다음과 같습니다.

> **VLM은 보고 말하지만, VLA는 보고 이해한 뒤 행동합니다.**

# Concepts of VLA Models

그렇다면 본격적으로 VLA가 어떤 모델인지 알아보겠습니다.

## VLA의 정의

논문에서는 VLA를 **시각 입력을 처리하고, 자연어 명령을 해석하며, 실제 로봇 하드웨어에서 실행할 수 있는 행동 표현을 생성하는 지능형 시스템**으로 정의합니다.

<div align="center">
	<img src="/assets/post/vla.png" alt="Foundational Concept of VLA Models">
	<p><em>[그림 2] Foundational Concept of VLA Models</em></p>
</div>

[그림 2]에서는 사과를 따는 로봇의 예시를 통해 VLA의 핵심 과정을 보여줍니다.

1. **Multimodal Integration**: 이미지와 언어 등 여러 정보를 함께 처리

2. **Tokenization and Representation**: 입력 정보와 로봇의 상태, 행동을 모델이 처리할 수 있는 표현으로 변환

3. **Learning Paradigms**: 웹 데이터와 로봇 행동 데이터를 이용해 학습

4. **Adaptive Control and Real-Time Execution**: 실제 환경에서 행동하고 피드백을 받아 수정

중요한 점은 VLA가 단순히 "이미지 이해 모델 + 언어 모델 + 로봇 제어 알고리즘"을 이어 붙인 것만을 의미하는 것이 아니라는 점입니다.

현대 VLA 연구의 핵심은 **Vision, Language, Action을 하나의 표현 공간 또는 긴밀하게 연결된 구조에서 함께 다루는 것**에 있습니다.

예를 들어 "빨간 사과를 집어라"라는 명령을 받았다고 해보겠습니다.

카메라 영상에서 사과를 찾음
-> 여러 사과 중 빨간 사과가 어떤 것인지 판단
-> "집어라"라는 언어 명령의 의미를 해석
-> 현재 로봇 팔의 자세를 확인
-> 사과까지 이동할 경로를 생성
-> 그리퍼를 닫는 행동을 생성

이 모든 과정이 VLA가 해결하려는 문제입니다.

## VLA의 발전 과정

논문에서는 2022년부터 2025년까지 VLA의 발전을 크게 세 단계로 나눕니다.

<div align="center">
	<img src="/assets/post/vla3.png" alt="Timeline of VLA Models">
	<p><em>[그림 3] VLA 모델의 발전 과정</em></p>
</div>

### 1. Foundational Integration (2022-2023)

초기에는 **Vision-Language 표현을 로봇 정책과 연결하는 것** 자체가 중요한 연구 주제였습니다.

CLIPort, Gato, RT-1, VIMA와 같은 모델이 등장했고, 이후 RT-2, VoxPoser, Diffusion Policy 등이 발전했습니다.

이 시기의 핵심은 "이미지와 언어를 이해하는 모델의 지식을 어떻게 실제 로봇 행동으로 연결할 것인가?"라고 볼 수 있습니다.

### 2. Specialization and Embodied Reasoning (2024)

2024년에는 단순한 통합을 넘어 **3D 공간 이해, 기억 효율성, 부분 관측(Partial Observability), 객체 중심 표현** 등 실제 로봇이 부딪히는 문제를 해결하는 방향으로 발전합니다.

즉, "무엇을 해야 하는가?"만 아는 것이 아니라 **실제 3차원 공간에서 어떻게 행동해야 하는가?**에 대한 연구가 많아졌습니다.

### 3. Generalization and Safety-Critical Deployment (2025)

최근에는 **일반화와 안전성** 이 굉장히 중요해졌습니다.

새로운 로봇, 새로운 환경, 새로운 객체에서도 잘 동작하는 Cross-Embodiment, Sim-to-Real, Human-in-the-loop, Safety Verification 등이 연구되고 있습니다.

정리하면 VLA는 처음에는 Vision-Language와 로봇 Policy를 연결하는 수준에서 시작했지만, 점점 **3D 공간 이해 → 기억 → 안전성 → 일반화 → 인간과의 상호작용** 까지 포함하는 방향으로 확장되고 있습니다.

## Multimodal Integration

기존 로봇 시스템은 보통 다음과 같은 형태를 가집니다.

```text
Perception Module → Planner → Controller
```

예를 들어 Perception Module이 카메라 영상에서 "사과"라는 라벨을 출력하면 Planner가 사과를 잡기 위한 경로를 만들고 Controller가 실제 모터 명령을 수행합니다.

이 방식은 각 모듈을 독립적으로 설계할 수 있다는 장점이 있지만, 모듈 사이의 인터페이스를 사람이 직접 정해야 하고 예상하지 못한 상황에서 유연하게 대응하기 어렵습니다.

반대로 VLA는 대규모 Pretrained Encoder와 Transformer를 이용하여 **시각 관찰과 언어 명령을 같은 계산 공간에서 해석** 하려고 합니다.

예를 들어 "익은 사과를 따라"라는 명령이 들어왔다고 해보겠습니다.

- Vision Encoder: 과일, 잎, 배경, 사과의 질감이나 모양 등 시각적인 특징을 인식

- Language Model: "익은 사과"와 "따라"라는 명령의 의미를 임베딩

- Multimodal Fusion: Cross-Attention 또는 Joint Tokenization을 통해 두 정보를 결합

- Action Policy: 결합된 정보를 이용해서 실제 행동 생성

여기서 중요한 것은 **Semantic Grounding** 입니다. 단순히 "사과"라는 단어와 픽셀을 연결하는 것이 아니라, 현재 상황에서 언어가 가리키는 실제 객체와 행동 가능성(Affordance)을 연결해야 합니다.

## Tokenization and Representation

개인적으로 VLA를 이해할 때 가장 중요한 부분 중 하나라고 생각합니다.

기존 VLM은 이미지와 텍스트를 토큰으로 표현합니다. VLA는 여기에 **로봇의 상태(State)와 행동(Action)까지 토큰 또는 모델이 처리할 수 있는 표현으로 추가** 합니다.

<div align="center">
	<img src="/assets/post/vla4.png" alt="Tokenization in VLA Models">
	<p><em>[그림 4] VLA의 Prefix, State, Action Token</em></p>
</div>

### Prefix Token

Prefix Token은 환경의 이미지나 비디오와 자연어 명령을 담습니다.

예를 들어

> "초록 블록을 빨간 쟁반 위에 쌓아라"

라는 명령과 테이블의 이미지가 들어오면 Vision Encoder와 Language Encoder를 거쳐 Prefix Token이 만들어집니다.

쉽게 이야기하면 **지금 어떤 상황이고, 무엇을 해야 하는지** 를 담는 정보입니다.

### State Token

State Token은 로봇 자신의 현재 상태를 나타냅니다.

대표적으로 다음과 같은 정보가 들어갈 수 있습니다.

- Joint Position

- Gripper State

- Force-Torque Sensor

- End-Effector Pose

- Odometry

- LiDAR

- Proprioception

<div align="center">
	<img src="/assets/post/vla5.png" alt="State Tokens in real-world scenarios">
	<p><em>[그림 5] 실제 로봇 조작과 내비게이션에서 State Token의 역할</em></p>
</div>

[그림 5]의 위쪽을 보면 로봇 팔이 깨지기 쉬운 물체 근처까지 접근한 상황이 나옵니다. 이때 단순히 카메라와 언어 정보만 보는 것이 아니라 현재 관절 각도, End-Effector 위치와 같은 State 정보를 함께 사용하면 충돌 가능성을 판단하고 경로를 수정할 수 있습니다.

이동 로봇에서는 LiDAR, Odometry 등의 정보가 State 역할을 할 수 있습니다.

즉 State Token은 로봇에게 **"지금 내 몸이 어떤 상태인가?"** 를 알려주는 정보라고 생각하시면 됩니다.

### Action Token

마지막은 Action Token입니다.

Action Token은 모델이 다음에 수행할 로봇 행동을 나타냅니다. Joint Angle Update, Torque, Wheel Velocity 같은 Low-Level Control Signal일 수도 있고, "grasp", "move", "rotate"와 같은 High-Level Action Primitive일 수도 있습니다.

<div align="center">
	<img src="/assets/post/vla6.png" alt="How VLAs encode the world">
	<p><em>[그림 6] Vision, Language, State를 Action으로 변환하는 VLA Pipeline</em></p>
</div>

여기서 굉장히 재미있는 점이 있습니다.

LLM은 앞의 단어들을 보고 다음 단어를 하나씩 예측하면서 문장을 생성하죠? VLA도 비슷하게 **앞의 Vision, Language, State 정보와 이전 행동을 보고 다음 Action을 예측** 할 수 있습니다.

예를 들어 사과 따기 작업이라면

```text
팔을 사과 방향으로 이동 → 손목 회전 → 그리퍼 접근 → 그리퍼 닫기 → 사과 들어올리기
```

와 같은 행동 시퀀스가 생성됩니다.

즉 Transformer의 관점에서 보면 기존에는 **문장을 생성했다면, 이제는 동작을 생성하는 것** 입니다.

> 문장은 곧 동작입니다!!

이 아이디어가 RT-2와 같은 VLA를 이해할 때 정말 중요합니다.

## Learning Paradigms

그렇다면 이런 모델은 무엇을 보고 학습할까요?

VLA를 학습시키기 위해서는 크게 두 종류의 데이터가 필요합니다.

<div align="center">
	<img src="/assets/post/vla7.png" alt="Learning Paradigms for VLA Models">
	<p><em>[그림 7] VLA의 학습 데이터와 Co-Fine-Tuning</em></p>
</div>

### 1. Web-Scale Vision-Language Data

먼저 인터넷 규모의 Vision-Language 데이터입니다.

- Image-Caption Pair

- Visual Question Answering(VQA)

- Instruction-Following Data

- Video-Text Data

이 데이터를 이용하면 모델이 "컵", "사과", "문", "집는다", "왼쪽", "위에 있다" 같은 **세상의 의미적 지식(Semantic Prior)** 을 배울 수 있습니다.

CLIP과 같은 방식으로 이미지와 텍스트를 같은 Embedding Space에 정렬하는 것도 여기에 포함됩니다.

### 2. Robot Trajectory Data

하지만 인터넷 데이터만으로는 로봇을 움직일 수 없습니다.

그래서 실제 로봇이나 시뮬레이터에서 수집한 Robot Trajectory Data가 필요합니다.

Trajectory에는 카메라 영상뿐 아니라 Joint State, End-Effector Pose, Gripper State, Action Command 등이 포함될 수 있습니다.

이 데이터는 모델에게 **"이 장면과 이 명령이 주어졌을 때 실제로 어떻게 움직여야 하는가?"** 를 알려줍니다.

논문에서는 RoboNet, BridgeData, RT-X 같은 Robot Dataset을 예로 들고 있으며 Demonstration은 Teleoperation, Kinesthetic Teaching, Scripted Policy 등을 통해 수집할 수 있다고 설명합니다.

학습 방법으로는 Supervised Learning, Behavior Cloning, Imitation Learning, Reinforcement Learning 등이 사용됩니다.

최근에는 보통

```text
Vision-Language Pretraining → Robot Demonstration Fine-tuning
```

형태의 Multi-Stage Training을 많이 사용합니다.

RT-2도 Web-Scale Multimodal Data와 Robot Demonstration을 함께 학습하여 새로운 객체나 새로운 명령에 대한 Zero-Shot Generalization 능력을 얻었다고 설명합니다.

쉽게 이야기하면 **웹 데이터로 세상을 배우고, 로봇 데이터로 행동을 배운다** 고 생각하시면 될 것 같습니다.

## Adaptive Control and Real-Time Execution

VLA의 또 다른 중요한 목표는 **실시간 적응 제어(Adaptive Control)** 입니다.

기존 방식에서는 한 번 계획한 경로를 그대로 실행하는 경우가 많았습니다. 하지만 실제 환경은 계속 변합니다.

사람이 갑자기 지나갈 수도 있고, 사과가 바람에 움직일 수도 있고, 조명이 변하거나 물체가 가려질 수도 있습니다.

VLA에서는 실행 중에도 카메라와 센서 정보를 다시 입력받고 State를 갱신하면서 행동을 수정할 수 있습니다.

즉 VLA의 목표는 단순히

> "무엇을 할지 계획한다"

가 아니라

> **"실제로 행동하면서 변화한 환경을 다시 보고, 계속 계획을 수정한다"**

에 가깝습니다.

# Progress in VLA Models

이제 VLA가 최근 어떤 방향으로 발전하고 있는지 살펴보겠습니다.

논문에서는 Transformer 기반 LLM과 VLM의 성공이 VLA 발전의 중요한 출발점이었다고 설명합니다. LLM이 강력한 Semantic Reasoning 능력을 보여주고, GPT-4와 같은 모델이 이미지와 텍스트를 함께 처리하기 시작하면서 이러한 Foundation Model을 실제 Robotics로 확장하려는 연구가 자연스럽게 이어졌습니다.

또한 RT-1과 같은 대규모 Robot Demonstration Dataset이 등장하면서 Vision, Language, Action을 함께 학습할 수 있는 데이터 기반도 만들어졌습니다.

그 결과 RT-2에서는 로봇 제어를 **Autoregressive Sequence Prediction** 문제처럼 다루는 대표적인 VLA 구조가 등장했고, Octo 같은 Open-Source Generalist Robot Policy도 발전했습니다.

## Architectural Innovation

논문에서는 최근 VLA의 대표적인 아키텍처 발전을 크게 몇 가지 방향으로 설명합니다.

### Early Fusion Models

Early Fusion은 이름 그대로 **Vision과 Language 정보를 초기에 결합** 하는 방식입니다.

대표적으로 논문에서 소개하는 EF-VLA는 CLIP의 Frozen Encoder를 이용해 이미지와 텍스트 특징을 추출하고, Action Prediction 전에 두 표현을 먼저 융합합니다.

CLIP에서 이미 학습된 Vision-Language Alignment를 유지하면서 로봇 행동 부분만 학습할 수 있기 때문에 Overfitting을 줄이고 Generalization을 높이려는 접근입니다.

### Dual-System Architecture

두 번째는 NVIDIA의 GR00T N1과 같은 **Dual-System Architecture** 입니다.

인간의 사고 과정에서 영감을 받아 두 개의 시스템으로 나눕니다.

- **System 1**: 빠른 반응과 Low-Level Control

- **System 2**: 느리지만 복잡한 추론과 High-Level Planning

예를 들어 "테이블을 정리해"라는 명령이 들어오면 System 2는

```text
컵을 치운다 → 접시를 옮긴다 → 쓰레기를 버린다
```

처럼 큰 작업을 작은 Subtask로 나눌 수 있습니다.

반면 System 1은 실제 End-Effector 위치를 조절하고, 물체를 잡고, 충돌을 피하는 빠른 제어를 담당합니다.

이 구조는 개인적으로 로봇에서 굉장히 자연스러운 방향이라고 생각합니다. 실제 로봇에서는 복잡한 추론도 필요하지만, 동시에 수 ms~수십 ms 단위의 빠른 제어도 필요하기 때문입니다.

### Self-Correcting Framework

세 번째는 **실패를 감지하고 스스로 수정하는 구조** 입니다.

SC-VLA와 같은 구조에서는 평소에는 빠른 Inference Path를 사용하지만 Grasp Failure나 Collision 같은 문제가 감지되면 느린 Correction Path를 활성화합니다.

이 Correction Path에서는 실패 원인을 다시 추론하고 새로운 행동을 생성합니다.

사람도 컵을 집으려다 놓치면 같은 행동을 무한히 반복하는 것이 아니라 "손 위치가 잘못됐나?", "컵이 가려졌나?"를 다시 보고 행동을 바꾸죠. VLA도 이러한 **Failure Recovery** 능력을 갖추는 방향으로 발전하고 있습니다.

### VLA의 Design Space

VLA 구조를 공부하면 모델마다 구조가 너무 달라서 헷갈릴 수 있습니다. 논문에서는 이를 몇 가지 축으로 정리합니다.

#### End-to-End vs. Modular

**End-to-End** 는 입력부터 행동까지 하나의 통합된 모델에서 학습하는 방식입니다. 구조가 단순하고 전체 최적화가 가능하지만, 어느 부분에서 문제가 발생했는지 분석하기 어려울 수 있습니다.

반대로 **Modular** 구조는 Perception, Reasoning, Planning, Control 등을 분리합니다. 각 모듈을 교체하거나 분석하기 쉽지만 모듈 사이의 Interface 설계가 필요합니다.

#### Hierarchical vs. Flat

**Flat Policy** 는 하나의 정책이 바로 행동을 생성합니다.

**Hierarchical Policy** 는 High-Level Planner와 Low-Level Controller를 나누어 긴 작업을 처리합니다.

예를 들어 "주방을 정리해"라는 명령을 High-Level Planner가 여러 단계로 나누고, Low-Level Controller가 각 단계의 실제 움직임을 담당할 수 있습니다.

#### Low-Level Policy vs. High-Level Planner

Low-Level Policy는 Joint, Torque, End-Effector Trajectory처럼 정밀한 실제 제어에 강합니다.

High-Level Planner는 긴 시간 범위의 목표, Subgoal 생성, Reasoning에 강합니다.

결국 최근 VLA는 하나의 구조가 정답으로 수렴한다기보다 **End-to-End 통합성, 모듈성, 계층성, 실시간 제어 능력 사이의 Trade-Off를 조절하는 방향** 으로 발전하고 있습니다.

## Training and Efficiency

VLA가 실제 로봇에서 사용되기 위해서는 모델 성능뿐 아니라 학습 비용과 추론 속도가 중요합니다.

### Data-Efficient Learning

로봇 데이터는 웹 이미지나 텍스트처럼 쉽게 수집할 수 없습니다. 로봇을 직접 움직여 데이터를 얻어야 하기 때문에 비용이 매우 큽니다.

그래서 다음과 같은 방법들이 연구됩니다.

- **Co-Fine-Tuning**: Vision-Language Data와 Robot Trajectory Data를 함께 사용

- **Synthetic Data Generation**: 시뮬레이터를 이용해 부족한 상황을 생성

- **Self-Supervised Pretraining**: Label이 없는 대규모 데이터에서 표현을 먼저 학습

- **Sim-to-Real**: Simulation에서 학습한 정책을 실제 로봇으로 전달

논문에서는 UniSim을 이용해 Occlusion이나 Dynamic Lighting 같은 드문 Edge Case를 합성하는 방법도 소개합니다.

### Parameter-Efficient Adaptation

대표적인 방법은 **LoRA(Low-Rank Adaptation)** 입니다.

거대한 VLA 전체 파라미터를 다시 학습하지 않고 작은 Adapter만 학습해서 새로운 도메인이나 로봇에 적응하는 방식입니다.

로봇마다 하드웨어와 센서가 다르다는 것을 생각하면 굉장히 중요한 기술입니다. 로봇이 바뀔 때마다 수십억 개의 파라미터를 처음부터 다시 학습하는 것은 현실적으로 부담이 크기 때문입니다.

### Inference Acceleration

실제 로봇은 실시간으로 움직여야 하므로 추론 속도가 매우 중요합니다.

논문에서는 FAST와 같은 **Compressed Action Tokenization** , Parallel Decoding, Action Chunking, Quantization, Pruning, Hardware-Aware Optimization 등을 소개합니다.

예를 들어 일반 Autoregressive 방식은 Action을 하나씩 순차적으로 생성하기 때문에 길이가 길어질수록 Latency가 누적됩니다. 반대로 Action Chunking은 여러 Step의 행동을 하나의 Chunk로 묶어 한 번에 생성해서 추론 횟수를 줄일 수 있습니다.

결국 VLA에서는 **정확한 행동을 생성하는 것만큼 빠르게 생성하는 것도 중요합니다.**

# Applications of VLA Models

VLA는 다양한 로봇 분야에 적용되고 있습니다.

## Humanoid Robotics

Humanoid Robot은 사람과 비슷한 환경에서 사람과 비슷한 방식으로 움직여야 하기 때문에 VLA의 대표적인 응용 분야입니다.

카메라를 통해 주변 환경을 이해하고, 사람의 자연어 명령을 해석하고, 양팔이나 전신을 이용해 행동해야 합니다.

특히 장시간의 Task Planning과 빠른 전신 제어가 동시에 필요해서 앞서 설명한 Hierarchical 또는 Dual-System 구조와도 잘 연결됩니다.

## Autonomous Vehicle

자율주행에서도 카메라와 다양한 센서를 해석하고 상황을 이해한 뒤 행동을 생성해야 합니다.

단순히 차선을 검출하는 것을 넘어 "공사 구간이니 천천히 우회해"와 같은 언어적·의미적 판단을 주행 행동과 연결할 수 있다는 점에서 VLA의 적용 가능성이 있습니다.

## Industrial Robotics

산업용 로봇에서는 조립, 검사, 협동 작업 등에 활용할 수 있습니다.

기존 산업용 로봇은 정해진 환경과 반복적인 작업에는 매우 강하지만 작업 대상이나 공정이 바뀌면 다시 Programming해야 하는 경우가 많습니다.

VLA는 자연어 명령과 시각 정보를 이용해 새로운 작업에 더 유연하게 적응하는 방향으로 연구되고 있습니다.

## Healthcare and Medical Robotics

논문에서는 의료 로봇을 **High-Stakes Domain** 으로 이야기합니다.

의료 분야는 단순히 "성공률이 높다"만으로 충분하지 않습니다. 아주 작은 위치 오차도 문제가 될 수 있고, 안전성, 신뢰성, 설명 가능성이 중요합니다.

따라서 VLA의 강력한 Semantic Reasoning 능력은 매력적이지만 동시에 **Sub-Millimeter Precision, Safety Assurance, Real-Time Control** 같은 문제가 해결되어야 합니다.

## Precision Agriculture

농업에서는 과일 수확, 작물 모니터링, 이상 탐지, 선택적 살포 등에 적용할 수 있습니다.

예를 들어 "익은 사과만 수확해"라는 명령은 단순 객체 검출만으로는 충분하지 않습니다. 사과의 상태를 판단하고, 장애물을 피하고, 로봇 팔의 경로를 계획하고, 적절한 힘으로 파지해야 합니다.

논문에서 사과 수확을 VLA의 대표적인 예제로 계속 사용하는 이유도 이런 복합적인 문제를 직관적으로 보여주기 때문인 것 같습니다. 🍎

## Interactive AR Navigation

논문에서 재미있게 본 응용 중 하나가 AR Navigation입니다.

<div align="center">
	<img src="/assets/post/vla9.png" alt="Interactive AR Navigation with VLA">
	<p><em>[그림 9] Interactive AR Navigation with VLA</em></p>
</div>

Smart Glasses나 Smartphone에서 들어오는 연속적인 영상과 자연어 Query를 함께 처리하여 사용자가 실제로 보고 있는 장면 위에 Navigation 정보를 Overlay하는 방식입니다.

예를 들어 공항에서

> "계단을 사용하지 않고 22번 Gate까지 어떻게 가?"

라고 물으면 카메라로 현재 위치와 주변 시설을 이해하고, 사용자의 조건을 반영한 경로를 실제 시야 위에 보여주는 형태입니다.

이 경우에도 결국 **Vision + Language + Action/Navigation**의 결합이라고 볼 수 있습니다.

# Challenges and Limitations of VLA Models

지금까지 보면 VLA가 로봇의 모든 문제를 해결할 것처럼 보이지만 실제로는 해결해야 할 문제가 굉장히 많습니다.

논문에서도 상당한 분량을 VLA의 한계에 할애하고 있습니다.

## Real-Time Inference

첫 번째 문제는 **추론 속도** 입니다.

VLA는 이미지, 언어, 로봇 상태 등 굉장히 많은 정보를 Transformer에 입력합니다. 또한 Autoregressive 방식이라면 Action을 순차적으로 생성해야 합니다.

논문에서는 복잡한 시나리오에서 일부 VLA 시스템이 **3~5 Hz 수준의 제어 주파수** 에 머무를 수 있다고 지적합니다.

사람과 천천히 상호작용하는 작업에서는 가능할 수 있지만 빠르게 움직이는 물체를 잡거나 Humanoid의 균형을 제어하는 작업에서는 충분하지 않을 수 있습니다.

특히 의료 로봇처럼 높은 정밀도가 필요한 분야에서는 속도를 높이기 위해 모델을 단순화했을 때 Accuracy가 떨어지는 것도 문제가 됩니다.

즉,

> **정확하게 생각하려면 느리고, 빠르게 움직이려면 충분히 생각하기 어렵다**

라는 Trade-Off가 존재합니다.

## Multimodal Action Representation

두 번째는 Action을 어떻게 표현할 것인가에 대한 문제입니다.

Action을 너무 이산적인 Token으로 나누면 정밀한 제어가 어려워집니다. 반대로 완전히 연속적인 Distribution으로 표현하면 학습 안정성이나 Mode Collapse 같은 문제가 생길 수 있습니다.

Diffusion Policy를 사용하면 복잡하고 다양한 Action Distribution을 표현하기 좋아지지만 Sampling 과정 때문에 추론이 느려질 수 있습니다.

결국 **정밀도, 표현력, 속도 사이의 균형** 이 중요합니다.

## Safety Assurance in Open Worlds

실제 환경은 학습 데이터처럼 정돈되어 있지 않습니다.

사람이 갑자기 로봇 앞에 나타날 수도 있고, 물체 위치가 바뀌거나 센서가 잘못된 값을 줄 수도 있습니다.

현재 많은 로봇 시스템은 힘, 토크, 속도 등의 하드코딩된 임계값을 이용하여 안전 장치를 구성합니다. 하지만 Open-World 환경에서는 사전에 모든 위험 상황을 정의하는 것이 어렵습니다.

VLA가 더 높은 수준의 자율성을 가지게 될수록 **모델이 틀렸을 때 어떻게 안전하게 멈출 것인가?** 가 굉장히 중요합니다.

## Dataset Bias and Grounding

VLA가 사용하는 Vision-Language Backbone은 웹에서 수집된 대규모 데이터로 학습되는 경우가 많습니다.

그래서 기존 Web Dataset의 Bias를 그대로 가져올 수 있습니다.

논문에서는 예시로 "doctor"라는 단어를 남성과 과도하게 연결하는 Bias를 이야기합니다.

또한 각각의 단어는 잘 알고 있어도 학습 데이터에서 드문 조합을 만나면 문제가 생길 수 있습니다. 논문에서는 **"yellow horse"** 같은 비전형적인 조합을 예로 듭니다.

즉 모델이 "yellow"도 알고 "horse"도 알고 있다고 해서 항상 "yellow horse"를 올바르게 Grounding한다는 보장은 없습니다.

## Generalization to Unseen Tasks

VLA의 가장 큰 목표 중 하나는 Generalist Robot이지만 현재는 학습한 환경과 크게 다른 Task에서 성능이 떨어질 수 있습니다.

논문에서는 새로운 Task나 Distribution Shift에서 성능이 **최대 40%까지 감소할 수 있는 사례** 를 언급합니다.

원인은 모델이 특정 Training Distribution에 Overfitting되거나 충분히 다양한 Task Representation을 경험하지 못했기 때문입니다.

로봇에서는 카메라 위치, 조명, 물체 모양, 로봇 팔의 형태가 조금만 바뀌어도 입력 분포가 달라질 수 있기 때문에 이 문제가 더 어렵습니다.

## System Complexity and Computational Demands

VLA는 Vision Encoder, Language Model, Multimodal Fusion, Memory, Planner, Action Decoder, Controller 등 많은 요소를 포함할 수 있습니다.

모델이 커질수록 학습과 추론 비용도 증가합니다.

서버 GPU에서 동작하는 데모를 만드는 것과 배터리로 움직이는 로봇에 모델을 넣는 것은 완전히 다른 문제입니다.

그래서 Parameter Efficiency, Quantization, Pruning, Edge Deployment 연구가 중요합니다.

## Robustness and Ethics

마지막으로 실제 서비스에 적용하기 위해서는 Robustness뿐 아니라 Privacy, Bias, Accountability, Human Oversight 같은 문제도 해결해야 합니다.

로봇은 단순히 잘못된 문장을 출력하는 것이 아니라 **실제 물리적인 행동을 수행** 하기 때문에 LLM/VLM보다 안전 문제의 영향이 훨씬 직접적일 수 있습니다.

# Discussion

논문에서는 앞서 이야기한 문제에 대해 몇 가지 해결 방향을 제시합니다.

<div align="center">
	<img src="/assets/post/vla10.png" alt="Future Roadmap for VLA Models">
	<p><em>[그림 10] VLA의 Future Roadmap</em></p>
</div>

먼저 실시간 추론 문제에는 Model Pruning, Quantization, LoRA, Early-Exit, Efficient Tokenization과 같은 경량화 방법을 제시합니다.

Action과 Safety 문제에는 **Hybrid Policy Architecture** 가 중요한 방향으로 제시됩니다. 예를 들어 High-Level Planner는 Autoregressive VLA가 담당하고, 실제 정밀한 Low-Level Motion은 Diffusion Policy나 기존 Controller가 담당하는 방식입니다.

Dataset Bias와 Generalization 문제에는 Meta Learning, Transfer Learning, Domain Randomization, Sim-to-Real과 같은 방법을 활용할 수 있습니다.

저는 여기서 중요한 포인트가 하나 있다고 생각합니다.

**VLA가 발전한다고 해서 기존 로봇 제어 이론을 모두 없애는 방향은 아니라는 점입니다.**

논문에서도 VLA 기반 Planner가 유연한 Language-Conditioned Decision Making에는 강하지만, 현재의 Trajectory Accuracy와 Control Stability는 MPC(Model Predictive Control)나 Sampling-Based Motion Planning 같은 기존 방식보다 부족할 수 있다고 이야기합니다.

따라서 앞으로는

```text
VLA: 의미 이해 + 추론 + High-Level Planning
        ↓
Classical/Learned Controller: 정확하고 안정적인 Low-Level Control
```

같은 **Hybrid Architecture** 가 굉장히 중요할 것으로 보입니다.

# Future Roadmap

논문 마지막에서는 앞으로 VLA가 어떤 방향으로 발전할지 정리합니다.

## Foundation Model as a "Cortex"

현재 VLA는 VLM Backbone과 Task-Specific Policy Head를 연결하는 방식이 많습니다.

앞으로는 Image, Video, Text뿐 아니라 Interaction, Affordance, Dynamics까지 학습하는 거대한 Multimodal Foundation Model이 로봇의 **"Cortex(대뇌 피질)"** 역할을 할 수 있다고 이야기합니다.

즉 단순히 "이 물체가 컵이다"를 아는 것이 아니라

- 컵은 잡을 수 있음

- 깨질 수 있음

- 안에 액체가 있을 수 있음

- 기울이면 액체가 쏟아질 수 있음

같은 물리적인 상식까지 표현해야 한다는 것입니다.

## Agentic Self-Supervised Lifelong Learning

로봇은 한 번 학습한 뒤 끝나는 것이 아니라 실제 환경에서 계속 경험합니다.

앞으로는 실패 경험과 새로운 환경을 통해 스스로 학습하는 Continual/Lifelong Learning이 중요합니다.

다만 실제 로봇에서의 Continual Learning은 잘못된 경험을 학습하면 위험할 수 있기 때문에 **안전하고 감사 가능한(Auditable) 학습** 이 필요합니다.

## Hierarchical Neuro-Symbolic Planning

Deep Learning의 유연성과 Symbolic Planning의 해석 가능성을 결합하는 방향입니다.

긴 작업을 명시적인 Subgoal로 분해하고, 각 단계의 이유를 설명할 수 있다면 VLA의 Interpretability와 Safety를 높일 수 있습니다.

## World Model for Physical and Causal Reasoning

로봇이 행동하기 전에 "이 행동을 하면 어떤 일이 일어날까?"를 예측할 수 있다면 훨씬 안전하게 계획할 수 있습니다.

World Model은 환경의 변화를 내부적으로 예측해서 Physical/Causal Reasoning을 가능하게 하는 방향입니다.

## Efficiency and Scalability

아무리 좋은 모델이라도 실제 로봇에서 실시간으로 실행되지 않으면 사용하기 어렵습니다.

Parameter-Efficient Backbone, Early-Exit, Compact Action Tokenization, On-Device Cache, Hardware-Aware Compilation 등이 중요합니다.

## Cross-Embodiment Transfer

현재 로봇 데이터는 Robot Morphology에 크게 의존합니다.

7-DoF 로봇팔에서 학습한 정책을 다른 로봇팔이나 Humanoid에 그대로 사용할 수 있다면 데이터 활용성이 크게 높아집니다.

그래서 특정 로봇의 형태에 종속되지 않는 **Morphology-Agnostic Skill** 과 Cross-Embodiment Transfer가 중요한 연구 방향입니다.

## Evaluation Beyond Task Success

지금까지 로봇 연구에서는 "Task Success Rate가 몇 %인가?"가 굉장히 중요한 지표였습니다.

하지만 실제 로봇은 성공 여부만으로 평가하기 어렵습니다.

앞으로는 다음과 같은 요소도 함께 평가해야 합니다.

- Safety

- Energy Consumption

- Failure Recovery

- Latency

- Human Intervention

- Robustness

즉 **성공했느냐** 뿐 아니라 **얼마나 안전하고 효율적으로 성공했느냐** 가 중요해집니다.

## Safety, Ethics and Human-Centered Alignment

마지막으로 VLA가 사람과 함께 실제 공간에서 움직이게 된다면 Safety와 Ethics는 부가 기능이 아니라 모델 설계 단계부터 고려해야 합니다.

위험한 상황에서는 행동을 거부하거나 사람에게 다시 물어보고, 의사결정 과정을 기록하고, Privacy와 Bias를 검사하는 체계가 필요합니다.

# Conclusion

이번 포스팅에서는 VLA의 전체적인 개념을 리뷰 논문을 통해 살펴보았습니다.

가장 간단하게 다시 정리하면 VLA는

```text
Vision → 세상을 보고
Language → 명령과 상황을 이해하고
Action → 실제 행동을 생성하는 모델
```

입니다.

하지만 실제 구조를 살펴보면 단순히 세 모델을 붙여 놓은 것이 아니라 Vision, Language, Robot State, Action을 하나의 표현 공간에서 연결하고 Transformer의 Sequence Modeling 능력을 물리적인 행동 생성까지 확장한 모델이라고 볼 수 있습니다.

특히 저는 **Action Token** 이라는 개념이 VLA를 이해하는 가장 직관적인 포인트라고 생각합니다. LLM이 문장을 한 Token씩 생성하듯이 VLA는 로봇의 행동을 한 Step씩 생성합니다. 결국 Transformer가 "말"만 생성하는 것이 아니라 "행동"까지 생성하게 된 것이죠.

다만 아직은 해결해야 할 문제가 많습니다. Real-Time Inference, 정밀한 Low-Level Control, Safety, Dataset Bias, Generalization, Compute Cost 모두 실제 로봇에서는 굉장히 큰 문제입니다.

그래서 앞으로 VLA가 무조건 거대한 End-to-End 모델 하나로 발전하기보다는 **VLM/VLA의 의미적 추론 능력 + World Model + Hierarchical Planner + 기존의 안정적인 Controller** 가 적절히 결합되는 방향도 굉장히 중요할 것으로 보입니다.

최근 로보틱스 분야에서 Foundation Model, Humanoid, VLA가 정말 빠르게 발전하고 있는데요. 이번 글에서는 VLA의 전체적인 큰 그림을 보았으니 다음에는 OpenVLA, RT-2, π0, GR00T 같은 실제 VLA 모델들을 하나씩 자세하게 공부해보면 재미있을 것 같습니다. 😁😁

---
