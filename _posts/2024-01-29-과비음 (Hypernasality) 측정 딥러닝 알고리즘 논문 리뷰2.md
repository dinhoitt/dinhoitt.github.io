---
layout: post
title:  "과비음 (Hypernasality) 측정 딥러닝 알고리즘 논문 리뷰2"
date:   2024-01-29T03:15:52+09:00
author: DINHO
categories: "논문-리뷰"
sitemap :
  changefreq : weekly
  priority : 1.0
cover:  "/assets/post_논문리뷰1.png"
---

지난번에 이어서 "Mathad, Vikram C., et al. "A deep learning algorithm for objective assessment of hypernasality in children with cleft palate." IEEE Transactions on Biomedical Engineering 68.10 (2021): 2986-2996." 논문 리뷰를 이어서 하겠습니다. 

지난번 [논문 리뷰1](https://dinhoitt.github.io/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0/2024/01/17/%EA%B3%BC%EB%B9%84%EC%9D%8C-(Hypernasality)-%EC%A7%84%EB%8B%A8-%EB%94%A5%EB%9F%AC%EB%8B%9D-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-copy.html)에서는 구개열(Cleft Palate)과 구강, 비강 조음들을 알아보았는데요. 이번에는 본격적으로 데이터셋이나 어떤 구조의 DNN 모델인지 알아보도록 하겠습니다😀😀 아울러 이번 글에서 MFCC라는 전처리 특징이 언급되는데요. 이부분에 대한 설명은 제 블로그 [신호 처리 이론]()카테고리에서 다룬 내용들이 있으니 모르시는 분들은 한 번씩 보시는 걸 추천합니다👍👍

# DATABASES

1. Healthy Speech Corpus

 먼저 이 논문에서는 Librispeech 데이터베이스를 이용해서 100 시간 동안 건강한 사람들의 발음을 DNN을 이용해서 학습시켰습니다. 데이터베이스에는 성인 251명(남자 125명, 여자 126명)이 녹음한 영어 음석 샘플이 포함되어 있습니다.

2. Americleft Database
 
 
