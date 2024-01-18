---
layout: post
title:  "과비음 (Hypernasality) 측정 딥러닝 알고리즘 논문 리뷰1"
date:   2024-01-18T05:25:52+09:00
author: DINHO
categories: "논문-리뷰"
cover:  "/assets/post_논문리뷰1.png"
---

논문 리뷰에 앞서 첫 번째 블로그 포스팅을 하게 되어 너무 설레고 떨립니다!
부족한 부분이 있을 수 있지만 잘 부탁드립니다!!! 피드백은 언제나 환영입니다!!

이번에 제가 읽은 논문은 "Mathad, Vikram C., et al. "A deep learning algorithm for objective assessment of hypernasality in children with cleft palate." IEEE Transactions on Biomedical Engineering 68.10 (2021): 2986-2996."입니다. 구개열 환자들의 과비음(Hypernasality)을 측정하기 위한 딥러닝 모델을 제시한 논문입니다.

이 논문을 읽다보니 의학적인 지식도 필요하고 발음에 대한 내용도 많이 알아야 하더라고요. 그래서 1탄과 2탄으로 나누어서 리뷰를 하려 합니다.

1탄에서는 이 논문을 읽기 위한 간단한 의학 지식에 대해서 언급하고 2탄에서 본격적으로 모델 리뷰를 해보려 합니다!!

# Cleft Palate(CP)란? 

우리말로는 구순열, 구개열, 구순구개열로 불리는 질환입니다. 구순열과 구개열은 살짝 다르지만 영어 명칭은 같나봅니다.

![image](http://www.amc.seoul.kr/healthinfo/health/attach/img/30167/20111222092331_1_30167.jpg)

구개란 비강(코 안의 공간)과 구강(입 안의 공간)의 경계입니다.

(TMI: 고등학교 문법 시간에 구개음화를 배우셨을 텐데 기억하시나요??!!
'굳이' 라는 단어를 '구디'라고 발음하지 않고 '구지'라고 발음하시죠? 이런 현상을 구개음화라고 합니다.)

간단하게 요약해서 CP는 분리되어 있어야 할 구강과 비강이 합쳐졌다고 보시면 될 것 같습니다. 이 질환을 앓고 있으면 