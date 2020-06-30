---
title:  "openCV 프로그래밍 - 블러링, 모폴로지, 허프 변환"
excerpt: "알짜배기 예제로 배우는 openCV - 3"

categories:
  - openCV
tags:
  - openCV
  - c++
  - python
  - computervision
  - blurring
  - morphology
  - houghlines

last_modified_at: 2020-06-30T15:06:00-05:00

---

# 알짜배기 예제로 배우는 openCV - 3

## OpenCV 란?
OpenCV란 오픈소스 컴퓨터 비전 라이브러리로, 공통 API를 사용하여 컴퓨터 비전 또는 영상처리 프로그래밍을 할 수 있는 환경을 제공한다. 또한 윈도우, 맥, 리눅스, iOS, 등 다양한 운영체제에서 사용이 가능하다는 장점도 갖고 있다.
이 책에서는 openCV의 개념과 이론 보다는, 주요 openCV 함수와 알고리즘을 파이썬과 c++를 사용한 여러 예제를 통해 소개한다. 빠르게 openCV 프로그래밍에 익숙해 질 수 있고, 필요한 openCV 기능들을 찾아 사용할 수 있다는 장점이 있다.

또한, 동일한 예제에 대해 c++과 파이썬 두 언어를 모두 보여주기 때문에 언어에 구애받지 않고 프로그래밍을 할 수 있다는 장점이 있다. 무엇보다 코드 마다 **상세한 주석** 이 달려있기 때문에 코드 만을 보고도 openCV와 그 사용 방법에 대해 잘 이해할 수 있다.

시작하기 앞서, 해당 게시물은 본인이 이 책을 읽으면서 openCV의 기본 개념 정리와 새롭게 알게 된 내용을 정리하는 게시물임을 알려 드린다. 상세한 코드 설명이나 예제 모두를 다루는 것은 아니며, 핵심 코드의 일부와 사용 방법을 다루고 모든 코드는 c++을 사용한다.


## 블러링 Blurring

보통 이미지에서 노이즈를 제거하기 위해 사용되는 블러링은 **컨볼루션Convolution** 을 통해 이루어진다.

컨볼루션은 해당 게시물에서 자세히 다루진 않겠으나, 이미지에 적당한 크기의 마스크 mask 를 씌워서, 마스크 범위 내에 포함되는 이웃 픽셀을 마스크의 원소와 곱하여 결과 영상의 현재 위치값을 결정하는 것을 말한다. 그 마스크에 정의된 값에 따라 이미지를 흐리거나 선명하게 만들 수 있다.

openCV 에서는 이 컨볼루션을 쉽게 할 수 있도록 `filter2D` 함수를 제공한다.
마스크, 즉 커널은 다음과 같이 정의한다.

~~~c++
Mat kernel = Mat(5, 5, CV_32F, Scalar(1/25.0)) ;
filter2D(img, dst, -1, kernel) ;
~~~

이때 커널은 이미지의 현재 위치에서, `5 * 5` 범위의 주변 픽셀의 평균값을 구해서 결과 이미지의 픽셀 값으로 입력하게 하는 커널이다.

그런데 이렇게 커널을 만들어서 하는 컨볼루션은 번거롭기도 하고, 커널의 사이즈를 직접 설정해줘야 하는 어려움이 있다. 따라서 openCV 에서는 평균 블러링 함수인 `blur` 를 제공한다. 아래와 같이 사용하면 된다.

~~~c++
blur(img, dst, Size(5,5)) ;
~~~

### 가우시안 블러링 Gaussian Blurring
모든 픽셀에 똑같은 가중치를 부여했던 평균 블러링과 달리, 가우시안 블러링은 마스크 중심에 있는 원소에 **더 높은 가중치** 를 부여한다.

그렇기 때문에 가우시안 블러링은 에지가 선명하게 보이는 상태로 블러 처리가 되고, 따라서 보통 에지 검출 전 노이즈를 제거하기 위해 사용된다.

~~~C++
GaussianBlur(img, dst, Size(5,5), 0) ;
~~~


## 에지 검출 Edge Detection
보통 이미지에서 에지 Edge 는, 픽셀 값이 급격하게 변하는 지점이다. 따라서 곡선 값을 미분해 보면, 픽셀 값이 급격하게 증가한 부분에서 미분값이 크게 나오고, 이를 통해 **주변보다 1차 미분값이 큰 부분을 에지로 검출** 할 수 있다.

1차 미분의 근사값 계산을 위해 **소벨 Sobel** 이란 것을 사용하는데, 이는 X방향 에지와 Y방향 에지 검출을 위한 마스크이다.<br>
소벨 마스크 Gx는 수직 성분 에지를 검출, Gy는 수평 성분 에지를 검출하기 때문에 두 마스크를 결합하여 결과를 도출한다.

이때, 에지 검출을 위해서는 이미지가 **grayscale** 이어야 함을 유의한다 !
~~~C++
Mat img, sobel_x ;
Sobel(img, sobel_x, CV_64F, 1, 0, 3) ;
convertScaleAbs(sobel_x, sobel_x) ;
~~~

이와 같이 `y`에 대한 소벨 마스크도 구한 뒤, 이를 `addWeighted` 함수를 사용해 더해 결과 이미지를 도출한다.<br>(책에서는 `addWeighted` 를 사용했으나, 가중치 없이 그냥 `add` 를 사용해도 무방)

~~~C++
addWeighted(sobel_x, 1, sobel_y, 1, 0, dst) ;
~~~

### 캐니 에지 디텍터 Canny Edge Detector
존 케니에 의해 개발된 케니 에지 디텍터는, 에지 검출 시 실제 에지와 흡사하고, 정확한 위치의 에지를 검출하는 데 쓰인다. 두 개의 `threshold` 값을 사용하여 한 픽셀로 구성된 에지를 검출하는 것이다. 정확하고 사용이 쉽기 때문에 에지 검출에 많이 쓰인다.

앞서 이미지 ROI 영역 검출 부분에서 예제로 쓰였기 때문에 예제는 생략하겠다.

다만 캐니 에지를 적용하기 전에, **이미지를 grayscale로 변환** 해야 하고, **블러링으로 노이즈를 제거** 후 사용해야 한다는 것을 유념해야 한다.

또한, **첫번째 임계값 threshold 의 2~3배** 로 두번째 임계값을 정한다.


## 모폴로지 Morphology
모폴로지는, 보통 바이너리 이미지에서 흰색으로 나타나는 영역의 형태를 개선하기 위해 사용되는 처리 기법이다. Erosion, Dilation, Opening, Closing의 연산이 있다.<BR>
각 함수들을 쓰는 것은 어렵지 않으나, 각각의 커널 크기와 반복 횟수에 따라 결과가 달라지기 때문에 어떻게 다른지 아래 그림을 참고하면 좋겠다.

### Erosion
Erosion은 흰 오브젝트의 외곽 픽셀을 검정색으로 만들어 노이즈(흰색 작은 물체)를 제거하고, 붙어있는 오브젝트를 분리하는 데 주로 쓰인다. 커널의 사이즈와 반복의 정도를 조절하는 방법은 아래와 같다.

~~~C++
Mat kernel = getStructuringElement(MORPH_RECT, Size(3,3)) ;
erode(img_gray, img_result, kernel, Point(-1,-1), 1) ;
~~~

`erode` 함수에서 마지막 파라미터가 반복 횟수이고, 커널의 크기와 반복 횟수에 따른 정도 변화는 다음 그림을 참고하여라.

<p align="center">
  <img src="/assets/images/results/erosion1.jpg" width="500px" height="280px">
  <img src="/assets/images/results/erosion2.jpg" width="500px" height="280px">
</p>

### Dilation
Dilation은 erosion과 반대로, 흰 오브젝트의 외곽 픽셀 주변을 흰색으로 만들어서 erosion 이후 작아진 오브젝트를 원래대로 돌리거나, 인접해 있는 오브젝트를 연결해 하나로 만드는 데 주로 쓰인다. 커널의 사이즈와 반복의 정도를 조절하는 방법은 아래와 같다.

~~~C++
Mat kernel = getStructuringElement(MORPH_RECT, Size(3,3)) ;
dilate(img_gray, img_result, kernel, Point(-1,-1), 1) ;
~~~

커널의 크기와 반복 횟수에 따른 정도 변화는 다음 그림을 참고하여라.

<p align="center">
  <img src="/assets/images/results/dilation1.jpg" width="500px" height="280px">
  <img src="/assets/images/results/dilation2.jpg" width="500px" height="280px">
</p>

### Opening
Erosion 이후 Dilation을 적용하는 것이다.<br>
**노이즈를 제거하는 데** 주로 사용된다. Erosion을 통해 노이즈를 제거하면 오브젝트 자체도 줄어들게 되는데, 이를 다시 dilation으로 늘리는 방식이다.
사용법과 결과는 아래와 같다.

~~~C++
Mat kernel = getStructuringElement(MORPH_RECT, Size(3,3)) ;
morphologyEx(img_gray, img_result, MORPH_OPEN, kernel) ;
~~~

<p align="center">
  <img src="/assets/images/results/opening.jpg" width="400px" height="300px">
</p>

### Closing
Dilation 이후 Erosion을 적용하는 것이다.<br>
**흰 오브젝트 상의 작은 구멍을 메꾸는 데** 주로 사용된다.
사용법과 결과는 아래와 같다.

~~~C++
Mat kernel = getStructuringElement(MORPH_RECT, Size(11,11)) ;
morphologyEx(img_gray, img_result, MORPH_CLOSE, kernel) ;
~~~

<p align="center">
  <img src="/assets/images/results/closing.jpg" width="400px" height="400px">
</p>


## 허프 변환 Hough Transform

### Hough Line Transform
허프 라인 변환은 이미지 상에서 직선을 찾기 위해 사용되는 변환이다.<br>
원리를 간단하게 설명하자면, 한 점을 지나는 모든 직선에 대해 원점으로부터 직선의 수직 거리 **r** 과, x축 사이의 각도를 시계 반대방향으로 측정한 값인 **θ** 를 구하면,<br>
`r = xcosθ + ycosθ` 를 그려서, 해당 좌표를 지나는 모든 직선의 그래프를 그릴 수 있게 된다. (사인 곡선 형태)

이때 좌표 세 점을 지나는 모든 직선에 대해 **r** 과 **θ** 를 구하여 그려보면 3개의 사인곡선이 그려지는데, 이 곡선이 **한 점에서 만나는 것** 을 확인할 수 있다고 한다.

이 교차점을 구하기 위해 **2차원 배열에 사인 곡선을 누적** 하여 일정 개수 이상이 되면, 그 곡선이 교차한 지점의 r, θ값을 사용해 직선을 구할 수 있다.

<p align="center">
  <img src="/assets/images/results/1.jpg" width="200px" height="200px">
  <img src="/assets/images/results/2.jpg" width="200px" height="200px">
  <img src="/assets/images/results/3.jpg" width="300px" height="200px">
</p>

openCV 에서는 이러한 방식으로 직선을 출력하는데, 모든 점에 대해 계산하는 `HoughLines` 함수와, 이를 최적화해 임의의 점을 사용하는 함수인 `HoughLinesP` 함수를 제공한다.

두 함수를 사용했을 때 결과는 다음과 같다.

<p align="center">
  <img src="/assets/images/results/hough.png" width="300px" height="300px">
  <img src="/assets/images/results/houghp.png" width="300px" height="300px">
</p>

첫번째 결과는 `HoughLines` 함수, 두 번째는 `HoughLinesP` 함수 사용했을때의 결과이다. 확실히 모든 선에 대해 수한 `HoughLines` 함수가 더 잘 나오는 것을 확인할 수 있지만, 느리다는 단점이 있다.

-----
이렇게 이미지에 대해 블러링과 여러 모폴로지를 적용하고, 선을 추출하는 방법을 알아보았다.<br>다음 장에선 영상 분할과 템플릿 매칭, 컨투어 등 여러가지 Image Segmentation 기법을 알아보도록 하겠다.
