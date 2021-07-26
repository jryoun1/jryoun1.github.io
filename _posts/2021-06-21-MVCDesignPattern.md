---
title:  "[Design Pattern] MVC"
excerpt: "MVC(Model View Controller) 패턴에 대해서 알아보자"

categories:
  - Design Pattern
tags:
  - [MVC]
last_modified_at: 2021.06.21
---

## MVC 패턴이란

### MVC 패턴의 각 객체의 역할

![MVC](/assets/images/MVC/MVC.png)

UIKit app의 구조는 MVC 디자인 패턴을 기반으로 되어있다. <br> 
여기서 MVC는 `Model, View, Controller` 를 의미하며 각각의 객체는 목적에 따라서 구분된다.  

- `Model` 객체는 app의 데이터와 비지니스 로직을 관리 
- `View` 객체는 데이터를 보여주거나 UI를 담당
- `Controller` 객체는 Model 과 View 사이의 다리 역할을 하며 View로부터 사용자의 action을 받아 Model에게 어떠한 작업을 해야하는지 알려주거나, Model의 변화를 View에게 전달하여 업데이트하게 중재

### 기존의 MVC 패턴

![img](https://miro.medium.com/max/868/1*E9A5fOrSr0yVmc7Kly5C6A.png)

우리가 흔히 아는 MVC 패턴은 위의 그림과 같다. 

`View` 는 **stateless**, 즉 상태를 가지지 않는다. <br>
그리고 View는 단지 Model이 변화하면 Controller에 의해서 바뀌게된다. 

예를 들어서, 웹페이지에서 다른 페이지로 이동하기 위해서 link를 클릭하면 전체가 reload 되는 것을 생각하면 이러한 기존의 MVC 패턴은 iOS application에서 적용하기에는 효율적이지 못하다. 왜냐하면 View, Controller, Model 객체들의 결합도가 높으며 각각의 객체가 서로를 알아야만 한다. 

따라서 이러한 구조는 재사용성을 낮추고 이렇기 때문에 iOS application에서는 사용하기에 적합하지 않다. 

### Apple의 기대 MVC 패턴

![img](https://miro.medium.com/max/870/1*c0aGaDNX41qu6e8E4OEgwQ.png)

따라서 Apple에서 제안하는 MVC 패턴은 다음과 위와 같은 구조를 가지며 아래와 같이 동작하기를 기대한다. 

먼저 `Controller`는 `View`와 `Model`을 중재하며, 따라서 `View`와 `Model`은 서로를 알지 않아도 된다. `Controller`는 재사용하기는 어렵지만, `Model`에 적합하지 않는 비니지스 로직을 모두 수용할 공간이 있어야하기 때문에 위와 같은 패턴은 좋다. 

이론적으로는 MVC 패턴은 완벽해보이지만, 실제로는 기대와 다르게 Massive ViewController가 생성되는 문제가 발생하게 된다. 그리고 iOS 개발자들 사이에서는 View controller의 offloading, 즉 일들을 떠 넘기는 것은 논쟁거리가 되었다. <br>
그렇다면 기존의 MVC pattern을 Apple에서 개선했지만 왜 이런 문제가 발생하는 것일까?

### Apple의 현실 MVC 패턴

#### 문제점
분명이 기존의 MVC pattern을 Apple에서 개선했지만 다음과 같은 문제들이 발생한다. 

![img](https://miro.medium.com/max/1150/1*PkWjDU0jqGJOB972cMsrnA.png)

#####  Massive View Controller

실제로 Cocoa MVC Pattern은 Massive ViewController를 작성할 수 밖에 없다. 

왜냐하면 위의 그림과 같이 View와 Controller가 매우 밀집하게 연결되어 있고, Controller는 View Life Cycle 까지 관리하기 때문에 View와 Controller를 분리하기가 어려워진다. 

또한 Model, View에 들어가기 애매한 코드들(e.g. delegate, datasource, dateformatter, 네트워크 요청, DB 데이터 요청)은 모두 Controller에 들어가게 된다. 이렇게 되면 Controller는 매우 비대해지게 되고, 이렇게 바로 Massive ViewController가 되는 것이다. 

```swift
var userCell = tableView.dequeueReusableCellWithIdentifier("identifier") as UserCell
userCell.configureWithUser(user)
```

우리는 실제로 위와 같은 코드를 Controller에서 tableview를 사용할 때 엄청나게 많이 보게 된다. 여기서 Cell은 configureWithUser()라는 함수를 통해서 Model을 직접적으로 configure하는 view이다. 이것은 MVC 패턴을 위반하는 것이지만 실제로 이러한 코드를 많이 사용한다. 

만약 MVC 패턴을 엄격하게 따른다면, controller에서 cell을 구성하고 Model을 View에 전달하지 않아야한다. 그러나 말한 것처럼 구현하게 되면 controller는 점점 더 비대해지게 된다. 이렇게 view controller가 비대해지는 것을 조금이나마 막기 위해서 MVC 패턴을 위반하면서 위와 같이 코드를 작성하게 된다. 

##### Unit Test

또한 Unit Test 측면에서 보면, `Model`의 경우에는 따로 분리가 되어있어 Unit Test가 가능하지만, `View`와 `Controller`는 매우 강하게 결합되어 있기 때문에 Unit Test를 진행하기 어렵다. 

## MVC 패턴에서의 소통방법

Controller의 경우에는 직접적으로 View와 Model과 소통하며 지시할 수 있는 반면에 Model과 View는 Controller에게 직접적으로 알릴 수 없다. 

### View to Controller

![VtoC](/assets/images/MVC//viewToContorller.png)

`Controller`는 `View`에서 **발생할 수 있는 action에 대해 target을 생성**한다. 
그리고 `View`에서 사용자에 의해서 action이 발생하면 Controller의 target이 이를 감지하고 작업을 수행한다. 

또한 `View`는 delegate pattern의 **delegate와 datasource를 이용해 Controller에게 어떤 작업을 수행해야하는지 알릴 수도 있다.**
예로 UITableView의 UITableViewDelegate, UITableViewDatasource가 있다.

### Model to Controller

![CtoV](/assets/images/MVC/controllerToView.png)

`Model`은 **Observer pattern의 Notification과 KVO(Key Value Observation)을 통해서** `Controller`에게 알려준다. 
이에 Notification과 KVO는 일을 수행하는 객체가 진행하던 작업이 끝나면 자신들을 구독 중인 객체들에게 신호를 보낸다. 

즉, `Model`에서 작업이 완료되었을 때, `Controller`에게 신호를 보내게 된다. 


## MVC 패턴의 장점 및 단점

### 장점

- 다른 패턴에 비해서 코드량이 적음 
- 모두에게 친숙하기 때문에 직접 코드를 작성하지 않은 개발자도 쉽게 유지보수 할 수 있음
- 역할 분담을 고려한 구조를 빠르게 구현 할 수 있음 

### 단점

- Apple 의 현실 MVC 패턴에서 얘기했듯이 Massive view controller가 작성됨
- Model을 제외하고 View와 Controller는 Unit Test를 수행하기 어려움

## 마치며

위에서 언급했던 MVC 패턴의 단점을 해결하기 위해서 아래와 같은 방법들이 있다는데 추후에 공부하고 포스팅 할 예정이다. 
- [Model controller in Swift](https://www.swiftbysundell.com/articles/model-controllers-in-swift/) <br>
Model만 다루는 Model Controller를 따로 만들어 사용하는 방법
- [lighter view controllers](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/) <br>
Data Source 와 다른 protocol을 분리해내는 방법

## 참고 문헌

[Apple - UIKit about app development with UIKit](https://developer.apple.com/documentation/uikit/about_app_development_with_uikit) <br>
[medium - iOS architecture patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52) <br>
[Model controller in Swift](https://www.swiftbysundell.com/articles/model-controllers-in-swift/) <br>
[lighter view controllers](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/) <br>
[zooneon MVC pattern에 대해서 알아보자](https://velog.io/@zooneon/iOS-MVC-%ED%8C%A8%ED%84%B4%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)

