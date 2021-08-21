---
title:  "[Swift] frame과 bounds에 대해서 알아보자 - 1"
excerpt: "UIView에 나오는 frame과 bounds의 차이에 대해서 알아보자"

categories:
  - Swift
tags:
  - [frame, bounds, superview]
last_modified_at: 2021.08.22
--- 

## 들어가며
[지난 포스팅]({{site.url}}{{site.baseurl}}/swift/CGPointCGSizeCGRect)에서는 CGPoint, CGSize, CGRect에 대해서 다뤘었는데 <br>
그 이유는 바로 UIView의 **frame과 bounds**를 다루기 위한 큰그림이었다 ㅎㅎ 

```swift
extension UIView {

    open var frame: CGRect

    open var bounds: CGRect
```
`UIView`를 보면 CGRect 타입의 frame과 bounds를 가진다. <br>
CGRect는 **View의 위치(origin), 크기(size)의 값**을 가지는 사각형이다. <br>

따라서 frame과 bounds는 **View의 위치와 크기**를 가진다는 것을 알 수 있다. <br>
그렇다면 아래와 같이 view를 하나 생성하고 frame과 bound값을 확인해보자. 
```swift
let view = UIView(frame: CGRect(x: 100, y: 200, width: 200, height: 150))
view.backgroundColor = .green
        
self.view.addSubview(view)
print("view의 frame origin: \(view.frame.origin)")
print("view의 frame size: \(view.frame.size)")
print("view의 bounds origin: \(view.bounds.origin)")
print("view의 bounds size: \(view.bounds.size)")
```
![1](/assets/images/FrameBounds/1.png)![2](/assets/images/FrameBounds/2.png)

frame과 bounds의 size는 동일하지만, **origin의 값이 다른 것**을 확인할 수 있다. <br>
⚠️ frame과 bounds의 차이점이 origin만 다르다는 착각은 금지! <br>
이제부터 size도 어떻게 다른지 알아보자.

## frame
![3](/assets/images/FrameBounds/3.png)

`frame`은 **superview 좌표계에서 View의 위치와 크기를 나타낸다.**

### superview
그렇다면 superview는 무엇인가? <br>

iOS에서는 화면을 구성할 때 `UIView`를 이용해서 구성하게 된다. <br>
예를 들면 `UILabel`, `UIButton` 등은 모두 `UIView`를 상속받고 있다. <br>
따라서 이러한 View로 화면을 구성할 때에는 **계층 구조**라는 것이 있다.

![4](/assets/images/FrameBounds/4.png)

위의 그림에서 ViewController에는 기본적으로 `view(회색)`가 있고 <br>
그 위에 `view(주황색)`를 올리고, 또 그 위에 `view(초록색)`를 올린 것이다. <br>
이때 **superview는 자신의 view 바로 한칸 윗 계층 view를 의미**한다. <br>

따라서 `초록색 view`의 superview는 `주황색 view`이고 <br>
`주황색 view`의 superview는 `회색 view`이다.

### frame의 origin
frame의 origin은 **super view의 원점을 (0,0)으로 놓고 원점으로부터 얼마나 떨어져 있는지**를 나타낸 것이다. 이때 원점은 view의 가장 왼쪽, 가장 윗 부분을 말한다. 
superview의 원점이 곧 좌표의 시작점(0,0)이 되니까, **superview의 좌표계에서 나타낸다**고 할 수 있다. 

즉, superview의 **원점으로부터 얼만큼 떨어져 있는지**를 의미한다. <br>
그렇다면 아까 예시에서의 frame orgin을 확인해보자. 

![5](/assets/images/FrameBounds/5.png)

먼저 `주황색 view`의 superview는 `회색 view`였다. <br>
따라서 `회색 view`의 가장 왼쪽, 가장 윗부분을 (0,0)으로 하여 (40,180)만큼 떨어져있어 <br>
`주황색 view`의 frame.origin은 (40, 180)이 된다. 

![6](/assets/images/FrameBounds/6.png)

`초록색 view`의 superview는 `주황색 view`이다. <br>
따라서 `주황색 view`의 가장 왼쪽, 가장 윗부분을 (0,0)으로 하여 (20,40)만큼 떨어져있어 <br>
`초록색 view`의 frame.origin은 (20,40)이 된다. <br>

이때 `주황색 view`의 origion은 (40,180)이지만 **여기서는 전혀 중요하지 않다.** <br>
단지 원점에서 얼만큼 떨어져 있는지만 나타내면 되기 때문에 (20,40)이 되는 것이다. 

### frame의 size
frame의 size는 **View 영역을 모두 감싸는 사각형**이다. <br>

![7](/assets/images/FrameBounds/7.png)

다음과 같이 width=200, height=150인 `갈색 view`가 있을 때 <br>
`view.frame.size`를 출력해보면 위와 같이 지정한 값으로 출력된다. 

```swift
view.transform = .init(rotationAngle: 60)
```
그러나 `갈색 view`를 회전시키면 다음과 같이 변화하게 된다. <br>

![8](/assets/images/FrameBounds/8.png)

즉, 지정한 값이 아닌 다른 값으로 **width, height, origin이 모두 변한다**. <br>
이유는 위의 frame size의 정의가 **view 영역을 모두 감싸는 사각형**이기 때문이다. <br>

![9](/assets/images/FrameBounds/9.png)

결국 그림과 같이 **View가 차지하는 영역을 감싸서 만든 사각형**이 바로 frame의 size이다. <br>
그리고 이때 frame의 origin도 기존의 (100,100) → (81,73)처럼 변할 수 있다. <br>
이러한 것이 바로 frame의 속성이다. 

## bounds

![10](/assets/images/FrameBounds/10.png)

`bounds`는 **자신의 좌표계에서 View의 위치와 크기**를 나타낸다. <br>
frame이 superview의 좌표계였다면 bounds는 **자신의 좌표계**이다. <br>

### bounds의 origin
`bounds`의 좌표계는 frame과 달리 superview와는 전혀 상관이 없다. <br>
그리고 **자기 자신의 원점(가장 왼쪽 가장 위)을 (0,0)으로 설정**한다. <br>
처음에 View를 생성하면 bounds의 **origin은 무조건 (0,0)으로 초기화** 되어있다. <br>

![11](/assets/images/FrameBounds/11.png)

따라서 `오랜지 view`의 bounds origin도 (0,0)이고 <br>
`초록색 view`의 bounds origin도 (0,0)이다. <br>

지금은 bounds의 origin은 **자신의 좌표계를 기준**으로 하며, <br>
**처음엔 (0,0)으로 초기화** 되는 것만 알고 추후의 포스팅에서 bounds가 바뀌는 경우에 대해서 알아보자<br>

### bounds의 size
`bounds`의 size는 **View 자체의 영역을 나타내는 것**이다. <br>
따라서 frame size와는 bounds size는 **view 자체 영역의 width, heigth만 나타낸다**. <br>

![12](/assets/images/FrameBounds/12.png)

frame size에서는 view가 차지하는 영역을 감싸서 만든 사각형의 origin,size로 변하지만 <br>
bounds size는 회전을 해도 전혀 변경되지 않고 그대로 유지된다. <br>
이유는 바로 **View 자체의 영역을 나타내기 때문에 origin, size 모두 변하지 않는다.** 

## 마치며

frame과 bounds를 비교하면 다음과 같이 나타낼 수 있다. <br>

![13](/assets/images/FrameBounds/13.png)

이번 포스팅에서는 개념을 알아봤다면 다음 포스팅에서는 좀 더 자세하게 frame, bounds를 사용하는 곳과 이유에 대해서 포스팅 할 예정이다. 

## 참고 
[소들이 frame bounds](https://babbab2.tistory.com/44) <br>
[Apple Document Frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame) <br>
[Apple Document Bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds) <br>
