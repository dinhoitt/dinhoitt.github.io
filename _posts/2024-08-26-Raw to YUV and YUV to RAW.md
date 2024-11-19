---
layout: post
title:  "Raw to YUV and YUV to Raw"
date:   2024-08-26T20:08:52+09:00
author: DINHO
categories:
  - C
  - 신호-처리-이론
cover:  "/assets/post/lenacolour.png"
---

오늘은 YUV 파일에 대해 이야기 해보고, RAW TO YUV, YUV TO RAW 코드를 살펴보겠습니다.

YUV 컬러 공간은 인간의 눈의 특성을 고려하여 설계되었습니다.

- 인간의 눈은 밝기(Y)에는 민감하지만, 색차 신호(Cb, Cr)에는 상대적으로 둔감합니다.
- 이러한 특성을 활용하여 RGB 모델을 선형 변환하여 YUV 모델로 표현할 수 있습니다.

이번 글에서는 YUV 컬러 공간의 변환 공식과 저장 형식(YUV 422 및 420)에 대해 설명해보겠습니다.

## 변환 공식

RGB 모델에서 YUV 모델로의 선형 변환 공식은 다음과 같습니다.

### RGB → YUV

$$
Y = 0.257R' + 0.504G' + 0.098B' + 16
$$

$$
Cb = -0.148R' - 0.291G' + 0.439B' + 128
$$

$$
Cr = 0.439R' - 0.368G' - 0.071B' + 128
$$

### YUV → RGB

$$
R' = 1.164(Y - 16) + 1.596(Cr - 128)
$$

$$
G' = 1.164(Y - 16) - 0.813(Cr - 128) - 0.392(Cb - 128)
$$

$$
B' = 1.164(Y - 16) + 2.017(Cb - 128)
$$

- **Y (밝기, Luminance)**: 밝기를 나타내는 신호입니다. YUV 컬러 영상에서 Y 성분만 추출하면 흑백 영상을 얻을 수 있습니다.

- **Cb 및 Cr (색차, Chrominance)**: 각각 파랑(Cb) 및 빨강(Cr)의 색차 신호를 나타냅니다.

## YUV 저장 형식: 422 및 420 ##

### YUV 422

- Y, Cb, Cr이 4:2:2 비율로 저장됩니다.

- Cb와 Cr 값은 가로로 두 픽셀당 하나씩 공유됩니다.

### YUV 420

- Y, Cb, Cr이 4:1:1 비율로 저장됩니다.

- Cb와 Cr 값은 2x2 블록의 픽셀에서 공유됩니다 (가로로 네 픽셀이 아님에 주의!).

### 저장 순서

- 데이터는 **Y 성분 → Cb 성분 → Cr 성분** 순으로 저장됩니다.

- RAW나 BMP처럼 픽셀별로 저장하지 않고, Y 성분을 모두 저장한 후, Cb와 Cr 성분을 차례로 저장합니다.

## YUV 압축의 장점

Cb와 Cr 픽셀 수를 줄이면서도 사람이 느끼는 품질을 유지할 수 있기 때문에 YUV 포맷은 파일 크기를 크게 줄일 수 있습니다. 
뷰어 프로그램의 차이로 인해 약간의 차이가 보일 수 있으나, 인간의 시각적 특성 때문에 결과 영상의 품질은 동일하게 느껴집니다.

YUV 컬러 공간과 YUV 422, 420과 같은 저장 형식은 비디오 압축 및 전송에 널리 사용됩니다. YUV 변환 및 저장 원리를 이해하면, 품질 저하 없이 파일 크기를 효과적으로 줄이는 방법을 더 잘 이해할 수 있습니다. 아래 결과를 보면 압축이 되는 것을 볼 수 있습니다.

<img src="/assets/post/yuvtoraw_result.png">

아래 C 코드를 공유하면서 오늘 포스팅 마치겠습니다.

## RAW TO YUV

```C
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define VER 256     // 이미지 수직 사이즈 상수 VER 정의
#define HOR 256     // 이미지 수평 사이즈 상수 HER 정의

void raw_to_YUV422(unsigned char input[][HOR * 3], unsigned char output[][HOR / 2], FILE* fout); // RGB 이미지를 YUV422형식으로 변환하는 함수
void raw_to_YUV420(unsigned char input[][HOR * 3], unsigned char output[][HOR / 2], FILE* fout); // RGB 이미지를 YUV420형식으로 변환하는 함수

int main() {
    unsigned char(*input)[HOR * 3] = (unsigned char(*)[HOR * 3])malloc(VER * HOR * 3 * sizeof(unsigned char)); // RGB 이미지 데이터를 저장하기 위한 메모리 할당
    unsigned char(*outputUV)[HOR / 2] = (unsigned char(*)[HOR / 2])malloc(VER * HOR / 2 * sizeof(unsigned char)); // YUV 변환을 위한 U, V 컴포넌트를 저장할 메모리 할당

    if (!input || !outputUV) { // 메모리 할당이 제대로 되었는지 확인
        perror("Memory allocation failed\n");
        return 1;
    }

    FILE* fin, * fout422, * fout420; // 파일 포인터 선언
    fin = fopen("Lena_Color.raw", "rb"); // 입력 RGB 파일 열기
    if (!fin) { // 파일 열기에 실패했는지 확인
        perror("Unable to open input file\n");
        return 1;
    }
    fread(input[0], sizeof(unsigned char), VER * HOR * 3, fin); // 입력 파일로부터 RGB 데이터 읽기
    fclose(fin);

    fout422 = fopen("Lena_Color422.yuv", "wb"); // YUV 4:2:2 출력 파일 열기
    if (!fout422) { // 파일 열기에 실패했는지 확인
        perror("Unable to open output file for YUV 4:2:2\n");
        return 1;
    }
    raw_to_YUV422(input, outputUV, fout422); // RGB를 YUV 4:2:2로 변환하여 파일에 쓰기
    fclose(fout422);

    fout420 = fopen("Lena_Color420.yuv", "wb"); // YUV 4:2:0 출력 파일 열기
    if (!fout420) {
        perror("Unable to open output file for YUV 4:2:0\n");
        return 1;
    }
    raw_to_YUV420(input, outputUV, fout420); // RGB를 YUV 4:2:0로 변환하여 파일에 쓰기
    fclose(fout420);

    // 할당된 메모리 해제
    free(input);
    free(outputUV);

    return 0;
}

void raw_to_YUV422(unsigned char input[][HOR * 3], unsigned char outputUV[][HOR / 2], FILE* fout) { // RGB를 YUV 4:2:2 형식으로 변환하는 함수
    unsigned char Y, U, V; // Y, U, V 컴포넌트를 저장할 변수 선언

    // Y 값 쓰기
    for (int j = 0; j < VER; j++) {
        for (int i = 0; i < HOR * 3; i += 3) {
            // RGB를 Y 컴포넌트로 변환하고 파일에 쓰기
            Y = (unsigned char)(0.257 * input[j][i] + 0.504 * input[j][i + 1] + 0.098 * input[j][i + 2] + 16);
            
            fwrite(&Y, sizeof(unsigned char), 1, fout);

        }
        
    }

    // U와 V 값 쓰기 (수평으로 2:1로 서브샘플링)
    for (int j = 0; j < VER; j++) {
        for (int i = 0; i < HOR * 3; i += 6) {
            // RGB를 U 컴포넌트로 변환하고 파일에 쓰기
            U = (unsigned char)(-0.148 * (input[j][i] + input[j][i + 3]) / 2 - 0.291 * (input[j][i + 1] + input[j][i + 4]) / 2 + 0.439 * (input[j][i + 2] + input[j][i + 5]) / 2 + 128);
            
            // RGB를 V 컴포넌트로 변환하고 나중에 쓰기 위해 저장
            V = (unsigned char)(0.439 * (input[j][i] + input[j][i + 3]) / 2 - 0.368 * (input[j][i + 1] + input[j][i + 4]) / 2 - 0.071 * (input[j][i + 2] + input[j][i + 5]) / 2 + 128);
            
            fwrite(&U, sizeof(unsigned char), 1, fout);
            outputUV[j][i / 6] = V;
            
        }
    }
    // V 값 쓰기
    for (int j = 0; j < VER; j++) {
        fwrite(outputUV[j], sizeof(unsigned char), HOR / 2, fout);
    }
    
}

void raw_to_YUV420(unsigned char input[][HOR * 3], unsigned char outputUV[][HOR / 2], FILE* fout) { // RGB를 YUV 4:2:0 형식으로 변환하는 함수
    unsigned char Y, U, V; // Y, U, V 컴포넌트를 저장할 변수 선언
    // Y 값 쓰기
    for (int j = 0; j < VER; j++) {
        for (int i = 0; i < HOR * 3; i += 3) {
            Y = (unsigned char)(0.257 * input[j][i] + 0.504 * input[j][i + 1] + 0.098 * input[j][i + 2] + 16);
            
            fwrite(&Y, sizeof(unsigned char), 1, fout);
            
        }
        
    }

    // U와 V 값 쓰기 (수평 및 수직으로 2:1로 서브샘플링)
    for (int j = 0; j < VER; j += 2) {
        for (int i = 0; i < HOR * 3; i += 6) {
            // RGB를 U 컴포넌트로 변환하고 파일에 쓰기
            U = (unsigned char)(-0.148 * (input[j][i] + input[j][i + 3] + input[j + 1][i] + input[j + 1][i + 3]) / 4 - 0.291 * (input[j][i + 1] + input[j][i + 4] + input[j + 1][i + 1] + input[j + 1][i + 4]) / 4 + 0.439 * (input[j][i + 2] + input[j][i + 5] + input[j + 1][i + 2] + input[j + 1][i + 5]) / 4 + 128);
            
            // RGB를 V 컴포넌트로 변환하고 나중에 쓰기 위해 저장
            V = (unsigned char)(0.439 * (input[j][i] + input[j][i + 3] + input[j + 1][i] + input[j + 1][i + 3]) / 4 - 0.368 * (input[j][i + 1] + input[j][i + 4] + input[j + 1][i + 1] + input[j + 1][i + 4]) / 4 - 0.071 * (input[j][i + 2] + input[j][i + 5] + input[j + 1][i + 2] + input[j + 1][i + 5]) / 4 + 128);
            
            fwrite(&U, sizeof(unsigned char), 1, fout);
            outputUV[j / 2][i / 6] = V; // Save V to write after U
        }
    }
    // V 값 쓰기
    for (int j = 0; j < VER / 2; j++) {
        
        fwrite(outputUV[j], sizeof(unsigned char), HOR / 2, fout);
    }
    
}

```

---------------------

## YUV TO RAW

```C
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

#define VER 256     // 이미지 수직 사이즈 상수 VER 정의
#define HOR 256     // 이미지 수평 사이즈 상수 HOR 정의

void YUV_to_RGB(int Y, int U, int V, unsigned char* R, unsigned char* G, unsigned char* B);
void YUV422_to_raw(unsigned char inputY[][HOR], unsigned char inputU[][HOR / 2], unsigned char inputV[][HOR / 2], unsigned char output[][HOR * 3]);
void YUV420_to_raw(unsigned char inputY[][HOR], unsigned char inputU[][HOR / 2], unsigned char inputV[][HOR / 2], unsigned char output[][HOR * 3]);

int main() {
    unsigned char(*inputY)[HOR] = (unsigned char(*)[HOR])malloc(VER * HOR * sizeof(unsigned char)); // Y배열에 대한 메모리 할당
    unsigned char(*inputU)[HOR / 2] = (unsigned char(*)[HOR / 2])malloc(VER / 2 * HOR / 2 * sizeof(unsigned char)); // 420 U 배열에 대한 메모리 할당
    unsigned char(*inputV)[HOR / 2] = (unsigned char(*)[HOR / 2])malloc(VER / 2 * HOR / 2 * sizeof(unsigned char)); // 420 V 배열에 대한 메모리 할당
    unsigned char(*inputU2)[HOR / 2] = (unsigned char(*)[HOR / 2])malloc(VER * HOR / 2 * sizeof(unsigned char)); // 422 U 배열에 대한 메모리 할당
    unsigned char(*inputV2)[HOR / 2] = (unsigned char(*)[HOR / 2])malloc(VER * HOR / 2 * sizeof(unsigned char)); // 422 V 배열에 대한 메모리 할당
    unsigned char(*output)[HOR * 3] = (unsigned char(*)[HOR * 3])malloc(VER * HOR * 3 * sizeof(unsigned char)); // 출력 배열에 대한 메모리 할당

    if (!inputY || !inputU || !inputV || !output) {
        perror("Memory allocation failed\n");
        return 1;
    }

    // YUV422 파일 읽기
    FILE* fin422 = fopen("YUV422_Lena_Color.yuv", "rb"); //YUV422_Lena_Color.yuv 입력 파일 포인터 선언, 읽기 모드
    if (!fin422) {
        perror("Unable to open input YUV422 file\n");
        return 1;
    }
    fread(inputY[0], sizeof(unsigned char), VER * HOR, fin422); // 422 Y값 읽기
    fread(inputU2[0], sizeof(unsigned char), VER * HOR / 2, fin422); // 422 U값 읽기
    fread(inputV2[0], sizeof(unsigned char), VER * HOR / 2, fin422); // 422 V값 읽기
    fclose(fin422);

    FILE* fout = fopen("Lena_Color_From_YUV422.raw", "wb"); // YUV422_Lena_Color.yuv 출력 파일 포인터 선언, 쓰기 모드
    if (!fout) {
        perror("Unable to open output raw file from YUV422\n");
        return 1;
    }
    YUV422_to_raw(inputY, inputU2, inputV2, output); // YUV422 TO RAW 함수
    fwrite(output[0], sizeof(unsigned char), VER * HOR * 3, fout); // 파일 쓰기
    fclose(fout);

    // YUV420 파일 읽기
    FILE* fin420 = fopen("YUV420_Lena_Color.yuv", "rb"); //YUV420_Lena_Color.yuv 입력 파일 포인터 선언, 읽기 모드
    if (!fin420) {
        perror("Unable to open input YUV420 file\n");
        return 1;
    }
    fread(inputY[0], sizeof(unsigned char), VER * HOR, fin420); // 420 Y값 읽기
    fread(inputU[0], sizeof(unsigned char), VER / 2 * HOR / 2, fin420); // 420 U값 읽기
    fread(inputV[0], sizeof(unsigned char), VER / 2 * HOR / 2, fin420); // 420 V값 읽기
    fclose(fin420);

    fout = fopen("Lena_Color_From_YUV420.raw", "wb"); // YUV420_Lena_Color.yuv 출력 파일 포인터 선언, 쓰기 모드
    if (!fout) {
        perror("Unable to open output raw file from YUV420\n");
        return 1;
    }
    YUV420_to_raw(inputY, inputU, inputV, output); //YUV TO RAW 변환 함수
    fwrite(output[0], sizeof(unsigned char), VER * HOR * 3, fout); // 파일 쓰기
    fclose(fout);

    free(inputY);
    free(inputU);
    free(inputV);
    free(output);

    return 0;
}


void YUV_to_RGB(int Y, int U, int V, unsigned char* R, unsigned char* G, unsigned char* B) { // YUV to RGB 공식
    int R_temp = (int)(1.164 * (Y - 16) + 1.596 * (V - 128));
    int G_temp = (int)(1.164 * (Y - 16) - 0.813 * (V - 128) - 0.392 * (U - 128));
    int B_temp = (int)(1.164 * (Y - 16) + 2.017 * (U - 128));

    // 클램핑 적용
    *R = (unsigned char)(R_temp < 0 ? 0 : R_temp > 255 ? 255 : R_temp);
    *G = (unsigned char)(G_temp < 0 ? 0 : G_temp > 255 ? 255 : G_temp);
    *B = (unsigned char)(B_temp < 0 ? 0 : B_temp > 255 ? 255 : B_temp);
}

void YUV422_to_raw(unsigned char inputY[][HOR], unsigned char inputU[][HOR / 2], unsigned char inputV[][HOR / 2], unsigned char output[][HOR * 3]) {
    for (int j = 0; j < VER; j++) {
        for (int i = 0; i < HOR; i += 2) {
            
            int Y1 = inputY[j][i];
            int Y2 = inputY[j][i + 1];
            int U = inputU[j][i / 2];
            int V = inputV[j][i / 2];

            unsigned char R, G, B;
            YUV_to_RGB(Y1, U, V, &R, &G, &B);
            output[j][i * 3] = R;
            output[j][i * 3 + 1] = G;
            output[j][i * 3 + 2] = B;

            YUV_to_RGB(Y2, U, V, &R, &G, &B);
            output[j][(i + 1) * 3] = R;
            output[j][(i + 1) * 3 + 1] = G;
            output[j][(i + 1) * 3 + 2] = B;
        }
    }
}

void YUV420_to_raw(unsigned char inputY[][HOR], unsigned char inputU[][HOR / 2], unsigned char inputV[][HOR / 2], unsigned char output[][HOR * 3]) {
    for (int j = 0; j < VER; j += 2) {
        for (int i = 0; i < HOR; i += 2) {
            int U = inputU[j / 2][i / 2];
            int V = inputV[j / 2][i / 2];

            for (int y = 0; y < 2; ++y) {
                for (int x = 0; x < 2; ++x) {
                    int Y = inputY[j + y][i + x];
                    unsigned char R, G, B;
                    YUV_to_RGB(Y, U, V, &R, &G, &B);
                    output[j + y][(i + x) * 3] = R;
                    output[j + y][(i + x) * 3 + 1] = G;
                    output[j + y][(i + x) * 3 + 2] = B;
                }
            }
        }
    }
}

```