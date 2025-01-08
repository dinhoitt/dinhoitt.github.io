---
layout: post
title:  "파이썬 입출력"
date:   2025-01-03T20:08:52+09:00
author: DINHO
categories:
  - Python
cover:  "/assets/post/python.png"
---

너무 오랜만이네요. 기말고사, 종강 후에 여러 일들이 겹치면서 블로그를 소홀히 하게 된 것 같습니다. 확실히 학기 중에 학교 생활하면서 블로그 쓰기는 쉽지 않네요. 
꾸준히 하는 것이 중요한데, 이제 졸업했으니 꾸준히 해보겠습니다😎

오늘은 간단하게 Python 기초에 대해 포스팅해보겠습니다. python 입출력 함수와 예제를 함께 이야기해보도록 하겠습니다.

Python의 input() 함수는 사용자로부터 입력을 받는 데 사용되는 기본적인 함수입니다. 입력 받은 데이터는 기본적으로 문자열(str) 형태로 반환됩니다. input() 뒤에 다양한 기능을 연결하여 데이터를 처리할 수 있습니다. 대표적인 기능과 활용 들을 아래에 정리해봤습니다.

1. __strip()__

    입력 값의 앞뒤 공백(스페이스, 탭, 줄바꿈 등)을 제거해줍니다.

    ```Python
    # 예시1

    data = input().strip()

    """
    입력 값이 " hello "인 경우 data는 "hello"가 됩니다.
    """
    ```

2. __split()__

    입력 문자열을 공백이나 특정 구분자를 기준으로 분리하여 리스트로 반환합니다.

    ```Python
    # 예시2

    data = input().split()

    """
    입력 값이 "1 2 3"인 경우 data는 ['1', '2', '3']가 됩니다. []는 리스트입니다.
    """
    ```

3. __int()/float()__

    입력값을 문자열이 아닌 정수 또는 실수로 변환하는 법입니다.

    ```Python
    # 예시 3

    a = int(input())
    b = float(input())

    """
    입력값이 "25"인 경우 a는 25(정수)
    입력값이 "3.14"인 경우 b는 3.14(실수)
    """
    ```

4. __join__

    여러 개의 입력값을 특정 구분자로 결합하여 하나의 문자열로 만듧니다.

    ```Python
    # 예시 4

    data = "-".join(input().split())

    """
    입력값이 "hello world"인 경우 data는 "hello-world"가 됩니다.
    """
    ```

5. __replace(old, new)__

    입력 문자열에서 특정 문자열을 다른 문자열로 대체합니다.

    ```Python
    # 예시 5

    data = input().replace(" ", "_")

    """
    입력값이 "hello world"인 경우 data는 "hello_world"가 됩니다.
    """
    ```

6. __upper()/lower()__

    입력값을 대문자 또는 소문자로 변환합니다.

    ```Python
    # 예시 6

    upper_data = input().upper()  # 대문자 변환
    lower_data = input().lower()  # 소문자 변환

    """
    입력값이 "Hello"인 경우, upper_data는 "HELLO", lower_data는 "hello"가 됩니다.
    """
    ```

7. __count(substring)__

    특정 문자열이 입력 값에서 몇 번 나타나는지 계산합니다.

    ```Python
    # 예시 7

    count = input().count("a")

    """
    입력값이 "banana"인 경우 count는 3이 됩니다.
    """
    ```

8. __map(함수, 반복가능데이터)__

    첫 번째 인자는 적용하고자 하는 함수, 두 번째 인자는 리스트, 튜플, 문자열, range 등 반복이 가능한 객체를 사용해서 여러 시퀀스를 처리하고, 함수에 여러 인자를 전달합니다.

    ```Python
    # 예시 8

    a, b = map(int, input().split())

    """
    입력이 "2 3" 이면 a와 b는 각각 2와 3(정수)이 됩니다.
    """

    numbers = [1, 2, 3, 4, 5]
    squared = list(map(lambda x: x**2, numbers))
    print(squared)  # [1, 4, 9, 16, 25]
    ```

그렇다면 아래 백준 예제와 풀이를 소개해드리면서 오늘 포스팅 마치겠습니다.

# 문제

(세 자리 수) × (세 자리 수)는 다음과 같은 과정을 통하여 이루어진다.


(1)과 (2)위치에 들어갈 세 자리 자연수가 주어질 때 (3), (4), (5), (6)위치에 들어갈 값을 구하는 프로그램을 작성하시오.


$$
\begin{array}{r}
\phantom{+} 472 \quad \text{...(1)} \\ 
\times\phantom{+} 385 \quad \text{...(2)} \\ 
\hline
\phantom{+} 2360 \quad \text{...(3)} \\ 
+ 3776\phantom{0} \quad \text{...(4)} \\ 
+ 1416\phantom{00} \quad \text{...(5)} \\ 
\hline
181720 \quad \text{...(6)} \\ 
\end{array}
$$

# 입력

첫째 줄에 (1)의 위치에 들어갈 세 자리 자연수가, 둘째 줄에 (2)의 위치에 들어갈 세자리 자연수가 주어진다.

# 출력

첫째 줄부터 넷째 줄까지 차례대로 (3), (4), (5), (6)에 들어갈 값을 출력한다.

# 예제 입력 1 

```
472
385
```

# 예제 출력 1 

```
2360
3776
1416
181720
```

```Python
# 풀이
a = int(input())
b = input()

re2 = a * int(b[2])
re1 = a * int(b[1])
re0 = a * int(b[0])

res = a * int(b)

print(re2)
print(re1)
print(re0)
print(res)
```