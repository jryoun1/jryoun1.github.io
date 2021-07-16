---
title:  "Xcode 프로젝트에서 Storyboard 제거하기"
excerpt: "Xcode 프로젝트에서 Storyboard 제거하기"

categories:
  - Swift
tags:
  - [app launch sequence]
last_modified_at: 2021.07.01
---
## 들어가며
Xcode에서 뷰를 작성할 때 스토리보드를 통해서 구현하거나, 코드를 통해서 구현할 수 있다.
하지만 협업을 할 때 스토리보드를 사용하여 여러 사람이 뷰를 구현하는 경우에는 충돌이 발생할 가능성이 크다. 따라서 협업을 할 때는 코드를 통해서 뷰를 작성하는 것이 괜찮다고 생각한다. 

그렇다면 코드로 뷰를 작성할 때, 사용하지 않는 Storyboard를 프로젝트 내부에서 가지고 있어야할까?

물론 가지고 있어도 상관은 없지만 사용하지 않는 파일은 제거하는 것이 좋다고 생각하여 이번 포스팅에서는 Storyboard를 제거하는 방법과 제거 이후에 App 실행시 화면을 불러오는 방법에 대해서 다뤄볼 예정이다. 

### 프로젝트 설정
먼저 일반적으로 프로젝트를 생성한 뒤에 Storyboard를 없애는 방법은 다음과 같다.

![](https://images.velog.io/images/minni/post/1093620e-6511-4bd4-84bb-81ecd7ee48ce/image.png) 

위의 스크린 샷에서 `Main Interface` 항목에서 기본으로 설정되어 있는 `Main` 을 삭제한다. 
이때 그냥 `Main` 을 지우고 enter를 누르면 된다. 
프로젝트의 `Main.storyboard` 파일은 휴지통으로 버려도 되고 reference만 삭제해도 상관없다.

### Info.plist 수정
다음은 Info.plist에서 수정을 해주면 된다. 

![](https://images.velog.io/images/minni/post/d40b1853-88f3-4cd8-ae49-1d6d541efff7/image.png) 

`Application Scene Manifest` 부분에 마우스를 갖다 대면 `+` 버튼이 나오는데, 계속 확장해서 위와 같이 펼친다. `Item 0` 을 확장해서 `Storyboard Name` 키 부분을 삭제한다. <br>
이때 삭제는 `-` 버튼으로 삭제하면 된다. 


### 💡 App launch steps
개발자가 Xcode에서 Run을 하거나 사용자가 App을 눌렀을 때, 
App이 launch 되는 과정은 아래의 그림과 같다. 

![](https://images.velog.io/images/minni/post/e2dc259a-f32f-4573-bf30-89772617728e/image.png) 


1. 사용자나 System에 의해서 App이 실행된다. 
2. Xcode는 UIKit의  `UIApplicationMain(_:_:_:_:)` 라는 main 함수를 호출한다.
3. `UIApplicationMain(_:_:_:_:)` 함수는 UIApplication 객체와 app delegate를 생성한다. 
4. UIKit이 app의 default interface를 main storyboard나 nib 파일로부터 불러온다.
5. UIKit이 app delegate의 `application(_:willFinishLaunchingWithOptions:)` 함수를 호출한다. 
6. UIKit은 app delegate와 view controller들에 있는 추가적인 함수들을 호출하는 **state restoration**을 수행한다. 
7. UIKit이 app delegate의 `application(_:didFinishLaunchingWithOptions:)` 함수를 호출한다. 

이렇게 초기화가 완료되면, system이 scene delegate나 app delegate를 사용해 UI를 나타내고, app의 life cycle를 관리하게 된다. 

### SceneDelegate 코드 수정
이제 프로젝트 내부에서는 Storyboard에 대한 설정은 모두 제거하였다. <br>
이에 위의 App launch steps에서 4번째 순서에서 default interface를 불러와야하는데, 
main storyboard를 제거했기 때문에 이를 대체할 코드들을 SceneDelegate와 AppDelegate에 작성해주어야한다. 

⚠️ SceneDelegate와 AppDelegate에 대한 차이는 아래의 링크로 가서 확인해보면 된다. <br>
👉🏻 [WWDC 2019 Architecting your app for multiple windows](https://velog.io/@minni/Architecting-your-app-for-multiple-windows) <br>
👉🏻 [iOS App life cycle](https://velog.io/@minni/iOS-Application-Life-Cycle)

먼저 iOS 13 이후부터 앱이 새로운 scene lifecycle을 채택했다면, 
UIKit은 이제 AppDelegate로 부터 UI의 상태(state)와 관련된 메서드들을 호출하지 않게 될 것이다.
대신에 SceneDelegate 메서드들을 호출하게 된다.

따라서 아래와 같이 SceneDelegate에 코드를 추가해주면 된다. 
```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = MainViewController() // 자신의 시작 ViewController
        window.makeKeyAndVisible()
        self.window = window
    }
}
```
<br>

#### 💡 그렇다면 iOS13 이후 scene life cycle을 채택한 App들은 iOS12 이전의 대한 지원을 중단해야할까?
<br>
그렇지 않다. 2가지 경우에 대해서 모두 작성을 해놓고 유지한다면 UIKit이 알아서 런타임에 옳바른 메서드들을 불러주게 될 것이다.

### AppDelegate 코드 수정
Scene life cycle을 채택했을 때, iOS 13이전의 기기에서도 작동하려면 다음과 같은 코드를 AppDelegate에 추가해주면 된다. 
```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // 13이상인 경우에는 SceneDelegate에서 이미 초기화 되었으니까 바로 return
        if #available(iOS 13.0, *) { 
            return true
        }
        
        // 13이전의 경우에는 SceneDelegate에서 해주었던 작업을 그대로 진행해주면 된다. 
        window = UIWindow()
        window?.rootViewController = MainViewController() // 초기 ViewController
        window?.makeKeyAndVisible()
        return true
    }
}
```
<br>
이렇게 위의 4가지 과정을 다 마치면 Storyboard를 프로젝트에서 없애도 정상적으로 App이 실행되고 코드로 작성한 UI가 나오는 것을 확인할 수 있다. 

## 참고 자료
[Apple Document - About the App Launch Sequence](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app/about_the_app_launch_sequence)<br>
[how-to-create-a-swift-app-without-storyboard](https://www.dev2qa.com/how-to-create-a-swift-app-without-storyboard/)