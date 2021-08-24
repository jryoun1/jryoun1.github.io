---
title:  "[Swift] frame과 bounds에 대해서 알아보자 - 2"
excerpt: "frame과 bounds의 origin을 변경해보자"

categories:
  - Swift
tags:
  - [frame, bounds, viewport, scrollview]
last_modified_at: 2021.08.24
--- 

## 들어가며
[지난 포스팅]({{site.url}}{{site.baseurl}}/swift/FrameBounds_1/)에서는 frame과 bounds의 origin, size의 차이에 대해서 알아보았다. 

이때 정리한 내용을 다시 간략하게 보면 아래와 같다. 

![13](/assets/images/FrameBounds/13.png)

그리고 이제 그렇다면 언제 frame, bounds를 써야하는지에 대해서 알아보자. <br>
차이를 알기 위해서는 **frame, bounds의 origin을 변경했을 때 무슨일이 발생하는지** 보자!! 

## frame origin 변경

![14](/assets/images/FrameBounds/14.png)

위의 그림과 같이 `grayview`, `yellowview`, `greenview` 가 있다. <br>
그리고 `greenview` superview = `yellowview`, <br>
`yellowview` superview = `grayview` 라는 계층 구조를 가진다. <br>

이때 `yellowview의` frame origin의 값을 변경해보면 아래와 같이 변경된다. 

![15](/assets/images/FrameBounds/15.png)

frame은 **superview 좌표계를 기준**으로 하기 때문에 `grayview`를 기준으로 <br>
(10,10)으로 위치가 변경되었다. 그리고 이때 `greenview` 역시도 같이 움직였는데 <br>
`greenview`의 superview인 `yellowview`가 이동했으므로 당연히 움직이게 된다. 

💡 따라서 frame의 origin을 변경하면 **subview도 같이 움직인다는 것**을 알 수 있다. 

## bounds origin 변경

이제 bounds의 origin을 변경해보자. 

![16](/assets/images/FrameBounds/16.png)

이번에도 `yellowview`의 bounds origin을 (0,0)에서 <br>
(20,20)로 이동했는데, 위의 그림과 같은 결과가 나왔다. 

분명 `yellowvie`w의 bounds origin을 (20,20)으로 이동했는데 <br>
마치 `greenview`의 bounds origin이 (-20,-20)만큼 이동한 것처럼 보여진다. 

이는 `frame`과는 전혀 다르다. `frame`에서는 origin의 값을 변경하면 superview를 기준으로 (x,y)만큼 이동했다면, `bounds`의 경우에는 origin을 변경한다고 해당 위치로 이동하는 것이 아니다. 

`yellowview`의 `bounds`의 origin을 변경하는 것은 바로 `yellowview`의 자체 좌표계에서 **viewport**를 (x,y)로 이동하는 것이다. 

### viewport

`viewport`는 실제로 화면이 보여지는 창을 의미한다. <br>
휴대폰 기기의 화면 사이즈는 크지 않기 때문에 화면 사이즈보다 큰 view의 경우에는 일부분만 보여지게 된다. 

![17](/assets/images/FrameBounds/17.png)

따라서 위의 그림과 같이 휴대폰 사이즈보다 큰 view의 경우에는 `viewport`만큼만 화면에 보여지게 된다.

![18](/assets/images/FrameBounds/18.png)

그리고 이때 view의 bounds origin을 변경하는 것은 바로 위의 그림처럼 **viewport를 (x,y)만큼 이동시키는 것**이다. 

⚠️ 이때 마치 `노란색`, `초록색`, `파란색` view의 frame이 변경된 것처럼 보일 수 있지만 실제로 그렇지 않다. 또한 `노란색`, `초록색`, `파란색` view가 (-20,-20)만큼 이동한 것처럼 보일 수 있지만 그렇지 않다는 점을 알아야한다. 즉, `subview`들의 frame이 **bounds origin을 이동한 것과 반대 방향으로 움직이는 것**처럼 보인다. 

💡 그러나 결국 view의 bounds origin을 변경하면 <br>
**frame이 움직이는 것이 아니라 `viewport`가 움직이는 것**이다! <br>

## frame의 사용

frame은 UIView의 **위치와 크기**를 나타낼 때 사용하게 된다. <br>
`위치`는 **superview의 좌표계**에서 나타내는 것이며<br>
`크기`는 **view 영역을 모두 감싸는 사각형의 크기**를 나타낸다. <br>

따라서 frame은 UIView를 코드로 그릴 때, 위치와 크기를 설정할 때 사용한다. <br>
```swift
let view: UIView = .init(frame: .init(x: 100, y: 150, width: 150, height: 100))
```

## bounds의 사용

bounds도 UIView의 **위치와 크기**를 나타낼 때 사용하게 된다. <br>
`위치`는 **자신의 좌표계**에서 나타내는 것이며 <br>
`크기`는 **view 영역 자체의 크기**를 나타낸다. 

### 변형 후 View의 실제 크기 확인

View를 transformation한 뒤에 해당 View의 실제 크기를 알고 싶을 때 사용할 수 있다.

### View 내부에 그림을 그릴 때 사용

View 내부에 drawRect로 그림을 그릴 때 사용할 수 있다. 

### ScrollView에서 사용

ScrollView에서 스크롤을 할 때 바로 이 bounds를 사용하게 된다. <br>
ScrollView는 실제로 휴대폰 화면보다 큰 view를 보여주고 싶을 때 사용하게 된다. 그리고 이때 스크롤을 통해서 원하는 콘텐츠를 볼 수 있는데, 이 스크롤을 하는 것이 바로 ScrollView의 **bounds를 바꿔주는 것**이다. 즉, ScrollView의 **subview중에서 어디를 보여줄 지 정해주는 것**이다. 

따라서 ScrollView에서 스크롤을 하면 bounds의 **origin이 계속해서 변화**하는 것을 확인해 볼 수 있다. 

## 마치며

frame과 bounds의 origin을 변경해보았고, 이를 통해서 두 개의 차이를 정확히 파악할 수 있었다. 또한 어디서 frame과 bounds를 사용하는지 알아 볼 수 있었다. 

![19](/assets/images/FrameBounds/19.png)


## 참고 
[소들이 frame bounds](https://babbab2.tistory.com/45?category=831129) <br>
[Apple Document Frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame) <br>
[Apple Document Bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds) <br>
