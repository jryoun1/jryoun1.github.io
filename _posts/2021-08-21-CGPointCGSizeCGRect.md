---
title:  "[Swift] CGPoint, CGSize, CGRect에 대해서 알아보자."
excerpt: "View의 위치와 크기를 결정하는 방법에 대해서 알아보자."

categories:
  - Swift
tags:
  - [CGPoint, CGSize, CGRect ]
last_modified_at: 2021.08.21
--- 

## 들어가며
iOS에서 View를 그리기 위해서는 **위치와 크기**를 결정해주어야한다. 

예를 들어서 아래의 그림과 같이 파란색 뷰를 그리려면 해당 **뷰의 위치**를 알아야한다. 
![1](/assets/images/CGRect/1.png)

이때 iOS 뷰는 **왼쪽 위 모서리 끝을 기준점으로 하여 (0,0)**으로부터 시작하게 된다. <br>
그리고 View의 시작 위치를 알기 위한 x, y 좌표가 필요하다. <br>
위의 그림에서는 (100,200)이 x, y 좌표이다.

그리고 위에서 언급하였듯이 위치뿐만 아니라 크기도 결정해주어야한다. <br>
이때 Label과 같이 크기가 유동적이지 않은 것은 **width, height**를 알아야한다.
![2](/assets/images/CGRect/2.png)

따라서 결론적으로 View를 그릴 때 필요한 것은 `x, y, width, height` 4가지 이다. 

## CGPoint
![3](/assets/images/CGRect/3.png)

`CGPoint`는 2차원 좌표계의 **점**을 포함하는 구조체이다. 
```swift
public struct CGPoint {

    public var x: CGFloat

    public var y: CGFloat

    public init()

    public init(x: CGFloat, y: CGFloat)
}
```
`CGPoint`의 코드를 보면 다음과 같이 **CGFloat 타입의 x, y 값**을 가지는 구조체이다. <br>
`CGPoint`는 View를 그릴때만 사용하는 것이 아닌 x,y를 나타낼 땐 언제든지 사용할 수 있다. <br>
```swift
let pos: CGPoint = .init(x: 100, y: 150)
```
이런식으로도 위치를 나타내기 위해서 사용할 수 있다. 

## CGsize
![4](/assets/images/CGRect/4.png)

`CGSize`는 **너비와 높이 값**을 포함하는 구조체이다. <br>
⚠️ `CGSize`는 너비와 높이의 값이지, 사각형을 의미하는 것이 아니다. <br>
보통 일러스트를 위해 너비와 높이를 사각형으로 표현하지만, 그렇지 않다는 것을 명심하자!
```swift
public struct CGSize {

    public var width: CGFloat

    public var height: CGFloat

    public init()

    public init(width: CGFloat, height: CGFloat)
}
```
`CGSize`의 코드를 보면 **CGFloat 타입의 width, height** 값을 가지는 구조체이다. 
```swift
let size: CGSize = .init(width: 150, height: 200)
```
이런식으로 view의 사이즈를 설정할 수 있다. 

## CGRect
위에서 `CGPoint`, `CGSize`를 사용해서 위치와 너비를 설정한다고 했지만 <br>
막상 View를 그릴 때는 아래와 같이 frame의 파라미터로 `CGRect`가 들어가게 된다. 
```swift
let view: UIView = .init(frame: CGRect)
```
그렇다면 이 `CGRect`는 도대체 뭐하는 자식인가!!!!!

![5](/assets/images/CGRect/5.png)

`CGRect`는 **사각형의 위치와 크기**를 포함하는 구조체이다. <br>
위에서 배운 `CGPoint`, `CGSize`와는 다르게 사각형이라고 말하고 있다. <br>
이때 내부의 코드를 살펴보면 위치와 크기뿐만 아니라 원점(origin)을 가지고 있다. <br>
```swift
public struct CGRect {

    public var origin: CGPoint

    public var size: CGSize

    public init()

    public init(origin: CGPoint, size: CGSize)
}
```
`CGRect`는 **CGPoint타입의 origin, CGSize타입의 size**를 가지고 있다. <br>
바로 `CGPoint`와 `CGSize`를 품고 있는 것이다. <br>

따라서 View를 나타낼 때 **origin은 x,y 좌표**를 <br>
**size는 width, height**를 나타낸다고 생각하고 사용하면 된다. 
```swift
let rect: CGRect = .init(origin: CGPoint(x: 100, y: 200),
                         size: CGSize(width: 100, height: 150))
```
위와 같이 View의 frame에 접근할 수 있다. <br>
그리고 이는 아래와 같이 더 간단하게 작성할 수 있다. 
```swift
let rect: CGRect = .init(x: 100, y: 200, width: 100, height: 150)
let myView: UIView = .init(frame: rect)

myView.backgroundColor = .blue
self.view.addSubview(myView)
```
맨 처음에 그렸던 파란색 사각형을 그릴 수 있는 것이다. 
![6](/assets/images/CGRect/6.png)

## 마치며
CGPoint, CGSize, CGRect에 대해서 알아보았고 <br>
다음 포스팅에서는 Frame과 Bounds에 대해서 알아보자!

## 참고 
[소들이 CGPoint, CGSize, CGRect](https://babbab2.tistory.com/42) <br>
[ZeddiOS CGRect와 CGSize의 차이, 그리고 CGPoint](https://zeddios.tistory.com/201) <br>
[Apple Document CGPoint](https://developer.apple.com/documentation/coregraphics/cgpoint) <br>
[Apple Document CGSize](https://developer.apple.com/documentation/coregraphics/cgsize) <br>
[Apple Document CGRect](https://developer.apple.com/documentation/coregraphics/cgrect) 