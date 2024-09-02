---
layout: post
title:  "Raw to Bmp and Bmp to Raw"
date:   2024-08-19T20:08:52+09:00
author: DINHO
categories:
  - C
  - 신호-처리-이론
cover:  "/assets/post/lenacolour.png"
---

오늘은 bmp 파일에 대해서 설명해드리고, raw 파일을 bmp 파일로, bmp 파일을 raw 파일로 바꾸는 과정을 c로 구현해보겠습니다.

# RAW 파일이란? 

먼저 그동안 Raw 파일에 대해 계속 다뤄왔었습니다!! Raw는 영어로 "날것의"라는 뜻이 있죠? 말 그대로 어떠한 압축이나 처리 과정이 없는 파일을 raw파일이라고 합니다. 이미지에서 raw 파일은 원본 그대로의 이미지이기 때문에 화질 저하의 문제가 없습니다. 다만 그만큼 용량이 크다는 단점이 있습니다.

따라서 별도의 헤더 파일이 없고 각각의 픽셀의 값들로만 구성된 가장 간단한 파일입니다. 픅백 이미지의 경우 각각의 픽셀이 1byte를 차지하며 좌측 상단부터 값들이 저장됩니다.

컬러 이미지의 경우 Red, Green, Blue의 순서로 각각의 색이 1byte씩 사용하여 하나의 픽셀이 3byte를 차지합니다.

# Bmp 파일이란?

Bmp 파일은 Raw 파일과 달리 헤더파일을 갖습니다. Bmp 파일은 하나의픽셀 당 사용하는 bit 수에 따라서 그 종류가 다릅니다. 대부분 24bit를 사용하므로 저도 24bit 파일을 기준으로 설명하겠습니다.

24bit Bmp 파일은 두 개의 구조체를 헤더파일로 갖습니다. 구조체에는 _BITMAPFILEHEADER_ 와 _BITMAPINFOHEADER_ 가 있습니다.

- __BITMAPFILEGEADER__ : BITMAPFILEHEADER는 파일의 크기, 형태 등의 정보를 가지고 있습니다. 자세한 정보는 아래와 같습니다.

<img src="/assets/post/bmp1.png">

- __BITMAPINFOHEADER__ : BITMAPINFOHEADER는 이미지의 가로, 세로, 비트 수 등의 정보를 가지고 있습니다. 자세한 정보는 아래와 같습니다.

<img src="/assets/post/bmp2.png">

Bmp 파일은 헤더파일 다음으로 이미지의 픽셀 값들이 저장되는데, 주의할 점은 Raw 파일과 저장 방식이 다르다는 것입니다. Raw 파일은 좌측 상단부터 우측 하단까지 r,g,b,r,g,b,... 순으로 저장되지만, Bmp 파일은 좌측 하단부터 우측 상단까지 b,g,r,b,g,r... 순서로 저장됩니다.

<img src="/assets/post/bmp3.png">

주의할 점이 한 가지 더 있습니다!! Bmp 파일은 이미지의 가로 크기가 4의 배수를 만족해야 합니다. 그 이유는 비디오 메모리가 4의 배수 체계를 갖고 있기 때문입니다. 제가 사용한 Lena_color 이미지는  256*256이기 때문에 고려하지 않았지만, 꼭 알아두어야 합니다.

그렇다면 코드를 살펴보겠습니다.

