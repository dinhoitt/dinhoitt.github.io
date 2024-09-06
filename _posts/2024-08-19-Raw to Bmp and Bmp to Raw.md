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

아래는 이번 주제의 C 코드입니다. 각 줄마다 주석을 달아두었으니 설명은 주석으로 대체하겠습니다. 감사합니다!!😁

raw to bmp부터 살펴보겠습니다.

```C
#define _CRT_SECURE_NO_WARNINGS  
#include <stdio.h>              
#include <stdlib.h>              
#include <string.h>              

#pragma pack(push, 1)            // 구조체 멤버의 메모리 정렬을 1바이트 단위로 설정

// 비트맵 파일 헤더 구조체 정의
typedef struct {
    unsigned short bfType;       // 파일 형식을 나타내는 필드 ('BM'을 나타냄)
    unsigned int bfSize;         // 파일의 전체 크기
    unsigned short bfReserved1;  // 예약된 필드 (사용되지 않음)
    unsigned short bfReserved2;  // 예약된 필드 (사용되지 않음)
    unsigned int bfOffBits;      // 비트맵 데이터의 시작 위치 (헤더 이후 위치)
} BITMAPFILEHEADER;

// 비트맵 정보 헤더 구조체 정의
typedef struct {
    unsigned int biSize;         // 구조체의 크기
    int biWidth;                 // 이미지의 가로 길이 (픽셀 단위)
    int biHeight;                // 이미지의 세로 길이 (픽셀 단위)
    unsigned short biPlanes;     // 사용되는 색상 평면 수 (항상 1)
    unsigned short biBitCount;   // 픽셀당 비트 수 (24비트 이미지의 경우 24)
    unsigned int biCompression;  // 압축 방식 (0은 압축 없음)
    unsigned int biSizeImage;    // 이미지의 크기 (비 압축의 경우 0으로 설정 가능)
    int biXPelsPerMeter;         // 이미지의 가로 해상도 (미터당 픽셀 수)
    int biYPelsPerMeter;         // 이미지의 세로 해상도 (미터당 픽셀 수)
    unsigned int biClrUsed;      // 사용된 색상 수 (0이면 기본값 사용)
    unsigned int biClrImportant; // 중요한 색상 수 (0이면 모두 중요)
} BITMAPINFOHEADER;

#pragma pack(pop)                // 원래의 메모리 정렬 방식으로 복귀

// 상하반전 함수의 프로토타입 선언 (세부 구현은 이후에 있음)
void change(unsigned char* Image);

// BMP 파일을 저장하는 함수
void save_bmp(const char* filename, unsigned char* input, int width, int height) {
    BITMAPFILEHEADER bfh;        // 비트맵 파일 헤더 구조체 변수
    BITMAPINFOHEADER bih;        // 비트맵 정보 헤더 구조체 변수

    // BITMAPFILEHEADER 초기화
    bfh.bfOffBits = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER); // 픽셀 데이터의 시작 위치 계산
    bfh.bfSize = width * height * 3 + bfh.bfOffBits; // 전체 파일 크기 계산 (픽셀 데이터 크기 포함)
    bfh.bfType = 0x4D42; // 'BM'을 나타내는 2바이트 값 (비트맵 파일 식별자)
    bfh.bfReserved1 = 0; // 예약 필드 (항상 0)
    bfh.bfReserved2 = 0; // 예약 필드 (항상 0)

    // BITMAPINFOHEADER 초기화
    bih.biSize = sizeof(BITMAPINFOHEADER); // 정보 헤더의 크기 설정
    bih.biWidth = width; // 이미지의 너비 설정
    bih.biHeight = height; // 이미지의 높이 설정
    bih.biPlanes = 1; // 평면 수 설정 (항상 1)
    bih.biBitCount = 24; // 픽셀당 비트 수 (24비트 컬러 이미지)
    bih.biCompression = 0; // 압축 방식 설정 (압축 없음)
    bih.biSizeImage = 0; // 이미지 크기 (압축 없는 경우 0)
    bih.biXPelsPerMeter = 0; // 가로 해상도 (미터당 픽셀 수, 기본값 0)
    bih.biYPelsPerMeter = 0; // 세로 해상도 (미터당 픽셀 수, 기본값 0)
    bih.biClrUsed = 0; // 사용된 색상 수 (0이면 모든 색상 사용)
    bih.biClrImportant = 0; // 중요한 색상 수 (0이면 모든 색상이 중요)

    FILE* file = fopen(filename, "wb"); // BMP 파일을 쓰기 모드로 열기
    if (!file) { // 파일 열기에 실패했을 경우 에러 처리
        perror("Could not open file");
        return;
    }

    fwrite(&bfh, sizeof(bfh), 1, file); // 파일 헤더를 파일에 쓰기
    fwrite(&bih, sizeof(bih), 1, file); // 정보 헤더를 파일에 쓰기

    // 이미지 데이터를 처리하고 파일에 쓰는 루프
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            unsigned char output[3];
            output[0] = input[(i * width + j) * 3 + 2]; // BGR 순서를 RGB로 변환
            output[1] = input[(i * width + j) * 3 + 1]; 
            output[2] = input[(i * width + j) * 3 + 0]; 
            fwrite(output, sizeof(output), 1, file); // 변환된 데이터를 파일에 쓰기
        }
    }

    fclose(file); // 파일 닫기
}

int main() {
    int width = 256; // Lena 이미지의 너비 설정
    int height = 256; // Lena 이미지의 높이 설정

    // 원본 이미지 데이터를 저장할 메모리 할당
    unsigned char* input = (unsigned char*)malloc(width * height * 3); // 픽셀당 3바이트(BGR)
    if (input == NULL) { // 메모리 할당 실패 시 에러 처리
        perror("Memory allocation failed");
        return 1;
    }

    // raw 파일을 열어서 이미지 데이터를 읽어오기
    FILE* rawFile = fopen("Lena_Color.raw", "rb"); // 원본 이미지(raw 파일)를 읽기 모드로 열기
    if (!rawFile) { // 파일 열기에 실패했을 경우 에러 처리
        perror("Could not open raw file");
        free(input); // 메모리 해제
        return 1;
    }

    // raw 파일에서 이미지 데이터를 읽어서 input 배열에 저장
    fread(input, sizeof(unsigned char), width * height * 3, rawFile); // 이미지 데이터를 읽어옴
    fclose(rawFile); // 파일 닫기

    // 상하 반전 처리
    change(input);

    // 처리된 이미지를 BMP 파일로 저장
    save_bmp("Lena_Copy.bmp", input, width, height);

    // 메모리 해제
    free(input);

    return 0;
}

// 이미지의 상하 반전을 수행하는 함수
void change(unsigned char* Image)
{
    unsigned int i, j, ch;
    for (i = 0; i < 256 / 2; i++) // 이미지의 절반만 반복 (상하 반전을 위해)
        for (j = 0; j < 768; j++) // 한 줄의 픽셀 데이터 반복 (256 * 3 = 768)
        {
            ch = Image[i * 768 + j]; // 상단의 픽셀 값을 임시 저장
            Image[i * 768 + j] = Image[(256 - i - 1) * 768 + j]; // 상단의 픽셀을 하단의 픽셀로 교체
            Image[(256 - i - 1) * 768 + j] = ch; // 하단의 픽셀을 상단으로 교체
        }
}
```

아래는 bmp to raw입니다.

```C
#define _CRT_SECURE_NO_WARNINGS  

#include <stdio.h>  
#include <stdlib.h> 
#include <string.h> 

#define VER 256  // 이미지의 세로 크기
#define HOR 256  // 이미지의 가로 크기
#define F_SIZE (VER*HOR)  // 이미지의 전체 크기 (픽셀 수)
#define T_SIZE 256*256*3  // 이미지의 전체 크기 (256x256 해상도의 24비트 RGB 이미지)
 
#define BI_RGB 0  // BI_RGB 상수가 정의되지 않은 경우 0으로 정의


#pragma pack(push, 1)  // 구조체 멤버를 1바이트 단위로 정렬

// 비트맵 파일 헤더 구조체 정의
typedef struct {
    unsigned short bfType;       // 파일 형식 (BMP는 'BM'으로 시작)
    unsigned int bfSize;         // 파일의 전체 크기 (바이트 단위)
    unsigned short bfReserved1;  // 예약된 필드 (사용되지 않음)
    unsigned short bfReserved2;  // 예약된 필드 (사용되지 않음)
    unsigned int bfOffBits;      // 비트맵 데이터의 시작 위치
} BITMAPFILEHEADER;

// 비트맵 정보 헤더 구조체 정의
typedef struct {
    unsigned int biSize;         // 정보 헤더의 크기
    int biWidth;                 // 이미지의 가로 크기 (픽셀 단위)
    int biHeight;                // 이미지의 세로 크기 (픽셀 단위)
    unsigned short biPlanes;     // 사용되는 색상 평면 수 (항상 1)
    unsigned short biBitCount;   // 픽셀당 비트 수 (24비트 컬러의 경우 24)
    unsigned int biCompression;  // 압축 방식 (0은 압축 없음)
    unsigned int biSizeImage;    // 이미지 데이터 크기 (바이트 단위)
    int biXPelsPerMeter;         // 가로 해상도 (미터당 픽셀 수)
    int biYPelsPerMeter;         // 세로 해상도 (미터당 픽셀 수)
    unsigned int biClrUsed;      // 사용된 색상 수 (0이면 기본값 사용)
    unsigned int biClrImportant; // 중요한 색상 수 (0이면 모두 중요)
} BITMAPINFOHEADER;
#pragma pack(pop)  // 원래의 구조체 정렬 방식으로 복귀

unsigned char* alloc_pic(int SIZE);  // 메모리 할당을 위한 함수
void change(unsigned char* Image);   // 이미지 상하 반전 함수

int main() {
    FILE* fin, * fout;
    fin = fopen("Lena_Color.bmp", "rb");  // 입력 BMP 파일을 읽기 모드로 엶
    fout = fopen("Lena_Color_modi.raw", "wb");  // 출력 RAW 파일을 쓰기 모드로 엶

    if (!fin || !fout) {  // 파일 열기에 실패했을 경우 에러 처리
        perror("Error opening file");
        return 1;
    }

    BITMAPFILEHEADER bh;  // 비트맵 파일 헤더 변수
    BITMAPINFOHEADER bi;  // 비트맵 정보 헤더 변수

    fread(&bh, sizeof(BITMAPFILEHEADER), 1, fin);  // BMP 파일 헤더를 읽음
    fread(&bi, sizeof(BITMAPINFOHEADER), 1, fin);  // BMP 정보 헤더를 읽음

    if (bh.bfType != 0x4D42) {  // 파일 형식이 'BM'이 아닌 경우 (BMP 파일이 아님)
        printf("Not a valid BMP file\n");
        return 1;
    }

    unsigned char* input = alloc_pic(bi.biSizeImage);  // 입력 이미지를 저장할 메모리 할당
    unsigned char* output = alloc_pic(256 * 256 * 3);  // 출력 이미지를 저장할 메모리 할당

    fread((void*)input, sizeof(unsigned char), bi.biSizeImage, fin);  // 입력 파일에서 이미지 데이터를 읽음

    // BGR을 RGB로 변환하는 루프
    for (int i = 0; i < VER * HOR * 3; i += 3) {
        output[i] = input[i + 2];    // Blue -> Red
        output[i + 1] = input[i + 1];  // Green -> 그대로 유지
        output[i + 2] = input[i];    // Red -> Blue
    }

    change(output);  // 이미지 상하 반전

    fwrite((void*)output, sizeof(unsigned char), 256 * 256 * 3, fout);  // 변환된 이미지를 RAW 파일로 저장

    fclose(fin);  // 입력 파일 닫기
    fclose(fout); // 출력 파일 닫기

    return 0;
}

// 이미지 상하 반전 함수
void change(unsigned char* Image) {
    unsigned int i, j, ch;
    for (i = 0; i < VER / 2; i++) {  // 이미지의 절반만 반복 (상하 반전 위해)
        for (j = 0; j < HOR * 3; j++) {  // 각 픽셀의 RGB 값 반복
            ch = Image[i * HOR * 3 + j];  // 상단 픽셀을 임시 저장
            Image[i * HOR * 3 + j] = Image[(VER - i - 1) * HOR * 3 + j];  // 상단 픽셀을 하단 픽셀로 교체
            Image[(VER - i - 1) * HOR * 3 + j] = ch;  // 하단 픽셀을 상단으로 교체
        }
    }
}

// 메모리 할당 함수
unsigned char* alloc_pic(int SIZE) {
    unsigned char* pic;
    if ((pic = (unsigned char*)calloc(SIZE, sizeof(unsigned char))) == NULL) {  // SIZE 크기만큼 메모리 할당
        printf("\n malloc_picture : Picture structure \n");  // 메모리 할당 실패 시 에러 메시지 출력
        exit(1);  // 프로그램 종료
    }
    return pic;  // 할당된 메모리 포인터 반환
}
```