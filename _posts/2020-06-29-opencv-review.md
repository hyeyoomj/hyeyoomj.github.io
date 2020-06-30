---
title:  "openCV 프로그래밍 - 기본 이론 및 이미지 객체 사용"
excerpt: "알짜배기 예제로 배우는 openCV - 1"

categories:
  - openCV
tags:
  - openCV
  - c++
  - python
  - computervision

last_modified_at: 2019-06-29T08:06:00-05:00

---

# 알짜배기 예제로 배우는 openCV - 1

## OpenCV 란?
OpenCV란 오픈소스 컴퓨터 비전 라이브러리로, 공통 API를 사용하여 컴퓨터 비전 또는 영상처리 프로그래밍을 할 수 있는 환경을 제공한다. 또한 윈도우, 맥, 리눅스, iOS, 등 다양한 운영체제에서 사용이 가능하다는 장점도 갖고 있다.
이 책에서는 openCV의 개념과 이론 보다는, 주요 openCV 함수와 알고리즘을 파이썬과 c++를 사용한 여러 예제를 통해 소개한다. 빠르게 openCV 프로그래밍에 익숙해 질 수 있고, 필요한 openCV 기능들을 찾아 사용할 수 있다는 장점이 있다.

또한, 동일한 예제에 대해 c++과 파이썬 두 언어를 모두 보여주기 때문에 언어에 구애받지 않고 프로그래밍을 할 수 있다는 장점이 있다. 무엇보다 코드 마다 **상세한 주석** 이 달려있기 때문에 코드 만을 보고도 openCV와 그 사용 방법에 대해 잘 이해할 수 있다.

------------

## 기본 개념 정리

이미지는 픽셀(pixel)이라고 부르는 작은 점이 2차원 matrix 상에 배치되어 구성된다.
이미지의 크기(size)는 보통 이 matrix의 너비 픽셀 개수 x 높이 픽셀 개수를 말한다.
openCV에서는 이미지의 왼쪽 위에 원점(0,0)이 위치하고, 한 픽셀에 접근 할 때 보통 **(y,x)** 순서를 사용한다.
컬러 이미지에서는 각 픽셀이 **(Blue, Green, Red)** 세개의 채널을 가지며, 이를 Gray Scale로 변환하면 하나의 채널을 갖는 픽셀로 구성된다. 이진화된 이미지 (Binary Image)에서는 모든 픽셀이 0 또는 255의 값을 갖게 된다.

<p align="center">
  <img src="/assets/images/grayscale.png" width="230px" height="230px" alt="grayscale">
  <img src="/assets/images/color.png" width="330px" height="250px" alt="color">
</p>

### Mat 객체
openCV 상에서, 이미지는 Mat 객체에 저장된다. (c++의 경우) (python의 경우 numpy를 사용한다)
아래와 같이 Mat 객체를 생성해서 사용하는데, 이때 이 이미지를 copy 한 이미지의 경우, 한 이미지에 변경되는 모든 속성들이 동일하게 적용됨을 유의해야 한다.

즉, 대입 연산자는 같은 이미지를 가리키게 되는 것이고 결국 같은 객체가 된다.
이미지의 복사본을 생성해 서로 다른 이미지가 되게끔 하려면 clone, 혹은 copyTo를 사용하면 된다.

* 픽셀 단위 접근
Mat 객체로 선언한 이미지의 픽셀 단위로 접근하는 방법은 다음과 같다.
~~~
for (int y = 0; y < height; y++) {
  // y좌표를 사용하여 포인터 위치 미리 계산
  uchar* pointer_input = img_color.ptr<uchar>(y) ;
  uchar* pointer_output = img_gray.ptr<uchar>(y) ;
  for (int x = 0; x < width; x++) {
    uchar b = pointer_input[x * 3 + 0] ;
    uchar g = pointer_input[x * 3 + 1] ;
    uchar r = pointer_input[x * 3 + 2] ;
    // BT.702 비율 사용해서 컬러 이미지를 grayscale 값으로 변환
    pointer_output[x] = r * 0.2126 + g * 0.7152 + b * 0.0722 ;
  }
}
~~~
이 예제는 컬러 이미지의 픽셀에 접근하여 그레이 스케일로 변환하는 부분이다. for문을 돌리면서 픽셀에 하나씩 접근하고, 컬러 이미지의 한 픽셀에 3개의 채널이 있기 때문에 컬러 x좌표에 3을 곱해주고 그레이 스케일 값을 계산해서 저장한다.

* 채널 분리 및 합치기
컬러 이미지를 각 R, G, B 채널로 나눌 수도 있다.
~~~
Mat img_channels[3] ; // 배열에 b, g, r 순으로 채널이 저장됨
split(img_color, img_channels) ; // 채널 나누기
vector<Mat> channels ;

Mat img_result ;
merge(channels, img_result) ; // 나눈 채널 합치기
~~~
컬러 이미지를 split 함수를 사용해 채널 별로 분리하고, 채널별 이미지를 channels로 할당해 원하는대로 바꾼 다음, merge 함수로 합쳐 결과를 만든다.

### 영상 다루기

openCV 에서 영상은, **연속적인 이미지** 로 다루어진다. 아래와 같이, 이미지를 윈도우에 보여주는 작업을 반복하여 화면에 영상으로 보여지도록 한다.

~~~
Mat img_frame ;
VideoCapture cap(0) ;
while (1) {
  cap.read(img_frame) ;
  imshow("video", img_frame) ;
  if (waitKey(1) == 27)
}
~~~

이런 식으로 카메라에서 이미지 캡쳐와 윈도우에 이미지 보여주기를 반복하여 동영상으로 보이게 하는 것이다.

---------
여기까지 openCV 에서 가장 기본적인 이미지와 영상을 다루는 법을 익혔다. 다음 장에서는 이미지에 다양한 연산과 함수들을 적용시켜 보도록 하겠다.
