---
layout: post
title:  "FFT with python"
date:   2024-01-31T03:08:52+09:00
author: DINHO
categories:
  - Python
  - 신호-처리-이론
cover:  "/assets/post/fft_cover.png"
---

오늘은 디지털 신호처리의 기초!! FFT에 대해서 아주 간단하게 언급하고 Python과 함께 직접 적용해보도록 하겠습니다. 이 글은 기본적인 디지털 신호 처리 과정, DFT의 내용을 알고 있다고 생각하고 글을 쓰겠습니다😄 나중에 기회가 된다면 신호 처리 생초보를 위한 디지털 신호 처리 이론도 다루어보도록 하겠습니다😋😋

# FFT 기초

FFT(Fast Fourier Transform)는 DFT(Discrete Fourier Transform)를 빠르게 수행하기 위한 알고리즘입니다. DFT는 아래의 식처럼 정의가 가능합니다.

<img src="/assets/post/DFT정의.png">

DFT 정의에 따라 계산하면 O(N 2)의 연산이 필요하지만, FFT를 이용하면 O(N log N)의 연산만으로 가능하기 때문에 굉장히 빠른 계산이 가능합니다. 아래 이미지는 빅오 표기법을 눈으로도 이해하기 쉽도록 이미지를 첨부하였습니다.

<img src="https://velog.velcdn.com/images/welloff_jj/post/5d29a3fb-c5e1-4f81-919b-7ddfd774add5/%E1%84%87%E1%85%B5%E1%86%A8%E1%84%8B%E1%85%A9.jpeg">
<figcaption> 이미지 출처: velog.io/@welloff_jj/Complexity-and-Big-O-notation </figcaption>

# FFT VS DFT

계산 시간에 대해서 코드로 직접 비교해보겠습니다. 아래는 기본 라이브러리를 이용해 DFT를 구현한 C코드의 일부입니다. 제가 전공 과제를 하기 위해 다음과 같이 DFT를 구현했었는데요. 이때 문제가 있었습니다. 바로 DFT를 구현하기 위해 4중 for문을 사용한다는 점인데요. 당시 DFT를 적용한 이미지가 512*512크기의 Lena.raw파일이였는데, dft 돌리는데 진짜 몇 시간이나 걸렸습니다. 

근데 이 부분을 FFT로 구현하면 어떻게 될까요??

```C
// Function to compute the 2D DFT
void computeDFT(unsigned char* imageData, double* outputReal, double* outputImag) {
    for (int u = 0; u < WIDTH; u++) {
        for (int v = 0; v < HEIGHT; v++) {
            double sumReal = 0.0;
            double sumImag = 0.0;

            for (int x = 0; x < WIDTH; x++) {
                for (int y = 0; y < HEIGHT; y++) {
                    int index = y * WIDTH + x;
                    double angle = 2 * PI * ((u * x / (double)WIDTH) + (v * y / (double)HEIGHT));
                    sumReal += imageData[index] * cos(angle);
                    sumImag += -imageData[index] * sin(angle);
                }
            }

            outputReal[u + v * WIDTH] = sumReal;
            outputImag[u + v * WIDTH] = sumImag;
        }
    }
}
```

아래는 FFT를 기본 라이브러리를 이용해서 적은 C코드의 일부입니다. 보시다 시피 4중 for문이 없어지고 2중 for문 많으로 계산할 수 있어서 비교할 수 없을 만큼 빠릅니다.

```C
// 2D FFT implementation
void fft2D(Complex* data, int width, int height) {
    // Apply 1D FFT row-wise
    for (int y = 0; y < height; y++) {
        fft1D(data + y * width, width);
    }

    // Apply 1D FFT column-wise
    Complex* temp = (Complex*)malloc(sizeof(Complex) * width);
    for (int x = 0; x < width; x++) {
        for (int y = 0; y < height; y++) {
            temp[y] = data[y * width + x];
        }

        fft1D(temp, height);

        for (int y = 0; y < height; y++) {
            data[y * width + x] = temp[y];
        }
    }

    free(temp);
}
```
-------------------------

이제 DFT를 안 쓰고 FFT를 쓰는 이유를 아시겠나요?? 그렇다면 본격적으로 Python을 이용해서 FFT를 직접 실습해보도록 하겠습니다. 이번 실습에서는 임의의 mp3파일로 진행하겠습니다. 1D 신호이기 때문에 2D인 이미지보다 직관적으로 와닿으실 겁니다!!

이번 실습 과정은 다음 이미지와 같습니다. 

<img src="/assets/post/과정.png">

- 먼저 시간 도메인에서 음성 신호의 Amplitude를 확인
- FFT를 적용하여 주파수 도메인에서 Amplitude를 확인
- Band-pass 필터를 적용하여 주파수 도메인에서 Amplitude를 확인
- 필터를 적용한 신호를 시간 도메인에서 Amplitude를 확인

아래는 Python Code입니다.

```Python
import numpy as np
import matplotlib.pylab as plt
import pandas as pd
from scipy.io import wavfile
from scipy.signal import spectrogram
from scipy.fft import fft, ifft
from pydub import AudioSegment

# mp3 file 로딩
audio = AudioSegment.from_file(r"C:\Users\Home\Desktop\testfft.MP3")
# mp3 파일을 모노로 변환하고 wav형식으로 변환
audio = audio.set_channels(1)
audio.export("C:/Users/Home/Desktop/testfft.wav", format="wav")

# wav 파일 로딩
fs, data = wavfile.read("C:/Users/Home/Desktop/testfft.wav")

# 데이터를 Float 타입으로 변환
data_float = data.astype(np.float64)

# 데이터의 최대 절대값 찾기
max_val = np.max(np.abs(data_float))

# 신호를 최대 절대값으로 나누어 정규화
norm_data = data_float / max_val

# 시간 축을 생성
time = np.arange(0, len(data)) / fs

# 전체 플롯을 위한 Figure 생성
fig = plt.figure(figsize=(12, 8))
```


