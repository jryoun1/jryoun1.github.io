---
title:  "[WWDC] Architecting your app for multiple windows"
excerpt: "WWDC19 Architecting your app for multiple window 영상에 대해서 알아보자."

categories:
  - WWDC
tags:
  - [Application Life Cycle, AppDelegate, SceneDelegate]
last_modified_at: 2021.01.17
---
## 들어가며
이번 포스팅에서는 크게 3가지에 대해서 포스팅 해 볼 예정이다. 

1. App life cycle 역할에 대한 변화
2. SceneDelegate를 활용해서 앱을 실행시켰을 때 life cycle과 call stack 확인
3. multiple window를 지원할 때 scene의 sync를 맞추는 법

에 대해서 포스팅 할 예정이다. 

## 1. Changes to App lifeCycle

### iOS 12 이전

iOS 12와 그 이전에 App Delegate는 2개의 주요한 역할을 가지고 있었다. 

1. **Process Lifecycle** : Process 수준에서의 event를 알려준다. 그래서 system은 AppDelegate에게 process가 실행상태인지, 아니면 종료하려고 하는지와 같은 정보를 알려준다. 

![](https://images.velog.io/images/minni/post/59ac004a-96c6-4e53-9011-c1815b5ebefe/image.png)

2. **UI Lifecycle**: Application UI의 상태(state)를 알게해준다. 그래서 didEnterForeground()와 같은 메서드를 통해서 시스템은 UI의 상태(state)를 알려주게 된다.  

![](https://images.velog.io/images/minni/post/f031a1ac-9948-4557-99b7-90f576f6475b/image-20210117160134676.png)

이러한 구조는 위의 그림처럼 **하나의 앱이 하나의 process와 하나의 user interface 객체와만 matching** 되었던 iOS 12 이전까지는 괜찮았다.

그래서 아마 아래의 그림과 같은 AppDelegate를 가졌을 것이다. 

![](https://images.velog.io/images/minni/post/27c0e76b-25eb-478d-80cb-548dfa6a00eb/image-20210117155815540.png)

여기서 우리의 App은 launching이 끝나게되면 non-UI setup인 global setup(e.g. 데이터베이스 연결, 자료구조의 초기화)를 했을 것이다.  그리고 그 이후에 UI(User Interface) 를 설정했을 것이다. 

### iOS 13 이후

이러한 방식은 iOS 12전까지는 유효했지만 iOS 13 부터는 이러한 방식은 유효하지 않는다고 한다. 

![](https://images.velog.io/images/minni/post/70fe3b51-1ac7-4630-b8a4-6fd67d26acb2/image-20210117160338409.png) 

왜냐하면 앱들은 **이전처럼 하나의 process를 사용하긴 하지만 다수의 UI 객체나 scene session들을 가지게 되기 때문**이다. 

즉, 이 말은 AppDelegate의 역할의 변화가 필요하다는 것이다. 
그 결과 **AppDelegate**는 **Process Lifecycle** 역할은 그대로 유지하지만 UI와 관련된 그 어떠한 작업들에 대한 **UI Lifecycle**의 역할은 **UI Scene Delegate**에게 넘겨주게 된다. 

![](https://images.velog.io/images/minni/post/c3ec41df-7c4a-4f73-957d-b5be6ebeb386/image-20210117161305421.png)

UI setup이나 teardown과 같은 AppDelegate가 했었던 작업들은 이제 모두 Scene Delegate로 넘어가게 되었다는 것이다.  

따라서 iOS 13 이후부터 앱이 새로운 scene lifecycle을 채택했다면, UIKit은 이제 AppDelegate로 부터 UI의 상태(state)와 관련된 메서드들을 호출하지 않게  될 것이다. 
대신에 아래의 그림에 있는 새로운 Scene Delegate 메서드들을 호출하게 된다. 
(대부분은 1:1로 AppDelegate에 있었던 함수들과 매칭되기 때문에 그리 크게 달라진 점은 없다.) 

![](https://images.velog.io/images/minni/post/91fe3d77-873a-4035-9a1b-028f07fa1088/image-20210117161509817.png)


#### 그렇다면 과연 iOS 13 이후에서 multiple window를 지원하게 되면, iOS 12 이전의 앱들은 지원을 중단해야할까?

그렇지 않다. 다시 배포를 할 때 위의 **2가지 경우에 대해서 모두 작성을 해놓고 유지**한다면 **UIKit이 알아서 런타임에 옳바른 메서드들을 불러주게 될 것이다.** 

#### AppDelegate에 추가된 역할

시스템은 App Delegate에게 새로운 scene session이 생성되거나 기존에 존재하던 scene session이 사라지는 경우에 알려주게 될 것이다. 

 ![](https://images.velog.io/images/minni/post/c96c241d-6fbe-4523-ab9f-6700ee23cecd/image-20210117162331340.png)


## 2. Using the scene delegate

### LifeCycle 예시

![](https://images.velog.io/images/minni/post/9bbea677-d5df-478e-b817-d5e87ae60e50/image-20210117164436365.png)

여기서 파란색 앱을 시작할 때 call stack을 한 번 살펴보자. (맨 처음으로 실행시키는 경우) 

![](https://images.velog.io/images/minni/post/11c4a659-0e29-4c42-af18-a19f9068a37d/image-20210117190923781.png)

1️⃣ 먼저 AppDelegate가 iOS 12 이전과 동일하게 **didFinishLaunching 함수를 부르게 될 것**이다. 여기서는 **한 번 뿐인 non-UI setup을 진행**하게 된다. 
그리고 나서 2️⃣ 번 그림처럼 시스템은 **scene session을 생성**하게 된다. 

그러나 **실제 UI Scene을 생성하기 전에 앱에게 UI Scene Configuration을 물어보게 된다.** 

![](https://images.velog.io/images/minni/post/39c807a1-9fb6-4def-bc99-3b28bb179a62/image-20210117170914866.png) 

이 configuration에는 **어떠한 scene delegate를 사용할 것인지, 어떤 storyboard인지에 대해서** 물어본다. 이때 scene subclass를 scene과 함께 생성하고 싶다면 같이 적어주면 된다. 이러한 **scene configuration**은 **code를 통해서 동적으로 정의**할 수도 있고 **info.plist 파일을 통해 정적으로 정의**할 수도 있다. (후자가 좀 더 선호되긴함)

이를 통해서 옳바른 configuration을 선택할 수 있게 해주는데, 예를 들어 앱에는 main scene configuration이 있고 accessory scene이 있을 수도 있으므로 **`options` parameter**를 통해서 알맞은 scene configuration을 선택할 수 있도록 해야한다. 

여기서 예를 들어 info.plist 파일안에 정의를 해두었다면, 아래의 코드와 같이 이름을 통해서 가져올 수 있다. 이때 `sessionRole` parameter를 제대로 작성했는지 확인해야한다.

 ![](https://images.velog.io/images/minni/post/0c284c45-abc1-4e02-a7fe-10d07073cffe/image-20210117171618499.png) 
 
 이제 그림 3️⃣번처럼 **App은 실행되고, scene session을 가지게 된다.** 
그러나 그림처럼 아직 UI 화면은 만들어 지지 않는다. 

이때 **UI를 보이게 해주는 부분이 바로 SceneDelegate가 didConnectToSceneSession이 호출되는 시점**이다. 

![](https://images.velog.io/images/minni/post/76ca8ead-9f82-4395-ba4e-a392d16782a2/image-20210117172440462.png) 

이 부분에서는 바로 **UI window를 새로운 UIWindow initializer를 사용해서 set up** 해주게 된다. 

이때 주어진 window scene을 아는 것도 중요하지만 window를 구성할 때 **유저 activity나 state restoreation activity도 확인**해야한다는 것이다. 바로 위의 그림에서 `if  `구문에 해당한다. 
이제 그림 4️⃣와 같이 **App이 화면에 보이게 되는 것**을 확인할 수 있다. 

이제 만약 user가 **swipe up해서 앱을 switcher로 보내고 home 화면으로 돌아가게 되면** 어떠한 일이 발생할까?

바로 그림 5️⃣와 그림 6️⃣이 발생하게 된다. **SceneDelegate에 의해서 `willResignActive `와 `didEnterBackground` 가 호출되고 그리고 나서 `didDisconnect` 가 호출**이 되게 된다. 

그러나 여기서 흥미로운 점이 발생된다. 
여기서 해당 scene은 disconnect가 될 것인데 이 말은 아래의 사진과 같다. 

![](https://images.velog.io/images/minni/post/e615fc7d-147a-4c63-9066-0d7d24a12d17/image-20210117173632294.png)

 **자원을 다시 요청하기 위해서**는 시스템은 scene이 background에 들어간 일정 시점에서 **memory에서 scene을 release 해주어야한다.** 즉, **SceneDelegate가 메모리에서 release**될 것이고, 해당 SceneDelegate가 가지고 있어서 window hierarchy나 view hierarchy들 역시도 모두 release 된다는 것이다. 

이것이 무슨 뜻이냐면 결국 **해당 scene과 관련되어 App 내부 어딘가에서 메모리의 많은 자료들을 사용하고 있던 것들을 deallocate하고 release**해 줄 수 있다는 것이다.  (그러나 scene은 다시 연결되고 나중에 다시 사용될 수 있으므로 실제로 사용자 데이터 또는 상태를 영구적으로 삭제하지 않아야 한다.)

그럼 이제 마지막으로 **user가 app switcher에서 scene을 swipe up 해서 아예 없애는 경우**에는 어떻게 될까? 

이러한 경우에는 이제 그림7️⃣ 번과 같이 시스템이 **App Delegate의 `didDiscardSceneSessions`를 호출**하게 된다. 

![](https://images.velog.io/images/minni/post/ebd0dccf-5202-4b11-aa91-2a82d68a3ce0/image-20210117185911835.png)

 그리고 이때가 바로 해당 **scene에 관련된 user state나 data를 지워주는 시점**이다. 예를 들면 app에서 text를 editing하고 있었던 부분과 같이 **저장되지 않았던 부분들을 지워주게 된다**. 

또한 app switcher에서 swipe up을 통해서 **실제 앱 프로세스가 not running하고 있는 하나 이상의 UIScene을 제거해 줄 수 있다.**  만약 **process가 not running이라면, 시스템은 제거된 session을 계속해서 tracking하고 있다가 다음 번 해당 앱이 실행될 때 이것을 불러오게 된다.** 

## 3. Architecture

그렇다면 이제 architecture pattern을 살펴보자.

### state restoration
iOS 13에서는 **state restoration**은 더 이상 나이스 하게 작동하지 않는다.  
iOS 13에서 중요한 것은 바로 **App의 scene 기반 상태 복원을 구현**하는 것이 중요하다. 

![](https://images.velog.io/images/minni/post/66473e62-461d-4531-ae6e-d8ad511afe9a/image-20210117191620165.png)

 다음과 같이 document 앱에 서로 다른 4개의 session이 각각 열려있다고 하자. 
이 중에서 Packing List와 Agenda에 좀 더 집중해서 봐야한다고 하자. 
이 시점에서 Road Trip, Attendees는 시스템에 의해서 release되고 disconnect 되었을 것이다. 

그런데 만약 여기서 state restoration을 구현하지 않았다면, 다시 Road trip으로 앱을 switch 하였을 때 내가 Packing List나 Agenda에서 작성했던 정보들은 남아있지 않을 것이다. 
그리고 마치 새로운 window처럼 아무것도 없이 깔끔한 상태일 것이다. 

그렇다면 이를 어떻게 해결해야할까?
### Per-Scene State Restoration

![](https://images.velog.io/images/minni/post/cfad0ebb-3869-4fbe-8e93-01b530ee701d/image-20210117192145820.png)

 iOS 13은 위의 문제를 해결하기 위해서 새로운 **scene 기반의 state restoration API**를 가지고 있다. 

이 API는 view hierarchy를 encoding하는 것이 아니라 **window를 다시 만들 수 있게 하는 상태를 encoding하게 해준다.** 

이것은 NSUer activity도 기반으로 하는데 따라서 앱이 spotlight 검색이나 전달과 같은 강력한 기술을 사용하는 경우에 동일한 활동을 앱의 상태를 인코딩 할 수 있게 한다. 또한 iOS 13에서 시스템에 반환하는 state restoration archive는 다른 앱의 데이터 보호 등급과 일치한다. 

이를 코드로 작성하게 되면 아래의 그림과 같다.  

![](https://images.velog.io/images/minni/post/3396966c-0e3a-49a8-8170-04c7c142a4a6/image-20210117192840544.png)

 SceneDelegate에서 scene에 대한 state restoration activity를 구현하고, 그리고 현재 window에서 가장 유저 활동이 활발한 것을 찾는 메소드(`fetchCurrentUserActivity`)를 호출하는 부분이 맨 위의 파란색 박스 부분이다. 

그리고 나서 scene이 foreground로 다시 돌아게되는 순간 connected되고, 해당 session이 state restoration activity를 포함하고 있는지 없는지에 대한 체크를 해주는 부분이 `if `문에 해당한다. 

만약 있다면 해당 activity를 다시 설정해주게되고 없다면 아무런 state 없는 새로운 window를 생성하게 된다. 

결국, **사용자는 절대로 background에서 scene의 연결이 끊어진다는 것을 알 수 없게 된다.** 
왜냐하면 다시 켰을 때 상태를 복원해주는 함수들이 있기 때문이다. 
(그러나 **실제로는 background에서 scene의 연결은 끊어진다**는 것이 중요한 key point)

### Keeping Scenes in Sync

마지막으로 multiple winodw를 지원하는 앱을 구현할 때 발생가능한 중요한 issue에 대해서 알아보도록 하자. 바로 scene이 아래의 그림처럼 동시에 켜져있는 경우에 sync를 맞춰주는 것에 대한 issue이다. 

![](https://images.velog.io/images/minni/post/b55005b0-1214-4a8d-a885-3c2f145d57a5/image-20210117194025738.png)

 같은 화면에서 문자를 보냈지만 한 쪽만 문자가 가고 다른 쪽은 문자가 발송되었다는 것을 UI에 업데이트 해주지 않고 있다. 

왜 이와같은 문제가 발생하는가하면 
바로 iOS 앱 개발의 구조는 아래의 그림과 같은 구조을 따르고 있었기 때문이다.

![](https://images.velog.io/images/minni/post/e5885e8a-13db-469f-8691-a64a7b7f5cfd/image-20210117194310431.png)

 즉, 문자 전송이라는 button이라는 누름으로서 view controller가 event를 받고, view controller가 스스로 자신의 UI를 업데이트 하게 되고 이를 model에 알리고 Model Controller가 업데이트하는 구조였다. 이러한 구조는 하나의 UI 객체를 다룰 때에는 괜찮았다. 

그러나 위의 그림과 같이 다른 scene에 있지만 같은 데이터를 보여주는 second view controller는 새로운 event가 발생했고 이에 맞춰서 UI를 업데이트 해야한다는 것을 모르는 문제가 발생하고 이것은 issue라고 할 수 있다. 

![](https://images.velog.io/images/minni/post/3ccb3053-a064-4b5a-8bf8-e6e466e630c6/image-20210117195035436.png)

 따라서 이를 해결하기 위해서 구조를 바꾸면 된다. 
view controller는 그림 1️⃣번 처럼 **단지 model을 업데이트 해주기만** 하고 
그림 2️⃣번 처럼 model controller가 **관련된 모든 subscriber나 view controller들에게 새로운 데이터를 업데이트 해줘야한다고 알려주면 된다.** 

#### 코드로 구현 (Notification, Enum)
이를 할 수 있는 방법은 delegate를 사용하거나 notification을 사용하는 방법이있다. 
아니면 새로운 Swift Combine framework를 사용할 수도 있다.  

![](https://images.velog.io/images/minni/post/c8b516ae-b0da-4b0e-aedc-4af4bea3955e/image-20210117195308901.png)

 왼쪽 그림 1️⃣번은 기존의 코드인데, 
여기서 빨간색 박스 부분인 Update view 부분들을 지워서 2️⃣번처럼 만들어준다. 

그리고 model controller의 부분을 수정해주면 되는데 
그 전에 UpdateEvent라는 enum 타입을 만들어서 구조화된 이벤트 패키징하는 방법에 대해서 먼저 알아보도록 하자. (디버깅하고 테스트에 용이하도록 하려면 이렇게 만드는게 좋겠죠?)

![](https://images.velog.io/images/minni/post/cb3741e0-81ab-4534-b3e3-7fdf9c49d26b/image-20210117200156399.png)

 빨간 박스가 바로 model controller가 **새로운 메세지를 받게되면 생성하고 관련된 view controller나 scene에게 보내주기 위한 객체**이다. 
그리고 이 객체를 post할 것이기 때문에 **NSNotificationCenter를 사용**한다. 
post() 메서드를 구현함으로써 실제로 객체들을 view controller나 scene에게 보내주게된다. 

여기서 **`notification object` 로 self를 보내주는 것**은 좀 이상하지만 나중에 알게될 것이다. 
일단 넘어가도록 하자!

![](https://images.velog.io/images/minni/post/442979b7-a72f-4dd8-a97a-1f8564f9bd35/image.png) 

그리고 위의 그림과 같이 model controller로 가서 message를 받았을 때 저장하고, event 객체를 생성하여 post()를 실행해주면 된다. 

이제는 실제로 연관된 view controller에서 어떻게 업데이트가 일어나는지 알아보자. 

![](https://images.velog.io/images/minni/post/3fa102d2-267d-480b-a7c6-10cdec5e37fb/image-20210117200700676.png)

 먼저 이벤트를 수신해야하므로 **NotificationCenter Observer를 등록**해 준 부분이 빨간 박스이다. 
그리고 이러한 이벤트 수신에서 인자로 넘어오는 notification을 처리할 handler함수를 구현해주는 부분이 handle() 함수이다. 
여기서 **notification 객체로 update event를 보냈기 때문에, 이를 바로 가져올 수 있는데** 이 부분이 바로 파란 박스 부분이다. 아까 위에서 self로 보낸 이유가 바로 이 부분이다. 

그리고 이렇게 가져온 이벤트를 case에 맞게 처리하는 부분이 바로 노란색 박스 부분이고 바로 이 부분에서 UI의 업데이트가 일어나도록 구현하면 될 것이다. 

## 결론
WWDC 2019 architecting your app for multiple window 영상을 보면서  
AppDelegate와 SceneDelegate의 분리와 Scene이라는 새로운 개념은 
**결국엔 multiple window를 지원하기 위해서**였다고 생각이 들었고, 
iOS 13이후에서 앱을 실행하였을 때 call stack에 쌓이고 release되는 순서에 대해서도 알아보았다. 

그리고 마지막으로 multiple window를 지원할 때 주의해야 할 부분들에 대해서 알아보았는데, 역시 Apple은 너네는 다 계획이 있구나..!!

## 참고
[WWDC 2019 Architecting your app for multiple window 영상](https://developer.apple.com/videos/play/wwdc2019/258/)