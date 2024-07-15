---
layout: post
title:  "Stereo Channel with python"
date:   2024-07-15T11:08:52+09:00
author: DINHO
categories:
  - Python
  - 신호-처리-이론
cover:  "/assets/post/binaural_beat.png"
---

스테레오 채널에서 바이노럴 비트(Binaural Beats)를 삽입하고, 왼쪽과 오른쪽에 다른 주파수를 출력하여 뇌가 하나의 음으로 인지하게 하는 python 코드를 소개해드리겠습니다.

먼저 스테레오는 동시에 하나 이상의 스피커에 맞춰 데이터를 사용하는 2채널 재생 및 녹음을 뜻합니다. 최근에는 영화관이나 텔레비전의 서라운드 사운드 5.1~6.1 채널 시스템과 같이 여러 개의 채널을 포함할 수도 있습니다.

그리고 스테레오 채널의 음원을 만드는 경우 보통 mp3 확장자를 이용합니다. wav파일은 어떠한 압축 과정도 거치지 않은 음원이라 용량이 큽니다. 그렇기 때문에 음악 작곡이나 편집에 씁니다. 하지만 스테레오타입과 같이 여러 채널에서 출력할 경우 용량이 커지기 때문에 최종적으로는 mp3파일로 압축합니다.

그럼 이제 본격적으로 스테레오 2채널을 이용하여 바이노럴 비트를 합성하는 python 코드를 소개하겠습니다. 알고리즘은 다음과 같습니다.

1. 오디오 파일 읽기

2. 기존 오디오의 평균 볼륨 측정

3. 사인파 생성

4. 볼륨 조절

5. 스테레오 사인파 생성

6. 기존 오디오와 사인파 합성

7. 결과 파일 저장

파이썬 코드는 아래와 같습니다.

```python
import numpy as np
import matplotlib.pylab as plt
from scipy.io import wavfile
from scipy.signal import spectrogram
from scipy.fft import fft, ifft
from pydub import AudioSegment
from pydub.generators import Sine

audio = AudioSegment.from_file("C:\\Users\\taeso\\OneDrive\\바탕 화면\\testbibeat.wav")


# 기존 오디오의 평균 볼륨 측정
average_dbfs = audio.dBFS

# 사인파 생성 설정
frequency_right = 428  # 오른쪽 채널 주파수
frequency_left = 438   # 왼쪽 채널 주파수
duration_ms = len(audio)  # 기존 오디오와 동일한 길이

# 사인파 생성 및 볼륨 조절
# 기존 오디오 볼륨보다 5dB 낮게 설정
sine_right = Sine(frequency_right).to_audio_segment(duration=duration_ms).apply_gain(average_dbfs - 5)
sine_left = Sine(frequency_left).to_audio_segment(duration=duration_ms).apply_gain(average_dbfs - 5)


# 스테레오 사인파 생성
stereo_sine = AudioSegment.from_mono_audiosegments(sine_left, sine_right)

# 기존 오디오와 사인파 합성
combined_audio = audio.overlay(stereo_sine)

# 결과 파일로 저장
combined_audio.export("C:\\Users\\taeso\\OneDrive\\바탕 화면\\testcomplete.mp3", format="mp3")
```

요약하자면 원본 오디오 파일을 2채널로 분리하고, 왼쪽 오른쪽 각 채널에 주파수가 다른 사인파를 합성한 뒤 mp3파일로 최종 오디오를 출력합니다. 

이 오디오 같은 경우 들었을 때 왼쪽만 듣거나 오른쪽만 들으면 삐- 소리만 나지만 두 채널을 동시에 듣는 경우 맥놀이 현상을 느낄 수 있습니다.

이러한 소리는 이명과 같이 소리 치료에 사용되는 기법이고 최근 많이 연구되고 있습니다. 출력 음원도 함께 공유하면서 오늘 포스팅 마무리하겠습니다.😀😀

<td><audio controls="" preload="none"><source src="/assets/post/testcomplete.mp3"></audio></td>