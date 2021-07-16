---
title:  "[Swift]Size Class"
excerpt: "모든 기기나 상황에 따라 유연하게 대응할 수 있는 방법 중의 하나인 Size class에 대해서 알아보자."

categories:
  - Swift
tags:
  - [Size class, UITraitCollection]
last_modified_at: 2021.02.28
---

# 들어가며
사람들은 일반적으로 자신들이 사용하는 앱이 iPhone 뿐만 아니라 iPad에서 사용할 수 있기를 원한다.
이에 iOS 앱에서는 인터페이스 요소 및 레이아웃을 구성하여 iPad에서 멀티태스킹, split view, 가로 화면, 세로 화면 일때도 다양한 장치에서 모양과 크기를 자동으로 변경하게 앱을 만들어 줄 수가 있다. 
즉, 모든 환경에서 사용자에게 적용가능한 interface를 제공하는 인터페이스를 설계하는 것이 중요하다.

그렇다면 이러한 interface를 설계하려면 어떻게 해야할까? <br>
[해당 문서](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/)에는 여러가지 방법들이 나와있지만 오늘은 그중에서 **Size class**에 대해서 포스팅 할 것이다.

# Size class
Size class는 크기에 따라 content area에 자동적으로 할당되는 특성을 의미한다.
즉, 시스템은 View의 높이와 너비를 설명하는 **regulart, compact**의 두가지 크기 클래스가를 정의한다. 

따라서 View는 다음과 같은 4가지 조합의 크기 클래스를 가질 수 있습니다.
- Regular width, regular height
- Regular width, compact height
- Compact width, regular height
- Compact width, compact height

그리고 iOS는 content area의 크기 클래스를 사용자가 장치를 가로방향에서 세로 방향으로 회전하면 
수직 크기 클리스가 compact → regular로 변경되며 tab bar의 막대가 높아지는 것과 같이 동적으로 layout을 조정하게 됩니다. 

![](https://images.velog.io/images/minni/post/e600d984-ba61-4761-aa60-4ef429fe3a7b/image.png) 

위의 그림이 바로 기기별로 Portrait, Landscape이냐에 따른 width, height를 확인할 수 있다. 


## Size class 적용하기
그렇다면 이러한 Size class에 따른 구현을 어떻게 해야할까?

Interface Builder는 총 9가지 size class를 인식하게 됩니다. <br>
4가지는 위에서도 언급한 경우입니다.
- Regular width, regular height
- Regular width, compact height
- Compact width, regular height
- Compact width, compact height

나머지 5가지는 다음과 같습니다.
- Regular width, any height
- Any width, compact height
- Compact width, any height
- Any width, compact height
- Any Width, any height

Any 클래스가 적용되면 다음과 같습니다. 예를 들어 Compact-Any 인 경우에는 Compact-Compact, 그리고 Compact-Regular 사이즈에서 둘 다 나타나게 됩니다. 

더 구체적인 크기 클래스 안에서 설정하는 것은 일반적인 사이크 클래스를 override 하게 된다. 
또한 9개의 모든 사이크 클래스에 대해 모호하지 않고 적응이 가능한 레이아웃을 제공해야한다.

따라서 일반적으로는 가장 일반적인 크기 클래스에서 가장 구체적인 클래스까지 작업하는 것이 가장 쉽다.
앱의 기본 레이아웃을 선택하고 Any-Any 클래스에서 이 레이아웃을 디자인하는 것이  바로 그 방법이다.
그리고 필요에 따라 다른 기준 또는 최종 크기 클래스를 수정하면 된다. 

![](https://images.velog.io/images/minni/post/f346b408-c6f4-47da-b26b-fc7bbb10ffcb/image.png) 

실제로 Xcode의 storyboard에서 다음과 같이 기기 별로 선택을 할 수 있고, 
그러면 빨간 박스에 나와있는 것처럼 `wC hR` 이라고 기기명 옆에 나오는데 
iPhon 11은 Portrait 상태에서 width = Compact하고 height = Regular 하다는 의미이다. 

이제 실제로 사이크 클래스에 따라서 특성을 다르게 해 볼 예정이다. 

![](https://images.velog.io/images/minni/post/1802a41f-3683-4146-ad46-cfdee26bdfb9/image.png) 

`wC hR`인 기기에서는 뒷배경이 **초록색**, font 색상 **보라색**, font size은 **System 25.0** <br>
`wR hR`인 기기에서는 뒷배경이 **회색**, font 색상 **주황색**, font size는 **Large Title**로 설정해보자.

Storyboard에서 inspector를 사용해서 사이즈 클래스마다 다르게 설정해 줄 수 있다. <br>
먼저 화면의 배경부터 적용해보자! 

![](https://images.velog.io/images/minni/post/9d846516-1ba7-429d-aee6-aeb09beea458/image.png) 

Background 왼쪽에 + 버튼을 누르면 다음과 같이 Width, Height, Camut, Idiom을 설정할 수 있는데
`wC hR` 일때는 Green, `wR hR` 일때는 Gray로 설정해주면 된다. 

다음으로는 라벨을 하나 올리고 글씨를 작성하고 난 다음에 Label inspector로 가서 배경과 똑같은 방법으로 지정해주면 된다. 

![](https://images.velog.io/images/minni/post/b400191a-bace-4771-90d8-773526f5a262/image.png) 

이런식으로 추가를 해주고 나서 아래에서 기기를 변경해보면서 배경의 색상, 글자색, 폰트 크기가 변경되는 것을 확인해보면 된다. 


![](https://images.velog.io/images/minni/post/e9d55013-5159-4ad0-b4a7-5a2bb9e15725/image.png) 

> 이때 해당 width, height에서 font, background 등등의 설정들을 바로 하려면 위의 그림에서 해당 width, height를 가지고 있는 기기를 고르고 오른쪽에 있는 **Vary for Traits**를 누른 뒤에 설정들을 하게되면 해당 width, height에서만 설정을 할 수 있다. 


## Size class 코드에서 확인하기
그렇다면 이러한 Size class를 코드상에서는 어떻게 확인할 수 있을까?

바로 `UITraitCollection` 클래스를 사용하면 된다. <br>
UITraitCollection을 사용하여 horiziontal, vertical size class나 display scale, user interface idiom 등을 파악할 수 있다. 

```swift
// horizontal, vertical SizeClass의 종류에 따라 분기 작성하기
if UITraitCollection.current.horizontalSizeClass == .compact {
    print("가로가 compact인 경우 📱(e.g. 아이폰XR)")
}
else if UITraitCollection.current.verticalSizeClass == .regular {
    print("세로가 regular인 경우 🖥(e.g. 아이패드Air)")
}
```
위의 코드처럼 UITraitCollection.current에 접근해서 horizontal, vertical의 사이즈 클래스의 종류에 따라서 다르게 행동할 수 있도록 코드를 작성할 수도 있다. 

또한 기기에 따라서도 아래와 같이 작성할 수 있다. 
```swift
// user interface idiom에 따른 분류
if UITraitCollection.current.userInterfaceIdiom == .phone {
    print("기기가 📱(폰)인 경우")
}
else if UITraitCollection.current.userInterfaceIdiom == .pad {
    print("기기가 🖥(아이패드)인 경우")
}
```
그리고 `traitCollectionDidChange(_:)` 를 override하여 iOS interface 환경이 변경되었을 때 
적절한 대응을 할 수 있도록 구현할 수 있다. 
```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
   super.traitCollectionDidChange(previousTraitCollection)
   // iPhone이 portrait → landscape로 가거나 landscape → portrait으로 변경되는 경우
   if ((self.traitCollection.verticalSizeClass!= previousTraitCollection!.verticalSizeClass) || (self.traitCollection.horizontalSizeClass != previousTraitCollection!.horizontalSizeClass)) {
       print("🙋 Device의 traitCollection이 변경됨")
    }
}
```
 
이런식으로 코드에서 SizeClass나 기기나 Portrait, Landscape에 따라서 코드를 작성해 볼 수 있다. 


# 참고
1. [Human Interface GuideLines](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/) <br>
2. [UITraitCollection Apple Document](https://developer.apple.com/documentation/uikit/uitraitcollection) <br>
3. [traitCollectionDidChange(\_:) Apple Document](https://developer.apple.com/documentation/uikit/uitraitenvironment/1623516-traitcollectiondidchange) <br>