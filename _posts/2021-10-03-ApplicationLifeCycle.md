---
title:  "[iOS] Application life cycle"
excerpt: "Application의 생명주기에 대해서 알아보자"

categories:
  - iOS
tags:
  - [UIApplication, UIApplicationMain, App Launch Sequence]
last_modified_at: 2021.10.03
--- 

## 들어가며
Application의 생명주기는 간단히 말해서 **Application이 실행되고 종료 될 때까지의 흐름**이라고 할 수 있다.
먼저 앱이 실행되었을 때 일어나는 일들은 다음과 같다. 

1. `main` 함수 실행됨
2. `main` 함수는 `UIApplicationMain` 함수를 호출
3. `UIApplicationMain` 함수는 앱의 본체에 해당하는 객체인 **`UIApplication` 객체 생성** 
4. nib 파일을 사용하는 경우나, Info.plist 파일을 읽어들여 파일에 기록된 정보를 참고하여 그 외 필요한 데이터 load
5. AppDelegate 객체를 만들고 App 객체와 연결하고 run loop를 만드는 등 실행에 필요한 준비를 한다
6. 실행 완료를 앞두고 App 객체가 AppDelegate에게 `application:didFinishLaunchingWithOptions:` 메시지를 보낸다

이때 개발자인 우리가 사실상 관여하는 부분은 4번부터라고 할 수 있고, 앱 실행과 동시에 동작해야하는 함수들은 보통 6번의 application `application:didFinishLaunchingWithOptions:` 에 작성을 하여 동작시킨다.

Application Life cycle을 알아보기 전에 App 구조와 main run loop에 대해서 먼저 알아보자. 

## Structure of App

iOS는 기본적으로 MVC(Model-View-Controller) 구조를 사용하는데, 이 디자인 패턴은 app의 data와 비지니스 로직을 UI요소로부터 분리시키는 역할을 한다. 그리고 앱이 시작되면서 `UIApplicationMain` 함수는 몇 가지 중요한 객체를 생성하고 앱을 실행시키는데, 이때 `UIApplication` 은 모든 iOS의 중심에서 시스템과 앱의 여러 객체들간의 상호작용을 가능하게 해준다. 

<img src="/assets/images/ApplicationLifeCycle/1.png" style="zoom:50%;" />

## Main Run Loop

Main run loop는 이벤트들을 받은 순서대로 처리한다. 

`UIApplication` 객체는 앱이 실행될 때, **Main run loop를 실행**하고 해당 run loop로 **이벤트를 처리**하게 된다. 사용자가 디바이스에서 특정 액션을 취하면, 그 **액션에 해당하는 이벤트가 시스템에 의해 생성**되어 UIKit에서 생성한 port를 통해 앱에 전달된다. 그리고 **전달된 이벤트들은 queue에 하나씩 보관**되었다가 **main run loop로 전달되어서 처리**가 된다. 

(해당 과정은 당연히 **mainThread에서 동작**)

<img src="/assets/images/ApplicationLifeCycle/2.png" style="zoom:50%;" />

## Application Lauch Sequence

### step1

<img src="/assets/images/ApplicationLifeCycle/3.png" style="zoom:50%;" />

단계 1에서는 먼저 사용자가 실행할 앱 아이콘을 터치하여 실행시킨다.

이후 AppDelegate에서 `@main` 어노테이션을 찾아서 해당 클래스를 실행하게 된다. 

이후 main함수의 실행 과정은 위의 4가지 단계이다. 해당 과정들을 `@main` 이 한 번에 처리해주고 있는 것이다. 



#### Main Function

Objective-C는 C 언어 기반 프로그램으로서 시작점이 main 함수이지만, Swift는 C언어 기반이 아니기 때문에 main 파일이나, 시작포인트가 존재하지 않는다. 그러나 시작 진입점이 없으면 안되기 때문에 Swift는 `어노테이션(@) `표기를 사용해서 이를 대체한다.

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
	//..
}
```

보다시피 AppDelegate안에 `@main` 이라는 어노테이션이 존재한다. (iOS 12이전에서는 `@UIApplication` )

iOS 앱에서는 **UIKit framework가 main 함수를 관리**하여 앱 개발자들이 직접 main함수를 작성하지 않는다는 것이다. 그렇다고 개발자들이 아예 앱의 실행에 관여하지 못하는 것은 아니다. UIKit은 main함수를 다루는 과정에서 `UIApplication` 객체가 생성되는데, **이 객체를 통해서 앱의 실행에 부분적으로 관여**할 수 있다.

main 함수 내부에서는 `UIApplicationMain()` 함수를 실행하게 된다. 

#### UIApplicationMain Function

`UIApplicationMain` 함수는 **코코아 터치 프레임워크에서 앱의 라이프 사이클을 시작하는 함수**이다. `UIApplication` **객체의 인스턴스를 만들고, 해당 객체의 앱으로서 기능하기 위한 기반을 마련**하는데 이 과정을 **앱 로딩 프로세스**라고 한다. 

Objective-c에서는 main.m 파일이 기본적으로 생성되어 있다

```c
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
 
int main(int argc, char * argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

그러나 Swift에서는 Appdelegate의 @main 어노테이션을 지우고 직접 main.swift를 만들어 main을 다룰 수 있다. 

```swift
// Swift에서 UIApplicationMain
UIKIT_EXTERN int UIApplicationMain(int argc, 
                                   char * _Nullable argv[_Nonnull], 
                                   NSString * _Nullable principalClassName, 
                                   NSString * _Nullable delegateClassName);
```

UIApplicationMain을 호출하면 아래와 같이 4개의 매개변수를 넘겨줘야한다.

<img src="/assets/images/ApplicationLifeCycle/4.png"/>

- `argc` 
  - argv의 개수 (실행 명령어의 개수)
- `argv` 
  - argument의 변수 목록
- `principalClassName`
  - UIApplication 클래스를 서브클래싱 한 경우 해당 클래스 이름을 전달하면된다
  - nil을 전달하면 이 값은 UIApplication으로 고정된다
- `delegateClassName` 
  - App delegate 클래스 이름을 전달하면된다
  - 만약 nib 파일 내 app delegate 객체가 정의되어있다면 nil을 전달해야한다

즉, main 함수안에서 실행되는 `UIApplicationMain` 함수는 앱에 중요한 객체인 `UIApplication`을 **생성하고, 스토리보드에서 UI를 로딩하고 앱의 초기 셋팅값(info.plist)를 불러오고 앱을 run loop에 올려놓으며 함수를 진행**시킨다. 

### step2

<img src="/assets/images/ApplicationLifeCycle/5.png" style="zoom:50%;" />

이때는 그림에서 오른쪽 상자에 Your Code라고 나와있는 부분을 보면 된다. 이러한 메서드 내부에서 **커스텀 코드를 작성하여 앱이 launch process를 시작하고 끝내는 동안 이벤트를 설정**할 수 있다.

`application:willFinishLaunchingWithOptions` 

- App 최초 실행될 때 호출되는 메서드
- AppDelegate에게 현재 launch process가 시작되었다고 알려준다
- 그러나 아직 state restoration은 진행되지 않은 상태이다

<br>

`app state restoration` 

- App 실행 시 UIKit은 앱을 이전 상태로 복원을 시도한다
- 이때 UIKit은 App에서 유저 인터페이스를 보존한 뷰 컨트롤러 객체를 생성 또는 locate하도록 App에 요청한다
- 이후 아래와 같은 로직을 수행하고 `application:didFinishLaunchingWithOptions` 를 호출한다
- 자세한 내용은 [이 글](https://medium.com/swift-india/app-state-restoration-in-ios-1-bbc903f17a46)을 참고하면 된다. <br>
  <img src="/assets/images/ApplicationLifeCycle/6.png" style="zoom:50%;" />

<br>

`application:didFinishLaunchingWithOptions` 

- AppDelegate에게 현재 launch process가 거의 끝났고 앱이 거의 시작하기 전이라고 알려준다
- state restoration 직후에 호출되지만, 아직 App의 window나 UI는 표시되기 전이다

### step3

<img src="/assets/images/ApplicationLifeCycle/7.png" style="zoom:50%;" />

이제 app launch가 끝나고 실제로 running에 들어왔다. <br>
앱이 실행되고 사용자에게 화면이 보여진다.

`applicationDidBecomeActive:` 

- App이 active 상태로 전환된 직후에 호출이 된다

`Event Loop` 

- App이 작동하면서 앱을 만들때 개발자가 작성했던 코드들이 실행된다

⚠️ 위의 그림은 iOS 13이전에 즉, Scene의 개념이 등장하기 전에 AppDelegate에 의해서 Application Life cycle이 관리되었을 때의 그림이다. 

<img src="/assets/images/ApplicationLifeCycle/8.png" />

따라서 iOS 13이후에는 이제 step2 단계(initailization)이 끝나면 위의 그림처럼 system이 scene delegate나 app delegate를 사용해서 UI를 표시하고, app의 life cycle을 관리하게 된다. 이후의 내용, 즉 scene delegate와 app delegate에 의한 이후 life cycle 관리는 [App Delegate && SceneDelegate의 역할]({{site.url}}{{site.baseurl}}ios/AppDelegate&SceneDelegate/)를 보면된다. 


## App Execution State

App은 총 5가지 실행 상태를 가지고 있다. 그리고 상태의 전환은 AppDelegate 객체의 메소드 호출을 통해서 이뤄진다.

`AppDelegate` 객체는 UIResponder, UIApplicationDelegate를 상속 및 델리게이트 참조하고 있고, `UIApplicationDelegate` 는 UIApplication 객체의 작업에 개발자가 접근할 수 있도록 하는 메소드를 담고 있다. 

<img src="/assets/images/ApplicationLifeCycle/9.png" style="zoom:50%;" />

`NotRunning`

- 실행되지 않았거나, 시스템에 의해 종료된 상태
- 앱이 동작중이지 않다

`Inactive`

- 앱 실행중이지만, 이벤트를 수신할 수 없다 
- Not running과 Active 사이의 상태
- 예를 들어, 앱 실행 중 미리 알림 또는 일정 alert이 화면에 덮여서 앱이 실질적으로 이벤트는 받지 못하는 상태 등을 의미

`Active`

- 앱이 실질적으로 활동하고 있는 상태
- 이벤트를 수신할 수 있다

`Background`

- 백그라운드 상태에서 실질적인 동작을 하고 있는 상태
- 예를 들어 백그라운드에서 음악을 실행하거나, 걸어온 길을 트래킹, 앱을 사용하다가 홈화면으로 switch 하는 등의 상태를 의미

`Suspended`

- 백그라운드 상태에서 활동을 멈춘 상태
- 빠른 재실행을 위해 **메모리에 적재된 상태**지만, 실질적으로 **동작하고 있지는 않는 상태**
- **메모리가 부족할 때 비로소 시스템이 강제 종료**를 하게 된다

<br>

💡 **Foreground** 상태 : `inactive` , `active` 상태에 있는 경우 <br>
💡 **Background** 상태 : `background` 상태에 있는 경우 


#### ⚠️ **Key point**
앱이 background 상태에서 foreground 상태로 전환 될 때 **바로 active 상태로 가지 않는 점** <br>
**background 상태에서도 코드 실행이 가능**하다는 것을 명심해야한다. 


### 마치며
Application Life cycle에 대해서 알아볼 수 있었으며, iOS 12이전과 이후의 차이점에 대해서 좀 더 자세하게 살펴볼 수 있었던 것 같다. 이 글에서는 App launch sequence에 대해서 전반적으로 다뤘기 때문에 Application life cycle에 대해서 전체적으로 확인해보려면 [App Delegate && SceneDelegate의 역할]({{site.url}}{{site.baseurl}}ios/AppDelegate&SceneDelegate/)도 읽는 것이 좋다. 

### 참고 
[Apple document About the App Launch Sequence](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app/about_the_app_launch_sequence) <br>
[App restoration](https://medium.com/swift-india/app-state-restoration-in-ios-1-bbc903f17a46) <br>
[App의 state restoration](https://devmjun.github.io/archive/Restorazation-2) <br>
[앱의 생명주기 INSWAG](https://atelier-chez-moi.tistory.com/29) <br>