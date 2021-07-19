---
title:  "[Swift] CustomStringConvertible"
excerpt: "CustomStringConvertible 프로토콜에 대해서 알아보자!"

categories:
  - Swift
tags:
  - [CustomStringConvertible]
last_modified_at: 2021.01.14
---

## 들어가며
프로젝트를 진행하는 도중에 캠퍼로부터 다음과 같은 리뷰를 받은 적이 있었다. 
```swift
enum CustomerPriority: String, CaseIterable {
     case VVIP = "0"
     case VIP = "1"
     case normal = "2"

      var description: String {
         switch self {
         case .VVIP:
             return "VVIP"
         case .VIP:
             return "VIP"
         case .normal:
             return "일반"
         }
     }
 }
```
손님의 우선순위라는 Enum 타입이 있고 안에 description이라는 변수가 있다. 
이에 대해서 팀원이었던 **Lina**가 `CustomStringConvertible`을 사용해보는 것은 어떤지 리뷰를 남겨주었다.
고마워요 **Lina**🙇‍♂️

## CustomStringConvertible
![](https://images.velog.io/images/minni/post/607b5134-d75d-4bec-a7b3-629eb1ae6289/image.png) <br>
먼저 공식문서를 살펴보면 `CustomStringConvertible`은 protocol이라고 한다. <br>
그렇다면 어떠한 기능들을 가지고 있는지에 대해서 알아보도록 하자!!

### 역할
`CustomStringConvertible` 프로토콜을 채택한 type은 instance를 
문자열로 변환할 때 사용할 고유한 표현을 제공해준다고 한다. 

`String(describing:)` initializer는 모든 type의 instance를 문자열로 변환해주는 방법이다. 
만약 전달된 instance가 `CustomStringConvertible`을 채택하고 있는 경우에는 
`String(describing:)` initializer와 `print(_:)` 함수는 instance의 custom된 
description 프로퍼티를 사용하게 된다. 

type의 description 프로퍼티에 직접적으로 접근하거나, `CustomStringConvertible`을 일반 제약 조건으로 사용하는 것은 권장하지는 않는다고 한다.

### 사용 예시
위에서 역할을 글로만 보았을 때는 확실하게 와닿지는 않는다. 
실제 사용 예시를 확인해보자.
```swift
struct Point {
    let x: Int, y: Int
}

let p = Point(x: 21, y: 30)
print(p.x, p.y)
// Prints "21 30"
print(p)
// Prints "Point(x: 21, y: 30)"
```
다음과 같이 Point라는 구조체가 있을 때 구조체의 프로퍼티를 출력하고 싶은 경우에는 직접적으로 접근하거나 아니면 구조체를 출력하게 되면 위와 같은 결과가 나오는 것을 확인할 수 있다. 

이럴 때 `CustomStringConvertible`을 채택하여 구현하면 다음과 같이 구조체의 프로퍼티를 출력할 수 있다. 
```swift
extension Point: CustomStringConvertible {
    var description: String {
        return "(\(x), \(y))"
    }
}

print(p)
// Prints "(21, 30)"
```

맨 처음 코드리뷰를 받았던 코드도 `CustomStringConvertible` 채택하도록 수정해보겠다.
```swift
enum CustomerPriority: String, CaseIterable {
    case VVIP = "0"
    case VIP = "1"
    case normal = "2"
}

extension CustomerPriority: CustomStringConvertible {
    var description: String {
        switch self {
        case .VVIP:
            return "VVIP"
        case .VIP:
            return "VIP"
        case .normal:
            return "일반"
        }
    }
}

print("\(CustomerPriority.VVIP)") // Prints "VVIP"
print("\(CustomerPriority.VIP)") // Prints "VIP"
print("\(CustomerPriority.normal)")  // Prints "일반"
```
기존에는 `CustomerPriority.VIP.descritpion` 로 접근하여 해당 String을 출력할 수 있었다면, 
보간법을 사용할 때 `CustomStringConvertible` 을 사용하면 훨씬 가독성 있게 코드를 작성할 수 있을 것 같다.

## 참고
1. [Apple Document CustomStringConvertible](https://developer.apple.com/documentation/swift/customstringconvertible)