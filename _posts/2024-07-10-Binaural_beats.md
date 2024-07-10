---
layout: post
title:  "Binaural Beat with python"
date:   2024-07-10T15:08:52+09:00
author: DINHO
categories:
  - Python
  - 신호-처리-이론
cover:  "/assets/post/binaural_beat.png"
---

오늘은 Binaural Beat에 대해 이야기해보겠습니다. 

먼저 Beat(맥놀이)란 두 귀로 서로 다른 주파수의 소리를 들었을 때, 뇌가 이 두 주파수의 차이만큼 비트를 생성하여 인지하는 현상입니다. 예를 들어 440Hz와 480Hz 두 주파수가 있다고 생각하고 이러한 사인파를 이용해서 Beat를 생성한다고 생각해봅시다. 그렇다면 함수는 아래와 같습니다.

$$440Hz = sin(\pi 440t)$$

$$480Hz = sin(\pi 480t)$$

$$440Hz + 480Hz = sin(\pi 440t) + sin(\pi 480t)$$

이렇게 두 사인파를 합친 소리는 어떤 소리일까요?? Python을 통해 두 파형을 만들어보겠습니다. 파형은 아래와 같습니다.

<img src="/assets/post/binaural_beat_result.png">

위 파형을 만드는 코드는 아래와 같습니다.

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.io.wavfile import write

# 시간 배열 생성
t = np.linspace(0, 2, 1000)
t_wav = np.linspace(0, 2, 2 * sr, endpoint=False)

# 주파수 정의
f1 = 440  # Frequency in Hz
f2 = 480  # Frequency in Hz

# 두 개 사인파 생성
y1 = np.sin(2 * np.pi * f1 * t)
y2 = np.sin(2 * np.pi * f2 * t)

y1_wav = np.sin(2 * np.pi * f1 * t_wav)
y2_wav = np.sin(2 * np.pi * f2 * t_wav)

# 사인파 합성
beat = y1 + y2
beat_wav = y1_wav + y2_wav

# Sampling rate
sr = 44100  # in Hz

# Clipping을 피하기 위한 정규화
max_val = np.max(np.abs(beat))
beat_wav_normalized = beat_wav / max_val

# plot
plt.figure(figsize=(14, 8))

plt.subplot(3, 1, 1)
plt.plot(t, y1, label='440 Hz')
plt.legend()

plt.subplot(3, 1, 2)
plt.plot(t, y2, label='480 Hz')
plt.legend()

plt.subplot(3, 1, 3)
plt.plot(t, beat, label='Beat (440 Hz + 480 Hz)')
plt.legend()

plt.xlabel('Time (seconds)')
plt.ylabel('Amplitude')
plt.title('Waveforms and Beats')

# Display the plot
plt.tight_layout()
plt.show()

# WAV file 생성
wav_file_path = "C:\\Users\\taeso\\Desktop\\440Hz_480Hz_Beats.wav"
write(wav_file_path, sr, beat_wav_normalized.astype(np.float32))

```

아래는 wav 파일입니다. 어디서 익숙한 소리가 들리죠? 바로 전화를 걸 때 연결음입니다!! 이 연결음이 사실 맥놀이 현상을 이용해서 만든 음입니다.

<td><audio controls="" preload="none"><source src="/assets/post/440Hz_480Hz_Beats.wav"></audio></td>

다음에는 스테레오 채널에서 왼쪽과 오른쪽에 다른 주파수를 입력하여 뇌가 하나의 음으로 인지하게 하는 python 코드로 찾아뵙겠습니다😁😀