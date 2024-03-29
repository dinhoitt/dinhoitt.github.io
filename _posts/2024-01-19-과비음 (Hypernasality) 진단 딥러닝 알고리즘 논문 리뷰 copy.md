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

1탄에서는 이 논문을 읽기 위한 간단한 의학 지식과 언어 지식에 대해서 언급하고 2탄에서 본격적으로 모델 리뷰를 해보려 합니다!!(아마 2탄에서 안 끝날 수 있을 것 같습니다...)

# Cleft Palate(CP)란? 

우리말로는 구순열, 구개열, 구순구개열로 불리는 질환입니다. 구순열과 구개열은 살짝 다르지만 영어 명칭은 같나봅니다.

![구순구개열 이미지-출처:아산병원](http://www.amc.seoul.kr/healthinfo/health/attach/img/30167/20111222092331_1_30167.jpg)
<figcaption> 이미지-출처:아산병원 </figcaption>

구개란 비강(코 안의 공간)과 구강(입 안의 공간)의 경계입니다.

(TMI: 고등학교 문법 시간에 구개음화를 배우셨을 텐데 기억하시나요??!!
'굳이' 라는 단어를 '구디'라고 발음하지 않고 '구지'라고 발음하시죠? 이런 현상을 구개음화라고 합니다.)

간단하게 요약해서 CP는 분리되어 있어야 할 구강과 비강이 합쳐졌다고 보시면 될 것 같습니다. 이 질환을 앓고 있으면 과비음(Hypernasality), 즉, 과하게 콧소리가 나서 발음이 이상해지고 소통에 어려울 수 있습니다.

외부에서 입술이 갈라진 상태면 CP를 알기 쉽지만 내부에서 CP를 앓고 있으면 겉으로는 알 수 없습니다. 따라서 과비음의 정도를 통해 구개열을 평가할 수 있다!! 라고 생각하시면 될 것 같습니다.

# NC, OC, NV, OV?

모든 언어에는 자음(Consonant)과 모음(Vowel)이 있습니다. 그렇다면 논문에 나오는 NC, OC, NV, OV 이것들은 뭘까요??? 간단하게 요약해보았습니다!!

- 비강 자음(Nasal Consonants, NC): 자음을 발음할 때 코로 소리를 내야 하면 비강 자음이라고 합니다.
- 구강 자음(Oral Consonants, OC): 자음을 발음할 때 입으로 소리를 내면 구강 자음이라고 합니다.
- 비강 모음(Nasal Vowels, NV): 모음을 발음할 때 코로 소리를 내야 하면 비강 모음이라고 합니다.
- 구강 모음(Oral Vowels, OV): 모음을 발음할 때 입으로 소리를 내면 구강 모음이라고 합니다.

![조음 기관-출처:ratsgo.github.io](https://i.imgur.com/oKrqW5Y.jpg "이미지 출처:ratsgo.github.io")
<figcaption> 이미지 출처: ratsgo.github.io </figcaption>

이해가 되셨을까요?? 더 쉽게 이야기 해드리자면 **비강 발음은 콧소리가 나기 때문에 코를 막고 발음하기 어렵습니다.**. 요즘 유명한 인플루언서 '냥뇽녕냥'님을 코를 막고 불러 보세요. 굉장히 어렵습니다ㅋㅋㅋㅋㅋ 그에 비해 **구강 발음은 코를 막고도 발음하기 쉽습니다.** '축구를 하고 풋살해서 졸려'라는 문장을 코를 막고 말해보세요. 굉장히 쉽게 발음할 수 있습니다.

'축구를 하고 풋살해서 졸려'처럼 비강 발음이 없는 문장을 말할 때 과하게 콧소리가 나면, 즉, 과비음(Hypernasality)이 있다면??!! 구순구개열, 더 나아가 조음 장애를 겪고 있다라고 판단할 수 있는 것이죠!! (조음이란 소리를 낼 때의 구조를 뜻합니다.)

![NASOMETER-출처:somnotec](https://www.somnotec.net/wp-content/uploads/2014/01/jaredwithheadsetcloseup1.jpg "이미지 출처:somnotec")
<figcaption> 이미지 출처: ratsgo.github.io </figcaption>


사진과 같이 과비음(Hypernasality)을 측정하여 언어적인 문제를 진단하는 방법 중 가장 유명한 방법으로는 Nasometer라는 것이 있습니다. 하지만 이 방법은 돈과 시간이 상당히 많이 듭니다. 

딥 러닝 모델을 이용하여 돈과 시간의 소비를 줄이고자 하는 것이 제가 연구하고 있는 주제입니다!! 그렇다면 2탄에서 본격적으로 데이터 설정, 전처리 과정, 인공지능 모델 등을 살펴보겠습니다. 부족한 글 읽어 주셔서 감사합니다😁

![image](https://news.kw.ac.kr/mascot/img/uni_02.png)