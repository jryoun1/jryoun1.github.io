---
title:  "[Swift] Dynamic Type"
excerpt: "iOS에서 Dynamic Type은 사용자들의 Accessibility를 증가시켜 줄 수 있는 부분으로서 알아보자."

categories:
  - Swift
tags:
  - [Autolayout, HIG]
toc: true
toc_sticky: true
last_modified_at: 2020.12.30
---

## 들어가며
프로젝트를 진행하던 와중에 모든 view에 Dynamic Type을 적용하는 조건이 있었다. 
처음에는 Dynamic Type이라는게 단어 그대로 동적인 화면을 만들라는 것인 줄 알았다. 
그래서 TableView의 cell이나 Scroll View가 dynamic하게 사이즈가 커지고 줄어드는 것을 의미하는 줄 알았다. 그러나 Dynamic Type in iOS로 검색을 해보니 [여기](https://www.ioscreator.com/tutorials/dynamic-types-ios-tutorial) 페이지에서 이것이 무엇을 의미하는지 알게 되었다. <br>
같이 알아보도록 하자!!

## Dynamic Type 이란?
먼저 Apple의 공식 문서 중에서 H.I.G(Human Interface Guidelines)에서 Visual Design 파트에서 Typograpy부분을 보면 다음과 같은 내용이 있다. 

![](https://images.velog.io/images/minni/post/0c963980-cafd-491a-9874-20d39c053d21/image.png) 

Dynamic Tpye은 사용자가 원하는 글씨 사이즈로 App의 contents를 볼 수 있는 적응성(flexibility)을 제공한다고 나와있다. 이게 무슨 말인가 하면 예를 들어서 jake의 할아버지는 iPhone을 사용하시는데 눈이 침침하셔서 작은 글씨는 돋보기 안경을 써도 확인하지 못 하신다. 

![](https://images.velog.io/images/minni/post/5e3db579-0ddb-479d-ac8b-af27267b31a9/image.png) 

그래서 휴대폰을 사용할 때 위에 화면 처럼 설정에서 글씨 크기를 최대로 해서 사용하신다. 그런데 만약 앱에서 Dynamic Type으로 설정을 하지 않았다면 어떻게 될까? 

말로만 하면 무슨 의미인지 잘 안오니까 직접 확인해보자.
확인하기 전에 먼저 Accessibility Inspector에 알아보자!!

### Accessibility Inspector 란?
Accessibility Inspector는 말 그대로 접근성을 검사해 볼 수 있는 tool이다. 
여기서는 앱의 font의 사이즈를 변경해 볼 수도있고, 시각장애인들을 위한 VoiceOver를 테스트 해 볼 수도 있다. <br>
Xcode에서 실행시키는 방법은 아래와 같다. 
![](https://images.velog.io/images/minni/post/4bf72711-59e5-49fc-842f-90efa6615da7/image.png) 

Xcode → Open Developer Tool → Accessibility Insepctor을 누르면 아래와 같은 창이 생긴다.

![](https://images.velog.io/images/minni/post/a6006738-b5e7-4b8f-a9da-d5206875b864/image.png) 

이때 프로젝트를 실행시키면 Simulator를 설정할 수 있고 알맞은 Simulator로 설정해주면 된다. <br>
지금은 font size의 변경에 대해서 테스트 해 볼 것이므로 아래의 그림의 빨간 박스부분을 눌러주면 된다. 

![](https://images.velog.io/images/minni/post/34c0ab28-b5c5-4192-a8e3-231b99d57e65/image.png) 

그럼 위의 그림처럼 Font size를 조절할 수 있는 slider를 볼 수 있다. 

### Dynamic Type 의 필요성
이제 Dynamic Type이 적용된 UILabel과 그렇지 않은 UILabel의 차이를 확인해 보자.

![](https://images.velog.io/images/minni/post/5cf0a5d9-ad01-4d1d-86d0-7e848083fa92/image.png) 

먼저 사진 속 iPhone에서 빨간 박스는 Dynamic Type이 적용되지 않은 UILabel이고 
파란 박스는 Dynamic Type이 적용된 곳이다. 그리고 왼쪽은 Font size를 중간으로 해놓았고 오른쪽은 Font size를 최대로 설정해 두었다. 

여기서 차이를 볼 수 있는데 Dynamic Type이 설정되어 있지 않은 UILabel의 경우에는 Font size가 커져도 글자의 사이즈가 그대로인 반면에 Dynamic Type이 적용된 UILabel은 글씨가 커진 것을 확인할 수 있다. 

**즉, Dynamic Type이 적용되지 않은 앱에서는 사용자가 설정에서 글씨의 크기를 키워도 App contents의 크기는 커지지 않게 된다.** 따라서 Dynamic Type을 적용해주는 HIG를 중요시 여기는 Apple의 관점에서는 매우 중요하다. 또한 **모든 사용자를 배려하는 앱을 만들기 위해서는 필수적으로 적용**시켜야 할 것 같다는 생각이 든다. 

그럼 이제 실제로 코드나 InterfaceBuilder에서 어떠한 형태로 적용시키는지 알아보자.

## Dynamic Type 적용하기
Dynamic Type을 적용하는 방법은 code를 통해서 적용하거나 Interface Builder를 통해서 적용할 수 있다. 

### Interface Builder를 통해서 적용하기
![](https://images.velog.io/images/minni/post/5b43b186-3e90-412d-aac4-d4d1157ba306/image.png) 

Interface Builder를 통해서 Dynamic Type을 정의하는 방법은 위의 그림처럼 Dynamic Type을 적용할 UILabel을 선택하고 Font에서 빨간 박스를 누르면 default는 **System**으로 설정되어 있을 것이다. 

이때 Text Styles에 있는 Font를 골라주고 Dynamic Type에 check를 하면된다. 
이렇게 UILabel의 경우에는 Dynamic Type을 적용해 줄 수 있지만 button안에 있는 글자들은 interface builder에서 font는 설정해 줄 수 있지만 Dynamic Type을 적용해 줄 수는 없다. 이 부분은 코드로 작성해주어야한다. 

### Code로 적용하기
코드로 적용하는 방법은 다음과 같다. 

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        label.font = UIFont.preferredFont(forTextStyle: .title3)
        label.adjustsFontForContentSizeCategory = true
    }
}
```

먼저 label의 font를 `prefferedFont()` 를 통해서 정해주고 
이후에 `adjustsFontForContentSizeCategory = true` 로 설정하는 부분은 위의 Interface Builder에서는 Automatically adjusts Font에 check를 해주는 부분이다. 

그렇다면 이제 button안의 글씨에 Dynamic Type을 적용해보자.

```swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var button: UIButton!
    @IBOutlet weak var label: UILabel!
    override func viewDidLoad() {
        super.viewDidLoad()
        
        label.font = UIFont.preferredFont(forTextStyle: .title3)
        label.adjustsFontForContentSizeCategory = true
        
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(adjustButtonDynamicType),
                                               name: UIContentSizeCategory.didChangeNotification,
                                               object: nil)
    }
    
    @objc func adjustButtonDynamicType() {
        button.titleLabel?.adjustsFontForContentSizeCategory = true
    }
}
```

위의 코드에서 button을 추가해주었다. 그리고 NotificationCenter를 사용해서 `UIContentSizeCategory` 의 값이 바뀌게 되면 `adjustButtonDynamicType()` 함수를 실행주도록 한다. <br>
이때 `adjustButtonDynamicType()` 함수 안에서는 `adjustsFontForContentSizeCategory = true` 로 설정해주게 되면 button 안의 글자들에도 Dynamic Type을 적용할 수 있다. 

### Code로 작성할 때 유용한 팁
코드를 통해서 Dynamic Type을 설정해주는 경우에는 만약 ViewController가 여러 개인 경우에는 모든 ViewController 에서 다 설정 해줘야하는 경우 이를 protocol로 만들어주어서 Dynamic Type을 보장해 주는 방법이 있다. <br>
이는 코드 리뷰에서 **delmaSong**이 제안해주었다. 고맙습니다 **delmaSong**👍🏻💯

```swift
// protocol을 이용한 방법
import Foundation
@objc protocol DynamicTypeable {
    func setLabelFontStyle()
    @objc optional func adjustButtonDynamicType()
}

// ViewController 
extension ViewController: DynamicTypeable {
    func setLabelFontStyle() {
        titleLabel.font = UIFont.preferredFont(forTextStyle: .headline)
        visitorsLabel.font = UIFont.preferredFont(forTextStyle: .body)
        durationLabel.font = UIFont.preferredFont(forTextStyle: .body)
        locationLabel.font = UIFont.preferredFont(forTextStyle: .body)
        descriptionLabel.font = UIFont.preferredFont(forTextStyle: .body)
        goKoreaExpoListButton?.titleLabel?.font = UIFont.preferredFont(forTextStyle: .body)
    }
    
    @objc func adjustButtonDynamicType() {
        button.titleLabel?.adjustsFontForContentSizeCategory = true
    }
}
```

이런식으로 protocol로 구현하고 이를 채택하도록 한다면 까먹지 않고 적용할 수 있을 것 같다. <br>
이때 protocol을 `optional` 타입으로 구현하였는데 이는 button의 경우에는 해당 view에 있는 경우가 있고 그렇지 않은 경우가 있기 때문에 `optional` 타입으로 구현하였다. 

## 참고
[Autolayout - Dynamic Type 야곰](https://yagom.net/courses/autolayout/lessons/dynamic-type/) <br>
[Dynamic Type in iOS](https://www.ioscreator.com/tutorials/dynamic-types-ios-tutorial) <br>
[HIG Dynamic Type](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/typography/) <br>

