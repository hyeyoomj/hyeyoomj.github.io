---
title:  "openCV 프로그래밍 - 템플릿 매칭, 영상 분할, 컨투어"
excerpt: "알짜배기 예제로 배우는 openCV - 4"

categories:
  - openCV
tags:
  - openCV
  - c++
  - python
  - computervision
  - image_segmentation
  - template_matching
  - contour

last_modified_at: 2020-06-30T18:06:00-05:00

---

# 알짜배기 예제로 배우는 openCV - 4

## OpenCV 란?
OpenCV란 오픈소스 컴퓨터 비전 라이브러리로, 공통 API를 사용하여 컴퓨터 비전 또는 영상처리 프로그래밍을 할 수 있는 환경을 제공한다. 또한 윈도우, 맥, 리눅스, iOS, 등 다양한 운영체제에서 사용이 가능하다는 장점도 갖고 있다.
이 책에서는 openCV의 개념과 이론 보다는, 주요 openCV 함수와 알고리즘을 파이썬과 c++를 사용한 여러 예제를 통해 소개한다. 빠르게 openCV 프로그래밍에 익숙해 질 수 있고, 필요한 openCV 기능들을 찾아 사용할 수 있다는 장점이 있다.

또한, 동일한 예제에 대해 c++과 파이썬 두 언어를 모두 보여주기 때문에 언어에 구애받지 않고 프로그래밍을 할 수 있다는 장점이 있다. 무엇보다 코드 마다 **상세한 주석** 이 달려있기 때문에 코드 만을 보고도 openCV와 그 사용 방법에 대해 잘 이해할 수 있다.

시작하기 앞서, 해당 게시물은 본인이 이 책을 읽으면서 openCV의 기본 개념 정리와 새롭게 알게 된 내용을 정리하는 게시물임을 알려 드린다. 상세한 코드 설명이나 예제 모두를 다루는 것은 아니며, 핵심 코드의 일부와 사용 방법을 다루고 모든 코드는 c++을 사용한다.


## 템플릿 매칭 Template Matching

템플릿 매칭은, 주어진 템플릿 이미지와 일치하는 영역을 입력 이미지에서 찾는 방법이다. 2차원 컨볼루션 처럼, 템플릿 이미지를 입력 이미지 위에서 이동 시키면서 대응하는 픽셀들을 비교한다.<br>
openCV 상에서는 템플릿 검출을 위해 `matchTemplate` 함수를 제공하고, 아래와 같이 총 6가지 검출 방법이 있다.

<p align="center">
  <img src="/assets/images/results/matchtemplate.png" width="500px" height="400px">
</p>

`matchTemplate` 함수는 grayscale 이미지를 리턴하는데, 이 이미지의 각 픽셀은 템플릿 이미지와 어느정도 일치 하는지를 나타내게 된다.<br>
얻은 결과로부터 `minMaxLoc` 함수를 사용해 **템플릿과 일치하는 사각 영역의 왼쪽 위 코너** 위치를 리턴 받는데, 이때 사용한 방법에 따라 최댓값 혹은 최솟값의 위치가 리턴된다.<br>
(위의 표 상으로 위 두개 함수는 최솟값, 아래 네개 함수는 최댓값을 리턴한다)

그런데 이 함수는 값이 최대 혹은 최소 하나의 위치 만을 알려주기 때문에 오브젝트를 하나밖에 검출하지 못한다. 이때는 임계값 `threshold` 을 사용해 찾아줄 수 있다.

## 영상 분할 Image Segmentation
영상 분할은 입력 이미지에서 물체 인식, 추적 등을 하기 위한 전처리 과정으로서 사용된다.

### 이진화
영상의 이진화, 즉 grayscale 이미지를 바이너리 이미지로 바꾸는 것으로 배경과 오브젝트를 분리하는 데 사용할 수 있다. 원본 이미지를 grayscale 이미지로 변환한 후, threshold를 이용해 배경과 물체를 분리해 낼 수 있다.

고정된 threshold를 사용한 이진화를 위해, openCV 에서는 `threshold` 함수를 제공한다. **grayscale 이미지를 입력** 으로 받아, **이진화 이미지를 출력** 으로 준다. 이 `threshold` 값을 조정하면서 물체를 검출할 수 있다.

### HSV
위와 같이 이진화를 위해 컬러 이미지를 grayscale 이미지로 변환하는 과정에서, 이미지의 색 정보가 사라지기 때문에 이진화로는 특정 색으로 객체를 검출해 낼 수 없다. 따라서 이미지를 HSV 색공간으로 변환하여 검출을 진행한다.<br>
아래 그림과 같이, HSV의 Hue 성분은 일정한 간격을 두고 주요 색이 배치되어 있기 때문에 원하는 색의 범위를 지정하기 쉽다. 다만, 유의해야 할 것은 **openCV 에서 Hue 성분의 범위가 0~180** 이기 때문에 **표시된 값의 절반 값으로** 계산해 사용해야 한다. 예를 들어, 파란색의 범위는 120을 기준으로 +-20도 정도로 정하면 된다.

<p align="center">
  <img src="/assets/images/results/hue.png" width="300px" height="300px">
</p>

### Background Subtraction
동영상에서 배경을 제거하고, 물체를 검출할 수 있다. 배경이 되는 장소를 일정 시간 촬영 후, 새로운 객체가 등장 하면 해당 객체만 검출할 수 있는 것이다.
openCV 상에서는 해당 알고리즘 중 하나인 `BackgroundSubtractorMOG2` 함수를 제공한다.


### 라벨링

라벨링은 이진화 영상으로 바꾼 후, 각 흰색 영역을 구분하기 위해 영역마다 번호를 부여하는 것인데, 이는 한 장의 이미지에 있는 흰색 오브젝트들을 개별적으로 다룰 수 있다.
`connectedComponentsWithStats` 함수를 사용할 수 있고, 다음 그림과 같이 나타낼 수 있다.

<p align="center">
  <img src="/assets/images/results/labeling.jpeg" width="400px" height="400px">
</p>


## 컨투어 Contours
특정 영역의 경계를 따라 같은 픽셀값을 갖는 지점끼리 연결하는 선을 컨투어 라고 한다. 이는 영역에 대한 모양 분석이나, 오브젝트 검출을 위한 전처리로서 많이 사용된다.
물체를 찾아서 외곽선을 보강하는 데 사용되기도 한다. openCV 에서는 이 컨투어를 찾기 위해 `findContours` 함수를 제공한다.


이렇게 찾은 컨투어를 `drawContours` 함수를 통해 이미지에 그려낼 수 있다. 외곽선을 따라 그려지게 된다.

이 컨투어의 특징을 잘 사용하는 것이 중요한데, 이에 따른 다양한 함수들이 있다.

### 영역 크기
`contourArea` 함수는 컨투어의 영역 크기를 **픽셀의 개수** 로 리턴해준다. 이를 통해 도형 영역의 크기를 픽셀값으로서 알 수 있고, 객관적으로 계산할 수 있다.

### 근사화
컨투어를 직선으로 근사화 할 수도 있다. 도형에서 검출된 기존 컨투어를 직선으로 근사화하여 사각형으로 그릴 수 있는데, 아래는 `approxPolyDP` 함수를 사용하여 도형의 컨투어를 근사화 한 예시이다.

<p align="center">
  <img src="/assets/images/results/line.jpeg" width="400px" height="400px">
</p>

### Convex Hull
Convex Hull은 컨투어를 구성하는 모든 점을 포함하는 블록 다각형을 그리는 것이다. 손가락에 적용하면 아래 그림처럼 손가락 사이를 건너뛰고 손의 외곽 다각형을 그려 검출할 수 있다.<br>
openCV 상에서 제공하는 `convexHull` 함수를 사용한다.

<p align="center">
  <img src="/assets/images/results/hull.jpeg" width="400px" height="400px">
</p>

### Convexity Defects
Convexity Defects은 볼록 사각형 내부에 컨투어가 **오목하게 들어간 부분** 을 말한다. 손가락 이미지에 적용해보면 아래처럼 손가락 사이의 오목한 부분을 검출할 수 있는 것이다.<br>
아래는 `convexityDefects` 함수를 사용한 예시이다.

<p align="center">
  <img src="/assets/images/results/labeling.jpeg" width="400px" height="400px">
</p>
