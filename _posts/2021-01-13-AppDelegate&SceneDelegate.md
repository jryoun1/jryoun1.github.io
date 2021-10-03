---
title:  "[iOS] iOS 13이후 SceneDelegate와 AppDelegate의 역할"
excerpt: "SceneDelegate, AppDelegate, App Life Cycle, Scene Life Cycle에 대해서 알아보자."

categories:
  - iOS
tags:
  - [Scene Life Cyle, App Life Cycle, UIApplicationDelegate, UISceneDelegate]
toc: true
toc_sticky: true
last_modified_at: 2021.01.13
---

## 들어가며
앱이 background에 있는가, foreground에 있는가에 따라서 <br>
시스템 알림에 응답하고 다른 중요한 시스템 관련된 이벤트를 처리하게 된다. 

현재 **app의 state(상태)** 는 **어떠한 것을 할 수 있고 없는지를 결정**하게 된다. <br>
예를 들어 **foreground 앱**은 user가 사용하고 있기 때문에 user의 관심을 받게되고 <br>
따라서 CPU를 포함한 system resource에서 우선순위를 가지게 된다. <br>
반면 **background 앱**은 화면 밖의 일이기 때문에 가능한 작업을 수행하지 않는 것이 좋다. <br>
따라서 우리의 앱은 상태가 바뀔 때 마다 이에 대응해서 동작을 조정해야한다. 

앱의 상태가 바뀔 때 UIKit은 적절한 delegate object의 method를 호출하여 우리에게 알려준다. 
- iOS 13 이상은 **UISceneDelegate 개체를 사용하여 scene-based app에서 발생하는 life-cycle event에 응답**하면 된다.
- iOS 12 이하는 **life-cycle event에 응답하기 위해서 UIApplicationDelegate 개체를 사용**하면 된다. 

## AppDelegate 와 SceneDelegate
[AppDelegate와 SceneDelegate의 차이]({{site.url}}{{site.baseurl}}/wwdc/ArchitectingYourAppForMultipleWindows/)에 대한 내용은 해당 글에서 확인할 수 있다. 

먼저 AppDelegate, SceneDelegate에 대해서 알아보자. <br>
iOS13을 기준으로 SceneDelegate가 생성이 되었다고 하는데 아래의 그림을 통해 확인해보자. <br>

- iOS12 이하
![](https://images.velog.io/images/minni/post/1cd406dd-ff64-4adb-a887-0ddfc802e169/image.png) 

- iOS13 이상 
![](https://images.velog.io/images/minni/post/ba22c260-c43d-4f8b-a6ff-466377b07af6/image.png)

### iOS 13 이후 바뀐점

1. iOS 12까지는 **하나의 앱에 하나의 window(창)** 즉, 한 앱을 동시에 키는 것이 불가능하였지만 
iOS 13부터는 **window의 개념이 scene으로 대체되고 아래의 그림처럼 하나의 앱에서 여러 개의 scene을 가질 수 있게 되었다. 즉, 하나의 앱을 동시에 켜는 것이 가능해졌다.** 

![](https://images.velog.io/images/minni/post/0f1cda16-d4b1-49f3-a7cb-53fe385d0e27/image.png)

2. AppDelegate의 역할 중 **UI의 상태를 알 수 있는 UILifeCycle**에 대한 부분을 
**SceneDelegate가 대체**한다. 

![](https://images.velog.io/images/minni/post/73ea4a37-ad07-4716-8bba-1b8c6fcf8d3a/image.png)

3. AppDelegate의 역할에 **Session LifeCycle이 추가** 되었다.

![](https://images.velog.io/images/minni/post/ceeec60d-eaed-45e4-be73-52723272753c/image.png) 

위의 사진을 보면 **SceneSession이 생성되거나 삭제될 때 AppDelegate에게 알려주는 
두 가지의 메소드가 AppDelegate에 추가**된 것을 확인해 볼 수 있다.
**SceneSession은 앱에서 생성한 모든 scene의 정보를 관리**한다. 

### Scene이란?

#### Scene 개념이 생긴 이유는 무엇일까?
Scene이라는 개념은 iOS 13이후에 생긴 개념이다. <br>
위에서 보았듯이 SceneDelegate는 AppDelegate의 UILifeCycle 역할을 가져갔다. 

그렇다면 Apple은 scene이라는 개념을 단지 AppDelegate의 역할을 분리해 구조적으로 변화를 주어서 SceneDelegate와 AppDelegate가 **각각 자신의 역할을 잘 할 수 있도록 하기 위해서** scene이라는 개념을 만들었을까? <br>
아니면 구글링했을 때 가장 많은 이유로 등장하는 **Multiple Window를 지원**하기 위해서 등장하였을까?

같이 공부하는 **Glenn**은 SceneDelegate는 multiple Window를 지원하기 위해서 생긴것 같다고 만약 iPad에도 호환되는 App이 아닌 iPhone에서만 사용하는 App을 만든다면 SceneDelegate가 없지 않아도될까에 대한 의문을 제시해주었다.<br>(참고, iPhone은 Multiple Window를 지원하지 않음)

그리고 실제로 이에 대해서 **Glenn, 이니, 꼬말**이 테스트를 진행해주었다!! (감사합니다🙌🏻) <br>
테스트의 내용은 **SceneDelegate와 Multiple Window간의 상관관계**이다. <br>
총 4가지 경우에 대한 테스트이다. 

1. SceneDelegate가 **있는 상황**에서 Support Multiple Windows **비활성화**
2. SceneDelegate가 **있는 상황**에서 Support Multiple Windows **활성화**
3. SceneDelegate가 **없는 상황**에서 Support Multiple Winodws **비활성화**
4. SceneDelegate가 **없는 상황**에서 Support Multiple Windows **활성화**

이에 대한 결과는 다음과 같다. 

- 1번에서는 iPad 앱에서 Show All Windows라는 메뉴가 뜨지 않는다. <br>
**즉, Multiple Window를 사용 불가**
- 2번에서는 iPad 앱에서 정상적으로 **Multiple Window 사용 가능**
- 3번에서는 SceneDelegate안에 있는 메서드들이 AppDelegate안에 있어도 <br>
**iPad 앱이 잘 동작**한다. 
- 4번에서는 SceneDelegate안에 있는 메서드들이 AppDelegate안에 있어도 <br> 
**iPad 앱은 정상적으로 호출**되지만, **화면이 검은색으로 나오게 된다.** 
그리고 **Multiple Window 화면도 동일하게 검은색으로 생성**된다. <br>
(SceneDelegate에 있어야 할 window가 AppDelegate에 있어서 찾지 못해서 발생하지 않을까?)

결론은 **Multiple Window를 지원하려면 window 프로퍼티와 scene 메서드들이 SceneDelegate에 필수적으로 있어야한다.** 만약 Multiple Window를 지원하지 않는다면 AppDelegate에 SceneDelegate 메서드들을 이전해서 사용해도 정상적으로 작동한다. 

❗️ 최종 결론은 **Scene과 SceneDelegate는 하나의 프로세스에 여러 개의 UI(Multiple Window)를 지원하기 위해서** Apple에서 새롭게 만들어 낸 것 같다!!

#### Scene의 역할
**UIKit**은 **UIWindowScene 개체를 사용하여 앱 UI의 각 instance를 관리**한다. <br>
`scene` 은 **하나의 UI 인스턴스를 표현하기 위한 windows와 view controllers들을 포함**하고 있다.

각각의 `scene` 은 UIKit과 app 사이에서 interaction을 하기 위해 사용하는  
**UIWindowSceneDelegate** 를 가지고 있다. 

`scene` 들은 **같은 메모리와 앱 process 공간을 공유하며 concurrent(동시에) 실행**된다.<br>
그 결과 **하나의 앱에서 여러 개의 scene을 가질 수 있고, 
scene delegate 개체들은 동시에 작동**하게 된다. 

달라진 점 2번에서 UI의 상태를 알 수 있는 **UILifeCycle 역할**을 **SceneDelegate**가 하게 되었다고 했다.
그리고 AppDelegate는 sceneSession을 통해서 scene에 대한 정보를 업데이트 하는  Session Life Cycle이 추가되었는데 과연 sceneSession은 무엇일까?

### SceneSession이란?
UISceneSession 객체는 scene의 고유한 런타임 instance를 관리한다. <br>
user가 앱에 **새로운 scene을 추가**하거나, 개발자가 프로그래밍 방식으로 새로운 scene을 요구한다면 
**시스템은 해당 scene을 tracking하기 위한 session 객체를 생성**한다. 
Session은 **scene의 고유한 식별자와 세부 구성 정보가 포함**되어 있다. 

**UIKit**은 **scene 자체의 lifetime 동안 session 정보를 유지**하며, 
app switcher를 통해서 사용자가 **scene을 닫게되면 session을 삭제**한다. 
session 객체는 직접 생성하는 것이 아니라 **UIKit에 의해서** 앱과 사용자의 interaction에 대한 응답으로 **session이 생성**된다. 

개발자는 UIApplication의 **requestSceneSessionActivation(_:userActivity:options:errorHandler:) 메서드**를 통해서 **UIKit**에게 프로그래밍 방식으로 **새로운 scene과 session 생성을 요청**할 수 있다. 
UIKit은 app의 Info.plist 파일의 내용을 기반으로 기본 구성 데이터로 session을 초기화한다. 

### iOS 13 이후 AppDelegate의 역할
`UIApplicationDelegate`는 **protocol** 이다. <br>
그리고 이를 채택한 클래스가 바로 `AppDelegate` **class**이다. <br>

#### iOS 12 이전에 AppDelegate의 Life-Cylce Management 
iOS 12 이전에는 `Appdelegate` 를 사용해서 앱의 주요 life cycle 이벤트를 처리했다. <br>
특히 `Appdelegate` 는 앱이 foreground에 있는지 background로 옮겨졌는지, <br>
즉 앱의 상태를 업데이트하는데 사용되었다. 

#### iOS 13 이후 역할
1. 앱의 **중앙 데이터 구조를 초기화**
2. 앱의 **scene을 구성**(Configuration)
3. 앱 **밖에서 발생한 알림(배터리 부족, 다운로드 완료 등)에 대응**
4. 앱의 scenes, views, view controllers에 한정 되지 않고 **앱 스스로를 타겟하는 이벤트에 대응**
5. 애플 푸쉬 알림 서비스와 같이 **앱이 시작할 때 요구되는 모든 서비스를 등록**

`UIApplicationDelegate` 프로토콜은 앱을 설정하고, 앱의 상태 변화에 대응하며, <br>
다른 앱 수준의 이벤트를 처리하는데 사용하는 여러 메소드를 정의한다. <br>

`AppDelegate` 클래스는 **Xcode에서 프로젝트를 생성할 때 마다 자동으로 생성**된다.<br>
특별한 경우가 아닌 이상 **앱을 초기화하고 앱 수준의 이벤트에 반응**하기 위해서는 <br>
Xcode에서 제공하는 `AppDelegate` 클래스를 사용해야한다.  <br>

### UISceneDelegate의 역할
`UISceneDelegate` 는 **protocol**이다. <br>
scene에서 발생하는 life-cycle event에 반응하기 위해서 사용하는 중요한 메서드들이다. 

`UISceneDelegate` 개체는 직접 생성하는 것이 아니라, scene에 대한 구성 데이터의 일부로 custom delegate class의 이름을 지정하면 된다. 이러한 정보는 앱의 Info.plist 파일 또는 app delegate의 `application(_:configurationForConnecting:options:)` 함수에서 반환하는 UISceneConfiguration 개체에서 설정할 수 있다. 

`SceneDelegate` 클래스 또한 **Xcode에서 프로젝트를 생성할 때마다 자동으로 생성**된다. <br>
내부에는 **scene의 상태들에 관련된 method**들을 가지고 있다. <br>
함수들의 역할은 아래와 같다. <br>
```swift
func scene(**_** scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
// Scene함수는 UI창을 선택적으로 구성하고 제공된 UI창에 scene을 연결해준다. 
// (iOS 13~)만약 스토리보드를 사용하는 경우에는 자동으로 window 프로퍼티가 초기화되고 
// 화면에 첨부되게 된다.SceneDelegate는 연결된 scene 또는 session이 새로 생성된 
// 것이라는 것을 의미하지는 않는다.
}
func sceneDidDisconnect(_ scene: UIScene) { 
// 시스템에 의해서 scene이 해제되면 호출이된다. Scene이 background에 들어간 직후나 
// session이 삭제되었을 때 호출된다. 다음 번에 해당 scene에 연결되는 경우에 새로 생성되는
// 자료들과 관련된 모든 자료들을 해제해준다.Session이 반드시 삭제되는 것은 아니므로 scene은
// 나중에 다시 연결될 수도 있다.
}
func sceneDidBecomeActive(_ scene: UIScene) {
// Scene이 활성화 되었고 현재 사용자 이벤트에 응답하고 있음을 알린다.
// 인터페이스를 로드 한 후 인터페이스가 화면에 표시되기 전에 호출된다
}
func sceneWillResignActive(_ scene: UIScene) {
// Scene이 활성상태를 해제하고 사용자 이벤트에 대한 응답을 중지하려고 함을 알린다.
// 시스템 경고를 표시 할 때와 같은 일시적인 중단을 위해 이 메서드를 호출한다. 
// 이 메서드가 반환 될 때까지 앱은 background 또는 foreground로 다시 전환되기를 
// 기다리는 동안 최소한의 작업을 수행해야 한다.
}
func sceneWillEnterForeground(_ scene: UIScene) {
// scene이 foreground에서 실행되고, 사용자에게 표시 될 것임을 delegate에게 알린다.
// 이 전환은 새로 생성되고 연결된 scene뿐만아니라 background에서 실행 중이고, 
// 시스템 또는 사용자 작업에 의해 background로 가져온 scene 모두에 대해 발생한다. 
// scene이 화면에 표시되기 위해 foreground에 들어가므로 이 메서드는 항상 
// sceneDidBecomeActive (_ :) 메서드를 호출합니다.
}
func sceneDidEnterBackground(_ scene: UIScene) {
// Scene이 background에서 실행되고 더 이상 화면에 표시되지 않음을 Delegate에게 알립니다.
// 이 방법을 사용하여, scene의 메모리 사용량을 줄이고, 공유 리소스를 확보하며, 
// scene의 사용자 인터페이스를 정리합니다. 이 메서드가 반환 된 직후 UIKit은 
// 앱 전환기에 표시하기 위해 Scene의 인터페이스의 스냅 샷을 찍습니다.
}
```

### Deployment Target이 iOS 13미만인 상황에서도 앱을 사용해야한다면 어떻게 해야할까?
UIScene과 UISceneDelegate는 iOS 13이후에 생성된 것인데, 
**Deployment Target이 iOS 13미만도 지원을 해야한다면 프로젝트 설정을 어떻게 해야할까?**

이에 대한 해결책은 다음과 같다. <br>
AppDelegate.swift 파일과 SceneDelegate.swift 파일에서 새로 추가된 부분은 **iOS 13이상 부터만 적용한다고 `@available(iOS 13.0, *)`를 명시**해주면 된다. 
```swift
// AppDelegate.swift 파일 내부에서 

    // MARK: UISceneSession Lifecycle
    @available(iOS 13.0, *)
    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }
    
    @available(iOS 13.0, *)
    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
    }
    
// SceneDelegate.swift 파일 내부 

    @available(iOS 13.0, *)
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        if #available(iOS 13.0, *) {
             guard let _ = (scene as? UIWindowScene) else { return }
         } else {
             // Fallback on earlier versions
         }
    }
    
    @available(iOS 13.0, *)
    func sceneDidDisconnect(_ scene: UIScene) {
    }
    
    @available(iOS 13.0, *)
    func sceneDidBecomeActive(_ scene: UIScene) {
    }
    
    @available(iOS 13.0, *)
    func sceneWillResignActive(_ scene: UIScene) {
    }

    @available(iOS 13.0, *)
    func sceneWillEnterForeground(_ scene: UIScene) {
    }
    
    @available(iOS 13.0, *)
    func sceneDidEnterBackground(_ scene: UIScene) {
    }
```

## Respond to Scene-Based Life-Cycle Event
앱이 scenes을 지원한다면, UIKit은 각각의 scene에 대해서 개별적인 life-cycle event를 제공한다. 여기서 `scene` 이란 장치에서 **실행 중인 앱 UI의 한 인스턴스**를 나타내는 것이다. 앱을 사용하는 사용자는 각 앱에 대해 여러 scene을 생성하고 개별적으로 표시하거나 숨길 수 있다.
왜냐하면 각각의 scene은 각각의 life-cycle을 가지며, 또한 각각 다른 실행 상태를 가질 수 있다.
예를 들면, 하나의 scene은 foreground에 있을 수도 있고 나머지 scene들은 background에 있거나 중지되어 있을 수 있다. 

아래의 그림은 scene의 상태 전환을 보여준다. 

![](https://images.velog.io/images/minni/post/c9384e68-d8da-4b6e-b6bc-4f10ec9829ba/image.png) 

사용자 또는 시스템이 앱에 **새로운 장면을 요청하게 되면** 
`UIKit`은 해당 scene을 만들어 **첨부되지 않은(unattached) 상황으로 만들어놓는다.** 

이후에 **사용자가 요청한 scene**은 **실제 화면에 보이는 foreground로 올라가게 된다**. 
**시스템에서 요청한 scene**은 **일반적으로 background로 이동해서 event를 처리**하게 된다. 
예를 들어 시스템은 background에서 scene을 실행하여 위치 이벤트(location event)를 처리할 수 있다.

사용자가 **앱의 UI를 해제**하면(홈으로 돌아가거나 다른 어플을 실행시키는 경우), 
`UIKit`은 **해당 scene을 background 상태로 옮겨주는데 일시 중단된 상태로** 옮겨지게 된다. 

`UIKit`은 언제든지 background 또는 일시 중단된 scene의 연결을 끊고 
**리소스를 요청하여 unattached 상태로 되돌릴 수 있다.**

### UIScene의 state 종류
`UIScene` 클래스 안에 `ActivationState` 라는 Enum 타입을 가지고 있다. 
```swift
extension UIScene {

    @available(iOS 13.0, *)
    public enum ActivationState : Int {     
        case unattached = -1
        case foregroundActive = 0
        case foregroundInactive = 1
        case background = 2
    }
}
```
1. `unattached`
- `scene`은 처음에 **unattached 상태로 시작**되며 **시스템이 connection notification을 주기 전**까지는 계속 **이 상태를 유지**한다. 또한 scene은 유저가 app switcher로 부터 interface를 dismiss하거나 자원을 재요청 할 때 attached 상태로 다시 돌아오게 된다. 
2. `foregroundActive`
- `scene`이 **foreground에서 돌아가고 있으며, 현재 event들을 받고 있는 상태**이다.
active scene의 interface는 화면에 있으며 사용자에게 보여지게 된다. 
3. `forgroundInactive`
- `scene`이 **foreground에서 돌아가고는 있지만 event를 받지는 않는다.** 
`scene`이 **다른 상태로 전환되는 동안에 바로 이 foregroundInactive 상태를 통과**하게 된다.
예를 들면, foregroundInactive 상태는 시스템 알람이 오거나 알람 창을 내리거나 app-switching 상태에 있는 경우가 있다.
4. `background` 
- `scene`이 **스크린 위에서가 아닌 background에서 돌아가고 있는 상태**이다.
background scene은 보여지는 interface가 없다. 

⚠️ **suspended**이 상태는 실제로 case 안에는 존재하지는 않는다. 
`scene`이 **background 상태에 있으며, 아무것도 실행되지 않는 상태**를 의미한다.

### 이러한 task를 수행할 때 scene trasition을 사용할 것!
- UIKit이 scene을 앱에 연결해주며, 초기 UI 화면을 구성하고 화면에 필요한 데이터들을 로드해주는 경우
- foreground-active 상태로 전환될 때, UI를 구성하고 사용자와의 interact를 준비해야 할 때
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
- foreground-active 상태를 종료한 후, 데이터를 저장하고 앱을 조용히 동작시킬때
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
- background 상태로 전환되면, 중요 업무를 끝내고 가능한 많은 메모리를 확보한 후 앱 스냅샷을 준비할때
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
- 화면이 disconnect 될 때, 화면과 관련된 모든 공유 리소스를 정리한다.
- 화면 관련된 이벤트 외에도 UIApplicationDelegate 개체를 사용하여 앱 시작에 응답해야 한다.
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app)
    
## Respond to App-Based Life-Cycle Events
위에서도 언급했듯이 iOS 12이하 버전에서는 scene을 지원하지 않았다. <br>
따라서 UIKit은 UIApplicationDelegate로 모든 life-cycle 관련 이벤트를 전달해주었다. 
App delegate는 앱 화면에 표시되는 창(window)과 아래의 그림에서 나타내는 모든 screen들을 관리한다. 
그 결과 **앱의 상태 전환**은 **외부 화면의 content를 포함하여 앱의 모든 UI에 영향을 미친다.** 

아래의 그림은 app delegate 개체를 포함한 상태 전환을 나타낸다. 

![](https://images.velog.io/images/minni/post/63da97e7-d4dc-409b-8369-6250f519fcd8/image.png) 

앱이 실행된 이후에 시스템은 **UI가 화면에 나타나는지 아닌지에 따라서** 
앱을 **inactive나 background 상태로 전환**한다. 
foreground로 앱이 시작한다면, 시스템은 자동으로 앱을 active한 상태로 변경한다. 
이후 앱이 종료될 때까지 상태는 activate 나 background 사이에서 전환된다. 

### UIApplication의 state 종류
`UIApplication` Class를 안에 `State` 라는 Enum 타입을 가지고 있다. 
```swift
    @available(iOS 4.0, *)
    public enum State : Int {
        case active = 0
        case inactive = 1
        case background = 2
    }
```
1. `active`
- 앱이 **foreground에서 실행되고 있으며, events를 받고 있다.**
2. `inactive`
- 앱이 **foreground에서 실행되고는 있지만, events를 받지는 않는다.**
**interruption으로 인해서 발생**하거나 **background에서 active로 상태 변환을 할 때 이 상태를 통과**한다. 
3. `background`
- 앱이 **background에서 실행**되고 있다. 

⚠️ **suspended**이 상태는 실제로 case 안에는 존재하지는 않는다. 
앱이 **background 상태에 있으며, 아무런 코드도 실행하지 않는 상태**를 의미한다. 

### 이러한 task를 수행할 때 app transition을 사용할 것!
- 시작할 때 앱의 데이터 구조와 UI를 초기화하는 경우 
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app)
- 활성화하는 경우 UI 구성을 마치고 사용자와 interact를 준비할 때
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground)
- 비활성화 시 데이터를 저장하고 앱 동작을 조용하게 할 때
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
- background 상태로 전환되면 중요한 업무를 마치고 가능한 많은 메모리를 확보한 후 앱 스냅샷을 준비
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
- 앱이 종료될 때 모든 작업을 즉시 중지하고 공유 리소스를 해제한다. 
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623111-applicationwillterminate)
    
### Respond to Other Significant Events
life-cycle event 처리 외에도 앱은 아래의 표에 나열되는 이벤트를 처리할 수 있도록 준비되어야한다.
UIApplicationDelegate 개체를 사용하여 이러한 이벤트 대부분을 처리할 수 있다.
경우에 따라서 notification을 사용해서 다른 앱을 사용하는 도중에도 이벤트들을 처리할 수 있을 것이다. 

![](https://images.velog.io/images/minni/post/5c44e24e-dc2a-4408-900b-27727b105e4a/image.png) 

- **Memory warning** :  앱의 메모리 사용량이 너무 높은 경우 
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle/responding_to_memory_warnings)
- **Protected data becomes available/unavailable** : 사용자가 기기를 잠구거나 잠금 해제할 때
	- [become available 👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623044-applicationprotecteddatadidbecom)
	- [become unavailable 👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623019-applicationprotecteddatawillbeco)
- **Handoff task** : NSUserActivity 개체가 처리되어야할 때 
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622963-application)
- **Time Changes** : 전화통신사가 시간 업데이트를 보내는 경우와 같이 여러 시간 변경사항이 발생할 때
	-[👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622992-applicationsignificanttimechange)
- **Open URLs** : 앱이 자원을 열어야하는 경우에 
	- [👉🏻 여기 링크로](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application)

## 고민해 볼 만한 부분들❗️❗️
고민해 볼 만한 부분들은 iOS 스타터 캠프에서 **야곰🐻**이 생각해 볼 만한 부분들에 준 질문들이다. 
이에 대해서 수업시간과 수업시간 이후에 알아본 내용으로 정리해본다. 
### Application Life Cycle 모식도의 점선과 실선
위의 scene-based Life-Cycle, app-based Life-Cycle의 모식도를 보면 **점선**과 **실선**이 있다. 
이에 대해서 **오동나무**🌳와 **야곰**🐻이 고민해 볼 만한 질문을 주었다.
과연 **실선**과 **점선**의 의미는 무엇일까?

**점선**의 경우에는 특별한 event 없이도 시스템이 자동으로 수행해주는 상태의 전환이라고 할 수 있다. 
**실선**의 경우에는 사용자나 시스템에 의해 발생한 event로 인해서 발생하는 상태의 전환이라고 할 수 있다. 

### Life Cycle에서 메모리와 프로세스의 관점
Life Cycle에서 Unattached, Suspended, Not Running의 상태에 대해 
**메모리와 프로세스의 관점에서의 차이**는 무엇일까라는 질문을 **야곰**🐻이 해주었다. 🙏

![](https://images.velog.io/images/minni/post/0a1981c1-27e0-426f-ab6c-1a220d250410/image.png) 

`Unattached` : `scene`이 **시스템으로 부터 connection notification을 받기 전**까지는 **이 상태를 유지**한다. 따라서 메모리를 점유하고 있고, 실행 중인 상태라고 할 수 있다. <br>
`Suspended` : `App`이나 `scene`이 **백그라운드에 있고 아무것도 실행되지 않는 즉, 프로세스 대기 상태**이다. 따라서 메모리는 점유하지만 대기중인 상태라고 할 수 있다. <br>
`Not Running` : 아예 `App`이 **실행되지 않았거나, 실행이 되었었지만 시스템에 의해서 종료된 상태**이다. 따라서 메모리에도 없고, 프로세스의 관점에서도 아무 것도 실행되지 않는다. 

### scene, window, view의 관계는 무엇일까?
![](https://images.velog.io/images/minni/post/3ebb7ba2-0b95-41a1-accf-c5f6d252d95a/image.png) 

**UIScene**은 app UI의 객체이며, window와 view controller(View)를 포함한다.

![](https://images.velog.io/images/minni/post/30b3a35d-3cab-4cc7-beb4-ce63ea4304b6/image.png) 

**UIWindow**는 app UI의 뒷 배경이라고 할 수 있고, view에 이벤트를 보내는 객체이다. 

![](https://images.velog.io/images/minni/post/a4026ce3-041d-433b-8e59-bba01c76e1f4/image.png) 

**UIView**는 화면에서 사각형 모양의 content를 관리하는 객체이다. 

### AppDelegate의 역할인 Process Lifecycle은 무엇일까? 
![](https://images.velog.io/images/minni/post/b2e83a5e-7a0d-46ca-8b82-13e641956cee/image.png) 

이 부분에 대해 많은 검색을 해보았지만 뭔가 명확하게 어떠한 역할인지 알려주는 문서를 발견하지 못했다. 
관련된 WWDC 영상을 보다가 Process Lifecycle의 역할은 App Launched, App Terminated와 같은 것들을 AppDelegate에게 알려주는 거라고 나왔길래 관련된 함수들을 찾아보았다. 

그래서 위와 사진과 같은 함수들을 발견하게 되었고 UIApplicationDelegate를 extension하여 구현된 함수들이다. 안에는 app이 실행되고, 종료되는 것을 알려주는 함수와 포스팅에서 언급했던 부분인 Respond to other significant Event에서 나왔던 event들에 대응하는 함수들이 있었다. 

즉, Process Lifecycle의 역할은 **앱이 실행되었는지, 혹은 종료되었는지를 확인하고 중요한 event들에 대응하는 것**이라고 생각한다. 

## 참고
1. [Apple Document Scenes](https://developer.apple.com/documentation/uikit/app_and_environment/scenes) 
2. [Apple Document UISceneSession](https://developer.apple.com/documentation/uikit/uiscenesession)
3. [Apple Document Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
4. [Apple Document UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
5. [Apple Document UISceneDelegate](https://developer.apple.com/documentation/uikit/uiscenedelegate)
6. [Apple Document UIScene.ActivationState](https://developer.apple.com/documentation/uikit/uiscene/activationstate)
7. [Apple Document UIApplication.State](https://developer.apple.com/documentation/uikit/uiapplication/state)
8. [Apple Document UIScene](https://developer.apple.com/documentation/uikit/uiscene)
9. [Apple Document UIWindow](https://developer.apple.com/documentation/uikit/uiwindow)
10. [Apple Document UIView](https://developer.apple.com/documentation/uikit/uiview)
11. [Lena's Blog AppDelegate와 SceneDelegate](https://velog.io/@dev-lena/iOS-AppDelegate와-SceneDelegate)
12. **야곰**🐻과 iOS 스타터 1기 캠퍼들(**준스, 오동나무**🌳 등등) 
13. Scene과 Multiple Window Support 상관관계를 테스트 해준 (**Glenn, 이니, 꼬말**)💯

> 야곰🐻과 캠퍼들 그리고 많은 부분을 참고한 Lena 선배님께도 감사의 말씀을 전합니다🙇‍♂️