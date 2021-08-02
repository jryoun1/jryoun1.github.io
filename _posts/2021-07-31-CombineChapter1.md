---
title:  "[Combine] Chapter1 Combine이란"
excerpt: "Combine이 어떠한 문제를 해결해주고, Apple 플랫폼의 reactive 프로그래밍 시작 이야기에 대해서 알아보자"

categories:
  - Combine
tags:
  - [Publisher, Operators, Subscribers, Subscriptions]
last_modified_at: 2021.07.31
---

## 들어가며
사이드 프로젝트를 진행하기로 했는데, MVVM 모델에 Combine을 적용해보기로 해서 Combine에 대해 공부가 필요한 것 같아서 [Combine: Asynchronous Programming with Swift, Chapter 1: Hello, Combine!](https://www.raywenderlich.com/books/combine-asynchronous-programming-with-swift) 책을 읽으면서 적용해보기로 했다. 

> Apple의 말에 따르면 Combine framework는 app이 이벤트를 처리하는 방법에 있어서 선언적인 접근 방법을 제공한다고 한다. 즉, 여러 개의 delegate callback이나 completion handler 클로저를 수행하는 것이 아니라 **하나의 처리 체인을 생성하여 이를 해결하는 방법**을 제공한다. 이때 각각의 체인은 **Combine Operator** 이며, 이전 단계로부터 받은 요소에 별개의 action을 수행하게 된다. 

처음 위의 내용을 읽으면, 대충 이해는 되지만 그래도 정확하게 이해는 되지 않을 것이다. 그러니까 이번 Chapter에서는 Combine의 기본적인 개념에 대해서 알아보자. 

## 비동기적 프로그래밍

```swift 
beign
    var name = "Jake"
    print(name)
    name += " Coding"
    print(name)
end
```
다음과 같이 하나의 스레드에서 실행되는 수도 코드가 있다고 할 떄, 프로그램은 한 줄 씩 실행하게 될 것이다. 따라서 이러한 동기적인 코드는 이해하기 쉬울 것이다. 

그러나 멀티 스레드 환경에서 비동기적 이벤트를 처리한다고 생각해보자. 
```swift
--- Thread 1 ---
beign
    var name = "Jake"
    print(name)

--- Thread 2 ---
name = "Lasagna Lee"

--- Thread 1 ---
    name += " Coding"
    print(name)
end
```
다음과 같이 실행된다고 할 때, 과연 결과로 Jake Coding이 나올 지 Lasanga Lee Coding이 나올지는 매번 실행할 때 마다 다른 결과를 낼 것이다. 왜냐하면 여러 개의 스레드가 동시에 작동하기 때문에 어느 것이 먼저 실행된다는 것을 보장할 수 없기 때문이다. 

## Foundation과 UIKit/AppKit

Apple은 수 년간 플랫폼에서 비동기적 프로그래밍을 개선해왔고, 비동기 코드를 생성하고 실행하는데 사용할 수 있는 몇 가지 메커니즘을 만들었다. 
- `NotificationCenter`
    - 이벤트가 발생할 때 작성한 코드를 실행시켜준다 
    - 예) 사용자가 휴대폰 기기 방향 변경, 스크린에서 키보드의 나타남과 숨김
- `Delegate Pattern`
    - 다른 객체를 대신하거나 다른 객체와 함께 조정하는 객체를 정의
    - 예) app delegate는 notification이 도착하면 어떠한 것을 할 지 정의되어 있지만, 실제로 언제 실행되는지 몇 번이나 실행될지는 전혀 모른다
- `Grand Center Dispatch(GCD) & Operations`
    - 우선순위에 따라 여러개의 큐에서 동시에 실행되게 할 수도 있고, 직렬 큐에서 순서대로 실행되게 할 수도 있다
- `Closures`
    - 코드 내부에서 전달할 수 있는 코드 블럭을 만들어 다른 개체가 어디서, 몇 번 실행할 지 결정할 수 있다

대부분의 코드들은 비동기적으로 작업을 수행하며, 모든 UI이벤트는 본질적으로 비동기적이기 때문에 App 코드 전체가 실행될 순서를 예측할 순 없다.

그럼에도 비동기 프로그램을 작성할 수 있고, 이러한 비동기적 코드와 리소스의 공유는 해결이 어려운 문제를 발생하게 된다. 이러한 문제의 확실한 원인 중 하나는 바로 실제 App이 아래의 그림과 같이 각각의 자체 인터페이스를 갖춘 **다양한 종류의 비동기 API를 모두 사용**하기 때문이다.

![1](/assets/images/Combine/Chapter1/1.png)

이에 `Combine`은 비동기적 프로그래밍을 도와줄 새로운 Swift 환경의 언어이다. Apple은 Combine API를 Foundation Framework에 통합시켰기 때문에, Timer, NotificationCenter 그리고 CoreData와 같은 core framework에서도 사용이 가능하다. 따라서 우리가 작성한 코드와도 아주 쉽게 통합할 수 있다. 

그리고 Apple은 `SwiftUI`라는 Combine과 통합해서 사용하기 쉬운 새로운 UI framework를 소개하였다. Foundation에서 SwiftUI까지 다양한 system framework는 Combine에 의존하며, 그들의 기존의 APIs들을 대신할 수 있게 Combine 통합을 제공한다. 

`Combine`은 Apple의 framework이기 때문에 Timer나 NotficationCenter와 같이 이미 검증된 것들의 역할을 뺏으려는 것이 아니라, 기존의 것들은 기존의 역할을 하고 있고 대신 Combine을 사용해서 **App에서 비동기적으로 동작하는 모든 타입들을 이 언어를 통해 통합**되게 해주는 것이다. 

## Combine Foundation 

선언적이며, 반응적인 프로그래밍은 새로운 개념이 아니지만 최근 10년동안 눈에 띄게 발전하게 되었다. 2009년에 Microsoft에서 .NET을 위한 **Reactive Extension** 라이브러리를 발표하였다. 그리고 2012년에 이를 open source로 제공하였고 이후 RxJS, RxKotlin, RxScala, RxPHP 등과 같ㅇㄴ 많은 Rx 표준들이 생겨났다.

Apple 플랫폼에서는 Rx 표준을 구현하는 RxSwift, Rx에서 영감을 얻어 반든 ReactiveSwift 등과 같은 framework가 있다. `Combine`은 Rx와는 다르지만 **Reactive Streams라는 유사한 표준을 구현**하고, Rx와 몇가지 주요 차이점이 있지만 대부분의 핵심 개념은 동일하다. 

현재는 iOS 13/macOS Catalina 이상을 지원하는 App에서만 Combine framework를 사용할 수 있다.

## Combine 기초

Combine에는 `publisher`, `operators`, `subscriber` 라는 3가지 중요 개념이 있다. 물론 다른 개념들도 있지만 3개 없이는 사용하기가 어렵다. 

### Publisher
> 시간의 경과에 따라 **value를 subscriber에게 방출** 하는 타입

`Publisher` 내부 로직이 어떠하든 아래의 3가지 타입에 대해서 여러 이벤트를 방출 가능하다. 
- `output value` : publisher의 제네릭 Output type에 맞는 결과 값
- `successful completion`
- `error` : publisher의 Failure 타입에 맞는 error

이때 publisher는 0개 이상의 output 값을 방출 할 수 있으며, 성공적이거나 오류로 인해 완료가 되는 경우 다른 이벤트를 방출하지는 않는다. 

![2](/assets/images/Combine/Chapter1/2.png)

위의 그림은 시간의 흐름에 따라서 publisher가 방출하는 Int value들이다. 이때 오른쪽 끝의 `0:25`의 화살표부분은 **successful stream completion**을 의미한다.

위에서 언급한 3가지 타입은 프로그램의 어떠한 동적 데이터 타입도 표현이 가능할 정도로 광범위하기 때문에 Combine `publisher`를 사용해서 모든 작업을 할 수 있는 것이다. 즉, 기존에는 네트워크 콜을 생성하거나, 사용자 제스처에 반응하거나, 데이터를 화면에 표시하는 작업들에서 자신의 도구상자에서 작업에 적합한 도구를 항상 찾는 것을 대신하여 delegate를 추가하거나, completion callback을 삽입했었다. 그러나 이제는 `publisher`를 사용하여 이를 해결할 수 있다. 

`publisher`의 가장 큰 장점 중 하나는 바로 에러 처리를 마지막에 선택적으로 추가하는 것이 아니라 **에러 처리가 내장**되어 있다는 것이다. `Publisher` 프로토콜은 두 가지 제네릭 타입을 따르게 된다. 
- `Publisher.Output` : publisher의 output value의 타입을 의미하며, 한 번 **특정한 타입으로 정해지면 다른 타입을 방출할 수 없다.**
- `Publisher.Failure` : publisher가 실패했을 때 던질 수 있는 error 타입을 의미하며, 만약 publisher가 **절대 실패할 일이 없다면 `Never` failure type을 명시**하면 된다. 

어떠한 publisher를 subscribe할지 안다면, 기대되어지는 값과 실패했을 때 던지는 error들을 알 수 있다. 

### Operators
> `Publisher` 프로토콜에 선언된 **메서드**로, 동일하거나 새로운 publisher를 반환하는 타입

이는 하나의 operator 다음에 또 다른 operator들을 **체이닝하면서 호출할 수 있기 때문에 매우 효율적**이다. 이러한 operator 메서드들은 매우 잘 분리되고 또 구성될 수도 있기 때문에 **복잡한 로직을 한 개의 subscription을 실행하고 이어서 작성(combine)**할 수 있다. 

그리고 이때 operator들은 잘못된 순서이거나, output type이 input type과 맞지 않는 경우에는 사용할 수 없게 된다. 즉, operator들은 아래의 그림과 같이 퍼즐처럼 **타입과 순서를 잘 맞춰서 사용해야한다**는 것이다. 

![3](/assets/images/Combine/Chapter1/3.png)

이때 각각의 작업의 순서는 올바른 input/output 타입과 내장된 에러 처리를 통해서 추상적인 비동기 작업을 정의할 수 있다. 그리고 operator는 항상 **upstream**으로 여겨지는 input, **downstream**으로 여겨지는 output이 있기 때문에, **shared state를 피할 수 있다.** 즉, 이전 operator로부터 data를 받고, 다음 operator에게 output을 제공하므로 다른 비동기 작업이 중간에 끼어들지 못한다는 것이다. 

### Subscribers
> Subscription chain의 마지막이라고 할 수 있고, 모든 subscription은 subscriber로 끝난다. 일반적으로 방출된 output value나, completion 이벤트로 무언가를 한다. 

![4](/assets/images/Combine/Chapter1/4.png)

Combine은 2개의 내장된 subscriber를 제공해준다. 
- `sink` : 이 subscriber는 **방출된 output value나 completions을 받는 우리가 작성할 수 있는 클로저를 제공**한다. 이 클로저 내부에서 받은 이벤트나 값으로 원하는 것을 수행하면 된다.  
- `assign` : 이 subscriber는 **사용자 지정 코드 없이도 결과 출력을 데이터 모델 또는 UI 컨트롤의 일부 속성에 바인딩**하여 키 경로를 통해 화면에 직접 데이터를 표시할 수 있게 한다.

데이터에 대한 다른 요구사항이 있는 경우에는 publisher를 생성하는 것보다 **사용자 지정 subscriber를 생성하는 것이 훨씬 쉽다.** Combine은 우리가 원하는 작업을 처리하기 위한 도구를 제공하지 않는 경우에 사용자 지정 코드를 작성하여 사용할 수 있도록 간단한 프로토콜 집합을 사용한다. 

### Subscritpions

> 여기서 subscription 용어는 Combine subscription 프로토콜과 이를 따르는 객체뿐만 아니라 publisher, operator, subscriber의 전체 체인을 의미한다. 

`subscriber`를 subscription 마지막에 추가하면, 체인의 맨 처음을 publisher에게 알려주게 된다. 이는 매우 중요한데, `publisher`는 **output을 받는 subscriber가 없는 경우에는 아무런 값도 방출하지 않는다.**  Subscription은 사용자 지정 코드와 에러처리를 **한 번만** 작성하여 비동기적 이벤트 체인을 선언한 뒤, 이후에는 신경쓰지 않아도 된다는 점에서 매우 좋다. 만약 full-Combine을 하게 된다면, subscription들을 통해서 App의 로직을 설명하고, 완료되면 시스템에서 데이터를 푸시하거나 풀링하거나 다른 개체를 호출할 필요 없이 모든 것을 실행하도록 할 수 있다. 

subscription 코드가 성공적으로 컴파일되고 사용자 지정 코드에 문제가 없다면, 이 subscription은 **특정 이벤트(타이머가 꺼지거나, publisher 중 하나를 깨우거나)가 발생할 때 마다 비동기적으로 실행**된다. 

게다가 `Cancellable` 이라는 Combine이 제공하는 프로토콜에 의해서 subscription의 **메모리 관리에 신경쓰지 않아도된다.** 시스템에서 제공하는 subscriber들은 `Cancellable`을 따르기 때문에, **subsciption 코드(publisher, operator, subscriber call chain)는 Cacellable한 객체를 반환**하게 된다. 그리고 해당 객체를 **메모리에서 release할 때 마다 전체 subscription을 취소하고 리소스를 메모리에서 release**한다. 
- 직접 처리하는 방법 : ViewController의 **프로퍼티에 subscription을 저장**하여 수명을 묶을 수 있다. 이 경우 사용자가 view controller를 view stack에서 해제하면, 프로퍼티들도 deinit되고 그럼 subscription도 해제된다. 
- 자동으로 처리하는 방법: 타입 **내부에 [AnyCancellable] 콜렉션 프로퍼티**를 가지고, 원할 때마다 **subscriptions을 throw**하면 된다. 그럼 자동적으로 프로퍼티가 메모리에서 release될 때 취소되고 release된다. 

## Combine의 장점
물론 Combine을 사용하지 않고 여전히 최고의 App을 만들 수 있다. 또한 CoreData, URLSession, UIKit 없이도 가능하다. 그러나 이러한 **framework를 사용하는 것**이 스스로 추상화를 작성하는 것보다 더 **편리하고 안전하며 효율적**이다. 

Combine은 비동기 코드에 또 다른 추상화를 추가하는 것을 목표로 하며, 시스템 레벨의 또 다른 추상화 레벨은 우수한 테스트를 거친 긴말한 통합과 장기간의 지원을 위한 안전한 베팅이다. 또한 아래과 같은 이유로 Combine을 사용하는 것을 고려해보면 좋을 것이다. 
- Combine은 시스템 레벨에 통합되어있다. 즉, Combine 자체는 공개적으로 사용할 수 없는 언어 기능을 사용하여 사용자가 직접 구축할 수 없는 API를 제공한다
- delegate, IBAction, closure를 사용하는 기존 스타일의 비동기식 코드를 사용하면 처리해야하는 모든 케이스에 대해서 사용자 지정 코드를 작성해야한다. 그리고 이러한 것을 테스트하기 위해서는 또 코드가 필요할 것이다. 반면에 Combine은 **모든 비동기 작업을 operator로 추상화하며, 이미 잘 테스트가 되어 있는 것**이다
- 모든 비동기 작업이 **동일한 인터페이스인 publisher를 사용**하면, **구성과 재사용성이 매우 강력**해진다
- Combine operator는 매우 구성되기 쉽다. 즉, 새로운 operator를 생성하면 **새 operator가 즉시 나머지 Combine과 Plug and play**하게 된다. 
- 비동기 코드 테스트는 일반적으로 동기 테스트보다 복잡한데, Combine을 사용하면 비동기 연산자가 이미 테스트되어 있으므로 **비지니스 로직만 테스트**하면 된다. 즉, 일부 입력을 제공하고 예상되는 결과를 출력하는지 테스트하면 된다

위에서 보다시피 대부분 **안전성과 편의성**이라는 혜택을 가지고 있다. 

## App의 구조와 Combine
Combine은 **App의 구조에 따라서 영향을 끼치는 framework가 아니다.** Combine은 비동기적인 데이터 이벤트나 이러한 커뮤니케이션을 통합하게 된다. 즉, 프로젝트에서 책임을 분리하는 방식은 전혀 변경되지 않는다. 따라서 MVC, MVVM, VIPER 등에서 모두 Combine을 사용할 수 있다. 

Combine은 코드에서 **개선하고자 하는 부분에만 사용**할 수 있다. 따라서 기존의 모델을 수정하거나, 아니면 기존 기능을 그대로 유지하면서 App에 새롭게 추가되는 부분에만 Combine을 사용할 수 도 있다. 

그러나 Combine과 SwiftUI를 동시에 채택한다면 다른 이야기이다. 이 경우에는 MVC 구조에서 Controller를 없애야만 한다. View Controller는 Combine/SwiftUI를 사용하는 경우에는 사용할 기회가 없다. 데이터 모델에서 view에 이르기 까지 reactive 프로그래밍을 사용하는 경우에는 단순히 view를 제어하기 위한 특별한 controller가 필요하지 않기 때문이다. 

![5](/assets/images/Combine/Chapter1/5.png)

## 마치며

Combine은 다른 서드파티와의 종속성이 없다. <br> 따라서 앞으로 진행하는 프로젝트는 Xcode나 Playground에서 진행하면 된다. 

요약하면 Combine은 다음과 같은 특징을 가진다. 
- Combine은 시간에 따른 **비동기 이벤트 처리를 위한 선언적이고 reactive한 framework**이다
- 비동기 프로그래밍을 위한 **도구들의 통합, mutable state와 에러 처리에 있어서 기존의 문제점을 해결하는 것을 목표**
- 3가지 주요 유형을 중심으로 사용된다.
    - `publishers` : 시간이 지남에 따라 이벤트를 방출
    - `oeprators` : upstream(input) 이벤트를 비동기적으로 처리 및 조작
    - `subscribers` : 결과를 사용하여 필요한 작업을 수행

이를 통해서 Combine이란 무엇이며, 왜 생겼으며 어떻게 사용하는지에 대한 개념들을 알 수 있었다. <br>Chapter2에서는 Publisher와 Subscriber에 대해서 알아볼 것이다.  

## 참고

[Raywenderlich Combine Chapter 1: Hello, Combine!](https://www.raywenderlich.com/books/combine-asynchronous-programming-with-swift/v2.0/chapters/1-hello-combine)