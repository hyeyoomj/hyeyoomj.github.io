---
title:  "openCV 프로그래밍 - 이미지 연산 및 ROI"
excerpt: "알짜배기 예제로 배우는 openCV - 2"

categories:
  - openCV
tags:
  - openCV
  - c++
  - python
  - computervision
  - roi

last_modified_at: 2020-06-30T08:06:00-05:00

---

# 알짜배기 예제로 배우는 openCV - 2

## OpenCV 란?
OpenCV란 오픈소스 컴퓨터 비전 라이브러리로, 공통 API를 사용하여 컴퓨터 비전 또는 영상처리 프로그래밍을 할 수 있는 환경을 제공한다. 또한 윈도우, 맥, 리눅스, iOS, 등 다양한 운영체제에서 사용이 가능하다는 장점도 갖고 있다.
이 책에서는 openCV의 개념과 이론 보다는, 주요 openCV 함수와 알고리즘을 파이썬과 c++를 사용한 여러 예제를 통해 소개한다. 빠르게 openCV 프로그래밍에 익숙해 질 수 있고, 필요한 openCV 기능들을 찾아 사용할 수 있다는 장점이 있다.

또한, 동일한 예제에 대해 c++과 파이썬 두 언어를 모두 보여주기 때문에 언어에 구애받지 않고 프로그래밍을 할 수 있다는 장점이 있다. 무엇보다 코드 마다 **상세한 주석** 이 달려있기 때문에 코드 만을 보고도 openCV와 그 사용 방법에 대해 잘 이해할 수 있다.

시작하기 앞서, 해당 게시물은 본인이 이 책을 읽으면서 openCV의 기본 개념 정리와 새롭게 알게 된 내용을 정리하는 게시물임을 알려 드린다. 상세한 코드 설명이나 예제 모두를 다루는 것은 아니며, 핵심 코드의 일부와 사용 방법을 다루고 모든 코드는 c++을 사용한다.


## 이미지 연산

### 블렌딩 Blending
블렌딩이란, 2개의 입력 이미지의 투명도를 조절하여 2개의 이미지가 겹쳐보이도록 만드는 것이다. openCV에서는 블렌딩을 위해 `addWeighted` 함수를 제공하는데, 이 함수는 다음과 같이 작동한다.

```c++
dst = src1 * alpha + src2 * beta + gamma
```

상수 alpha와 beta를 가중치로서 사용하여 이미지 소스의 투명도를 조절하는 것이다. 이때 상수의 범위는 **0.0 ~ 1.0** 이다.

아래와 같은 코드로 각 이미지1(손), 이미지2(강아지)에 대해 가중치를 다르게 하면서 이미지를 블랜딩한 결과이다. 각각의 alpha와 beta의 값은 다음과 같다.

```c++
addWeighted(img1, alpha, img2, beta, 0, dst) ;
```

<p align="center">
  <img src="/assets/images/results/blending1.png" width="400px" height="300px">
  alpha = 0.2, beta = 0.8
  <img src="/assets/images/results/blending2.png" width="400px" height="300px">
  alpha = 0.7, beta = 0.3
</p>

### 이미지 차 Image Subtract
이미지 차는  배경 이미지와, 같은 배경에서 객체가 있는 이미지를 빼어 차영상을 얻는 것이다. 아래와 같은 코드를 사용한다.

```c++
subtract(img_object, img_background, dst_sub) ;
```
이때 객체가 있는 이미지에서 객체가 없는 이미지를 빼야 한다. **순서** 를 헷갈리지 않도록 한다.<br>
이 차영상을 아래와 같이 이진화하면 더 좋은 결과를 얻을 수 있다.

<p align="center">
  <img src="/assets/images/results/subtraction.png" width="500px" height="300px">
</p>

```c++
threshold(dst_sub, dst_binary, 50, 255, THRESH_BINARY) ;
```

### 이미지 Bitwise Operation
이미지를 비트 별로 연산하여 합성할 수도 있다.
예를 들어, 어떤 로고를 이진화해 mask를 만들고, 그 이미지를 배경에 적용해 합성할 수 있는 것이다.

```c++
bitwise_and(logo, logo, img1, img_mask_inv) ; // 로고 이미지에서 배경 지우기
bitwise_and(img_roi, img_roi, img2, img_mask) ; // 배경 이미지에서 로고 지우기
add(img1, img2, dst) ; // 두 이미지 합치기
```
위 코드에서 `img_mask` 는 로고 이미지를 이진화 하여 mask를 만든 것이고, `img_mask_inv` 는 mask 이미지를 아래와 같이 연산하여 로고이미지에서 로고를 제외한 배경 만을 mask한 이미지를 만든 것이다.

```c++
bitwise_not(img_mask, img_mask_inv) ;
```

이런 식으로 한 결과는 아래와 같이 나온다. 이를 이용해 여러 실생활 디자인에도 적용해 볼 수 있겠다.

<p align="center">
  <img src="/assets/images/results/bitwise.png" width="500px" height="330px">
</p>


### 관심 영역 설정과 캐니 에지 ROI and Canny Edge

우리는 이미지에서 원하는 영역만을 잘라낼 필요가 있을 때가 있다. 잘라낼 영역을 관심영역, 즉 **Region of Interest(ROI)** 라고 부르는데, 이 부분에만 영상 처리를 할 수도 있다.

```c++
// 이미지 중심점에서 일정부분 roi로 설정
img_roi = img(Rect(center_x - 100, center_y - 100, 200, 200)).clone() ;

// roi 영역을 grayscale로 변환 후 캐니 에지 구하기
Mat img_gray ;
cvtColor(img_roi, img_gray, COLOR_BGR2GRAY) ;
Mat img_edge ;
Canny(img_gray, img_edge, 100, 300) ;
// 다시 컬러로 변환
cvtColor(img_edge, img_edge, COLOR_GRAY2BGR) ;

// 변환한 roi 영역 원래 이미지의 위치로 붙여넣기
img_edge.copyTo(img(Rect(center_x - 100, center_y - 100, 200, 200))) ;
```

이미지의 일부(중심점으로부터 일정부분)를 ROI 설정하고, 그 영역에만 grayscale로 변환 후 캐니 에지를 구한다.<br>
이후 변환한 ROI를 다시 컬러로 변환하여 (사이즈와 채널 수를 맞춰야 하기 때문에) 원본에 복사해서 보여주는 예제이다.

## 관심영역 ROI
위에서 본 것과 같이, 이미지의 일부를 지정하는 것을 ROI, Region of Interest 라고 한다. ROI를 추출해 해당 영역에만 다양하게 함수나 연산을 적용할 수가 있다. 이렇듯 이미지의 일부를 사이즈와 좌표로 추출할 수도 있지만, 마우스를 사용해 사용자가 직접 추출할 수도 있다.

### 실시간 ROI
웹캠 영상에 마우스를 사용하여 실시간으로 ROI를 검출하는 예제를 보겠다.<br>
마우스 이벤트가 발생하면 `mouse_callback` 함수를 호출하고, 해당 함수는 아래와 같이 커서의 위치를 스텝별로 계산해 ROI 영역의 시작점과 끝점을 지정한다.

```c++
void mouse_callback(int event, int x, int y, int flags, void *param) {
    if (event == EVENT_LBUTTONDOWN) {
        // 마우스 왼쪽 버튼을 누르면 ROI 시작점을 설정
        step = 1;
        mouse_is_pressing = true ;
        start_x = x ;
        start_y = y ;
    }
    else if (event == EVENT_MOUSEMOVE) {
        // 마우스 누른 상태에선 마우스 커서 위치를 ROI 끝점으로 저장
        if (mouse_is_pressing) {
            end_x = x ;
            end_y = y ;
            step = 2 ;
        }
    }
    else if (event == EVENT_LBUTTONUP) {
        // 마우스 떼면 ROI 끝점 설정됨
        mouse_is_pressing = false ;
        end_x = x ;
        end_y = y ;
        step = 3 ;
    }
}
```

위와 같은 함수를 메인 함수에서 호출하는 방법은 다음과 같다. 해당 함수를 호출해 마우스 이벤트를 제어하고, 마우스에서 손을 떼는 순간 지정된 좌표로 ROI를 설정한다.

```c++
setMouseCallback("Window name", mouse_callback) ;
// 마우스 왼쪽 버튼에서 손을 떼고 ROI 지정
Mat ROI(img, Rect(start_x, start_y, end - x-start_x, end_y - start_y)) ;
```

## 이미지 기하학적 변환

이미지의 기하학적 변환은 함수로 간단하게 구현이 가능하다.<br>
이미지의 회전, 크기 조정과 이동, 변환을 다루는 함수가 openCV로 지원이 되므로, 해당 함수들에 대해 간단하게 사용방법을 알아보고 넘어가도록 하겠다.


### 회전

회전 행렬을 사용하여 이미지를 회전시킬 수 있다.<br> openCV에서는 `getRotationMatrix2D` 함수를 사용해 회전 행렬을 생성하고, `warpAffine` 함수를 사용하여 이미지에 회전 변환을 적용한다.

```c++
Mat M = getRotationMatrix2D(
    Point(width / 2.0, height / 2.0), // 회전할 때 중심점
    45, // 회전 각도 (양수: 반시계방향, 음수: 시계방향)
    1 // 이미지 배율
) ;
```

이렇게 생성한 회전행렬을 사용하여 `img` 를 `img_rotated` 에 아래와 같이 적용하면 된다.
```c++
warpAffine(img, img_rotated, M, Size(width, height)) ;
```

### 크기 조정
이미지의 크기는 `resize` 함수를 사용하여 조정할 수 있다.<br>
이때 보간법 interpolation methods 을 지정해 줄 수 있는데, 지정해주지 않았을 때의 디폴트 값은 `INTER_LINEAR`이다. 확대할 시에는 `INTER_CUBIC` 혹은 `INTER_LINEAR`을, 축소할 시에는 `INTER_AREA` 를 권장한다.

```c++
// 가로 2배, 세로 2배로 이미지 확대
resize(img, dst, Size(), 2, 2, INTER_CUBIC) ;
// 너비와 높이를 0.5배로 축소
resize(img, dst, Size(0.5 * width, 0.5 * height), INTER_AREA) ;
```

### 이동
이미지의 모든 점을 같은 방향으로 이동시키는 변환을 이동 **translation** 으로 부른다.<br>
이동 시에는 이동행렬을 만들어 이미지에 적용하면 된다.

```c++
// 이미지를 오른쪽으로 100, 아래로 50 이동시키는 행렬을 만든다
Mat M(2, 3, CV_64F, Scalar(0.0)) ;
M.at<double>(0,0) = 1 ;
M.at<double>(1,1) = 1 ;
M.at<double>(0,2) = 100 ;
M.at<double>(1,2) = 50 ;
```
마찬가지로 만든 이동 행렬을 이미지에 적용시킨다.

```c++
warpAffine(img, img_translated, M, Size(width, height)) ;
```
### 아핀 변환 Affine Transformation
아핀 변환이란, 쉽게 말하면 이미지의 세 점을 찍어 그 세 점을 움직이는데, 이때 세 움직이는 위치는 대응하는 도형의 평행하게 움직여야 하는 것이다. 이미지로 보면 이해가 쉬울 것이다.

<p align="center">
  <img src="/assets/images/results/affine.png" width="500px" height="200px">
</p>


2D 평면에서, 임의의 삼각형을 또 다른 임의의 삼각형으로 매핑시킬 수 있는 변환이 affine이라고 생각하면 조금 더 이해가 갈 것이다. (단, **평행성을 보존하면서**)<br>
평행성을 보존한다는 것은 위 그림과 같이 점 p1, p2, p3를 p1', p2', p3'으로 매핑시키는 affine 변환을 구했을 때, 이 affine 변환을 가지고 p4를 매핑시키면 p4'이 나와야 한다는 의미라고 보면 된다.

이 변환 역시 `getAffineTransform` 함수를 사용해 변환행렬을 만들어 쉽게 변환이 가능하다.

```c++
Point2f src[3] ; // 원래 이미지의 점 위치
Point2f dst[3] ; // 변환된 이미지의 점 위치에 넣어주기
dst[0] = src[0] ;
dst[1] = Point2f(src[1].x, src[1].y+100) ; // src 점의 y좌표만 아래로 100 내리기
dst[2] = src[2] ;

Mat M = getAffineTransform(src, dst) ;
warpAffine(img, img_affine, M, Size(width, height)) ;
```

### 퍼스펙티브 변환 Perspective Transformation
퍼스펙티브 변환이란, 원본 이미지의 모든 직선은 출력 이미지에서도 직선으로 유지된 채로 변환하는 것을 말한다. 입력 이미지의 네 점과 대응하는 출력 이미지의 네 점이 필요하다.<br>
아래 그림과 같이, 일상생활에서도 쉽게 이 변환의 예를 찾아볼 수 있다.

<p align="center">
  <img src="/assets/images/results/perspective.png" width="450px" height="350px">
</p>

마찬가지로 `src`의 점을 지정해서 적당히 변환 후 `dst`의 점으로 넣어주는데,<br> 이때 보통 이미지가 똑바르게 보이도록 하기 위해 이미지의 네 모서리, 즉 `(0,0), (width,0), (0,height), (width,height)` 를 지정해준다. 그 후 퍼스펙티브 변환 행렬을 생성하고, `warp` 함수로 적용시키면 된다.

```c++
Point2f dst[4] ; // 변환된 이미지의 점 위치 지정
// 보통 이미지의 모서리 네 점을 지정한다

Mat M = getPerspectiveTransform(src, dst) ;
warpPerspective(img, img_result, M, Size(width, height)) ;
```

-----
여기까지 이미지에 연산과 기하학적 변환을 주는 방법을 알아보았다.<br>
다음 장에서는 이미지의 블러링, 모폴로지, 허프 변환과 히스토그램을 알아보도록 하겠다.
