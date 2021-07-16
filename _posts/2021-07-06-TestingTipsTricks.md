---
title:  "Testing Tips and Tricks"
excerpt: "WWDC18 Testing Tips and Tricks"

categories:
  - WWDC
tags:
  - [testing]
last_modified_at: 2021.07.06
---



# WWDC18 Testing tips & Tricks

이번 세션에서는 발표자 Brian과 Stuart가 개발하는 앱을 테스팅하는 과정에서 생기는 문제들을 해결하기 위해서 다음과 같은 4가지 주제에 대해서 다루게 된다.

1. Testing network requests
2. Working with notifications
3. Mocking with protocols
4. Test execution speed

본격적으로 들어가기에 앞서 **WWDC 2017 Engineering for Testablility** 세션에 등장하는 `Pyramid of Tests` 에 대해서 간단하게 알아보고 넘어가자.



## 💡 Pyramid of Tests

![1](/assets/images/WWDC18TestingTipsTricks/1.png)

`Pyramid of Tests` 는 테스트를 진행함에 있어 **테스트가 어떠한 형태로 이뤄지면 바람직한지를 나타내는 개념**이라고 할 수 있다. Pyramid의 위로 올라갈수록 실제 유저가 사용하는 영역이라 믿을 만한(reliable) 영역입니다. 하지만 테스트를 진행하는데에 있어서 **실행 시간이 오래 걸리고 유지보수, 디버깅 하기 더욱 어려운 영역**이다. 반면 아래로 내려올수록 더 많은 테스트 코드를 작성하고 진행하게 된다.  

애플에 의하면 이러한 Pyramid 모델에 의한 테스트 접근은 **철저성(thoroughness), 품질(quality), 실행 속도(execution speed) 사이의 균형**을 잡는 데 도움이 된다고 한다.



### Unit Test

Pyramid의 가장 하단에 존재하는 `Unit Test (단위테스트)` 는 함수 같이 a single piece of code를 검증하는데 도움을 주게된다. 주로 함수에 input을 넣고 기대되는 output이 나오는지 확인하는 방식으로 테스트를 진행하게 된다. 단위 테스트는 **짧고 간단하며 빠르게 실행된다는 특징**을 가진다. 



### Integration Test

Pyramid의 중간에 위치하는 `Integration Test (통합테스트)` 는 단위테스트보다는 큰 부분을 검증하는데 사용된다. 단위 테스트가 함수 단위로 테스트를 진행했다면 통합테스트는 주로 클래스의 모음이나, 서로 다른 컴포넌트들이 함께 올바르게 동작하는지 확인하기 위해서 진행한다. 일반적으로 단위 테스트만큼 많은 테스트 코드와 양이 필요하지는 많지만, 시간이 조금 더 걸리게 되며 한 번에 더 많이 앱을 테스트한다. 

이에 단위 테스트만하고 통합테스트를 하지 않았을 때 발생할 수 있는 문제점에 대해 아래와 같은 좋은 예시 사진이 있다.

![img](https://blog.kakaocdn.net/dn/bdf44d/btqxUKdSa6k/bft1axhweKlBfl4DkLRhz1/img.png)

즉, 각각의 문은 제대로 동작하지만, 합쳤을 때 정상적으로 작동하지 않을 수 있다라는 것이다. 



### End-to-End Test (UITest)

마지막으로 Pyramid의 최상단에 위치하는 `End-to-End Test ` 는 `UI Test` 라고도 하며, UI 또는 사용자 인터페이스에 대해서 테스트를 하게 된다. 이를 통해서 앱이 기대했던 것처럼 동작하는지를 검증하게 되며, 세 개의 테스트들 중에서 **가장 오래걸린다.** 그러나 앱의 동작을 검증하기 위해서는 매우 중요한 테스트이다. 



## 1️⃣ Testing network requests

이렇게 Pyramid of Test에 대해서 간단하게 알아보았고, 이제 본격적으로 Brian과 Stuart의 앱을 테스트해보자. 

먼저 이들이 개발하는 앱은 휴대폰 사용자의 현재 위치를 받아와서 자신이 지정해 놓은 관심있는 장소를 거리순으로 보여주는 앱이다. 이에 기존에 작성한 코드를 먼저 확인해보자.

![2](/assets/images/WWDC18TestingTipsTricks/2.png)

위의 그림과 같이 `loadData()` 라는 함수의 매개변수로 사용자의 현재 위치를 받고 이를 통해서

1. 위치로 URL을 생성하고
2. `dataTask()` 함수를 통해서 생성한 URL로 부터 데이터를 받아오고
3. 받아온 데이터를 View에 보여주는 부분

까지 나타나게 된다. 

그러나 이 3가지 역할들을 `loadData()` 라는 하나의 함수에 넣고 테스트 하는 것은 유지보수하거나, 테스트를 진행하기에는 어려움이 발생하게 된다. 즉, 지금의 `loadData()` 라는 함수는 아래와 같이 모든 과정을 진행하기 때문에 단위 테스트를 진행하기에 어려워진다. 

![3](/assets/images/WWDC18TestingTipsTricks/3.png)



### 🔍 Prepare URLRequest, Parse Response 분리

따라서 위에서 언급한 Pyramid 형태에 맞춰 Unit Test를 진행하기 위해서 아래의 두 가지 부분을 따로 Struct로 만들어서 분리해보자. 

![4](/assets/images/WWDC18TestingTipsTricks/4.png)

그렇게 URLRequest를 생성하고, 받은 데이터를 Parsing하는 부분을 `PointsOfInterestRequest` 라는 Struct로 만들면 다음과 같다. 



#### PointsOfInterestRequest Struct 생성

![5](/assets/images/WWDC18TestingTipsTricks/5.png)

`PointsOfInterestRequest`  Struct는 사용자의 위치를 받아와서 URLRequest를 반환해주는  `makeRequest()`  함수와 받은 데이터를 parsing 해서 반환해주는 `parseResponse()` 라는 함수를 가지게 된다.

이제 이렇게 분리한 함수들을 Unit test 하는 방법은 다음과 같다.  

![6](/assets/images/WWDC18TestingTipsTricks/6.png)

위에서 생성한 `PointsOfInterestRequest` Struct를 **request** 라고 생성해준다. 그리고 `makeRequest()` 함수를 테스트하는 `testMakingURLRequest()` 함수와 `parseResponse()` 함수를 테스트하는 `testParsingResponse()` 함수를 만들고 각각이 제대로 동작하는지 테스트를 진행하여 output을 검증해준다. 

이때 XCTest에서는 **throws** 키워드를 사용하면 `do - catch block` 없이 try를 사용할 수 있다는 장점이 있다. 이렇게 Prepare URLRequest, Parse Response 분리하여 Unit test를 진행하였다. 



### 🔍 Create URLSession Task 분리

![7](/assets/images/WWDC18TestingTipsTricks/7.png)

그렇다면 이제 Create URLSession Task도 따로 분리하여 Unit Test를 진행할 수 있도록 해보자. 위에서 분리한 Request Type에서의 구현한 함수을 가지는 `APIRequest protocol` 을 생성하여 Create URLSession Task에 관련된 코드를 view controller로부터 분리시킨다. 

#### ARIRequest protocol, APIRequestLoader Class 생성

![8](/assets/images/WWDC18TestingTipsTricks/8.png)

따라서 위의 코드와 같이 분리한 Request Type, Response Type, makeRequest(), parseResponse()를 가지는 `APIRequest protocol` 을 생성한다. 

그리고 이 프로토콜을 `ARIRequestLoader class` 가 채택하도록 한다.  이 클래스는 URLSession과 ARIRequest로 초기화하는 `init()` 함수와 requestData로부터 URLRequest를 생성하여 서버와 통신하여 responseData를 받고 파싱하는 `loadAPIRequest()` 함수를 가진다. 이런식으로 분리하여 Unit test를 진행할 수 있다. 

![9](/assets/images/WWDC18TestingTipsTricks/9.png)

그러나 Unit Test를 진행하기 전에 Test Pyramid의 중간 단계인 Integration Test를 보자. 위의 그림과 같은 통합 테스트를 통해서 URLSession API와 interaction이 올바르게 동작하는지와 지금까지의 data flow가 정상적인지 확인할 수 있다. 



### 🔍 URLProtocol

이때 foundation URL loading system은 위와 같은 3가지 단계를 해결할 수 있는 방법을 제공해준다.

![10](/assets/images/WWDC18TestingTipsTricks/10.png)

- `URLSession` : network request를 수행하는 high level API를 app에게 제공해준다. 
- `URLSessionDataTask` : 해당 객체는 in-flight(한 번 보내면 취소할 수 없는?) request를 나타냄
- `URLProtocol` : lower-level API 로서 아래와 같은 작업들을 화면 뒤에서 수행 
  - 네트워크 연결을 여는 작업, request를 작성하는 작업, response를 읽어오는 작업
  - URLProtocol은 URL Loading System을 확장할 수 있게 상속되어지게 설계됨

그리고 foundation은 HTTPS와 같은 common protocol을 위해 기본적으로 내장된 프로토콜을 채택하는 클래스를 제공해준다. 

![11](/assets/images/WWDC18TestingTipsTricks/11.png)

그러나 테스트 코드를 작성할 때 **APIs가 구현이 되어있지 않는 상황**이나, **network를 사용하지 못하는 경우**가 있을 수 있다. 이런 경우에는 `Mock Protocl`을 구현하여 request에 대한 검증과 mock response를 제공하도록 할 수 있다. 그리고 위의 그림과 같이 URLProtocol은 URLProtocolClient protocol을 통해 시스템에 진행상황을 시스템에 전달해 준다. 

#### MockURLProtocol 생성

위에서의 URLProtocol을 실제로 사용하는 방법은 다음과 같다. 먼저 test bundle에 `MockURLProtocol` 을 생성해주자. 

![12](/assets/images/WWDC18TestingTipsTricks/12.png)

1. test에서 MockURLProtocol 클래스를 사용하기 위해서 requestHandler 라는 closure property를 선언
2. URLSession task가 시작되면, system이 MockURLProtocol에 URLRequest value와 URLProtocol client 객체를 제공하며 초기화 
3. 이후 MockURLProtocol 클래스 내부의 startLoading() 함수 호출됨
4. requestHandler가 URLRequest를 매개변수로 하여 response와 data를 반환
5. 반환된 data나 error를 시스템에 전달

만약 request cancellation(취소)에 대해서 test를 하려면 stopLoading() 내부에 위와 비슷한 구현을 해주면 된다. 



### APILoaderTest 작성

이렇게 위에서 구현한 것들은 다음과 같은 test code를 통해서 테스트 할 수 있다. 

![13](/assets/images/WWDC18TestingTipsTricks/13.png)

ARIRequestLoader\<PointsOfInterestRequest> 인 loader를 변수로 가진다. 

`setup()` 은 테스트에 필요한 것들을 준비해주는 부분으로 loader에 request, urlSession를 매개변수로 초기화 해준다. 

`testLoaderSuccess()` 에서는 관리 가능한 inputCoordinate와 mockJSONData 사용하여 request가 정상적으로 만들어졌는지 테스트하고, 이후 response가 정상적으로 동작하는지 테스트하게 된다.

![14](/assets/images/WWDC18TestingTipsTricks/14.png)

이를 통해서 network나 APIs가 구현되지 않았어도 다음과 같은 통합테스트를 진행할 수 있다. 이를 통해서 작성한 코드가 서로 잘 작동하며, 시스템과도 제대로 통합되는 것을 확인할 수 있다. 

### End-to-End-Test 

또한 end-to-end test를 하는데에 있어서도 매우 도움이 된다. 특히 UI testing을 진행하는데에 큰 도움이 될 수 있다. UITest에 대한 자세한 내용은 WWDC2015 UI Test Session을 참조하면 좋다. End-to-end test를 작성할 때 직면하는 문제 중에 하나는 **테스트가 실패하였을 때 어디서 실패했는지 원인을 찾기 어려운 것이다.** 

![15](/assets/images/WWDC18TestingTipsTricks/15.png)

이러한 문제를 완화하는 방법 중에 하나는 바로 Mock Server에 local instance를 설정하여, UI Test 도중에 실제 서버가 아닌 mock server에 대한 요청을 수행하도록 하는 것이다. 이것이 해결책이 되는 이유는 **서버에서 앱으로 다시 제공되는 데이터를 제어할 수 있기 때문**에 UITest를 할 때 신뢰성이 높아지게 되는 것이다. 

![16](/assets/images/WWDC18TestingTipsTricks/16.png)

이를 실행하는 기술 중에 하나는 바로 Unit Test Bundle의 stack에서 해당 애플리케이션에 직접 호출하여 실제 서버에 대해 요청을 전달하는 몇 가지 테스트를 수행하는 것이다. 이를 통해 UI Testing으로 인한 복잡한 문제를 동시에 처리할 필요 없이, 애플리케이션이 만드는 request를 서버가 수락하는지에 대한 검증과 서버의 응답을 parsing 할 수 있는지를 검증할 수 있다. 



### Testing network 요약

1. Unit Test를 위해서 코드를 작게 분할하는 방법
2. URLProtocol을 사용해서 mocking network requests를 생성하는 방법
3. Pyramid testing 전략을 통해 작성 코드에 대한 확실한 신뢰를 얻을 수 있는 균형 잡힌 테스트를 구성하는 방법



## 2️⃣ Working with notifications

이제 이번 세션의 두 번째 주제인 working with notifications에 대해서 다뤄보자. 

여기서의 `Notifications`는 foundation level의 notification, 즉 `NSNotification`에 대해서 말하는 것이다. 

코드를 테스트 할 때 notification을 observe하거나 post하는 subject에 대해서 테스트 해야하는 경우가 있다. **Notification은 1대 다 통신(one - to - many)기술**이며, 그 말은 1개의 notification이 post되면 앱 내부의 여러 수신자에게 전송이 된다는 것이다. 따라서 **의도하지 않은 부작용(side-effect)을 방지하기 위해 notification을 테스트 할 때에는 항상 격리된 방식으로 테스트하는 것이 중요**하다. 그렇지 않으면 잘못된 테스트와 신뢰할 수 없는 테스트로 이어질 수 있다.

### 문제 발생 코드와 테스트

문제가 발생할 수 있는 코드를 확인해보자. 

![17](/assets/images/WWDC18TestingTipsTricks/17.png)

PointsOfInterestTableViewController는 위에서 구현하던 앱의 ViewController이다. App의 location authorization (위치 권한)이 변경될 때 마다 근처의 관심있는 장소들을 tableview에 보여주게 되는데, 이때 data를 다시 가져와야하는 경우가 있을 수도 있다. 따라서 CurrentLocationProvider class의 authChanged를 관찰하고 있다가 권한이 변경되면, 이에 따라 처리가 필요한 경우를 체크하기 위해서 boolean타입의 flag 변수를 바꿔주는 `handleAuthChanged()` 함수를 호출하게 된다. 

이떄 notification이 post를 수신하여 flag 변수가 제대로 변화하는지 테스트 코드를 작성해보자. 

![18](/assets/images/WWDC18TestingTipsTricks/18.png)

위와 같이 테스트 코드를 작성할 수 있는데, 이때 post는 NotficationCenter.default를 사용해서 post를 한다. 물론 테스트는 성공하고 flag 변수는 true로 변화하게 된다. 그러나 앱의 다른 코드에 알려지지 않은 부작용이 있을 수 있다. 예를 들어 UI Application appDidFinishLaunching notfication과 같은 **system notification은 많은 layer에서 관찰되고, 이는 알 수 없는 부작용을 발생시키거나 테스트를 지연시킬 수 있다.** 따라서 이 코드를 분리해서 테스트하는 것이 좋다. 



### 해결 방법

그 전에 NotificationCenter는 여러 개의 인스턴스를 가질 수 있고, `default` 라는 기본 인스턴스가 클래스 속성으로 있지만 필요한 경우에는 추가 인스턴스 생성이 가능하다는 것을 인식해야한다.  

1. `NotificationCenter.default` 가 아닌 새로운 NotificationCenter를 생성 
2. 생성한 새로운 NotificationCenter를 `init()` 에 넘기고, 새로운 프로퍼티로 저장
3. 기존의 `NotificationCenter.default` 를 새로운 프로퍼티로 변경

#### 🔍 Notification observe 테스트

이러한 방법은 dependency injection(의존성 주입)이라고도 한다. 아래와 같이 기존의 코드를 위에서 말한 순서에 따라서 수정해주면된다. 

![19](/assets/images/WWDC18TestingTipsTricks/19.png)

 `init()` 에 notificationCenter를 넘겨받아서 새로운 **notificationCenter** 변수에 저장하고 기존의 `Notification.default`를 **notificationCenter** 로 대체해주는 것을 확인해 볼 수 있다. 

여기서 눈 여겨 볼 것은 바로 `init(notificationCenter: NotificationCenter =.default)` 부분이다. 이 부분을 통해서 Unit Test에서만 새로운 NotificationCenter를 넘겨주어서 사용하고 실제 코드에서는 매개변수로 넘기지 않아서 default를 사용할 수 있게 된다. 즉, **기존 코드 변경을 피할 수 있게 해준다.** 

이제 이를 테스트 하는 코드는 다음과 같이 변경하면 된다. 

![20](/assets/images/WWDC18TestingTipsTricks/20.png)

테스트 코드에서 `default` 가 아닌 새로운 notificationCenter를 생성하고 이를 `init()`의 매개변수로 넘겨준다. 이렇게 작성하므로 Notification은 app 내부의 다른 곳에서 received되지 않고 오로지 해당 PointsOfInterestTableViewController 에서만 받을 수 있고 이로서 독립적으로 테스트가 가능해졌다. 

#### 🔍 Notification Post 테스트

이를 통해서 notification의 **observe**에 대해서는 독립적으로 테스트를 진행하는 방법을 알 수 있었다. 그러나 notification의 **post**를 테스트하기 위해서는 어떻게 해야할까? 똑같이 Notification을 분리하는 방법을 사용할 것이지만 이번에는 내장되어있는 APIs인 `XCTNSNotificationExpectation` 에 notification observer를 붙여서 사용해 볼 것이다. 

![21](/assets/images/WWDC18TestingTipsTricks/21.png)

먼저 `CurrentLocationProvider` 클래스를 생성한다. 이 클래스는 Auth가 변경되는 `NotificationCenter.default` 를 사용해서 post를 하는 함수를 가지고 있다. 이제 해당 코드를 테스트 하는 코드를 작성해보자. 

![22](/assets/images/WWDC18TestingTipsTricks/22.png)

이 테스트 코드를 통해서 notifyAuthChanged() 함수가 호출되었을 때 notification이 post되는 것을 검증할 수 있다. 위에서 생성한 `CurrentLocationProvider` 인스턴스를 poster라는 변수로 선언하고, block-based Observer를 생성하기 위해서 addObserver를 하고 내부 블럭에서 removeObserver를 하게 된다. 이를 통해서 NotficationCenter observer를 생성할 수 있는데 해당 부분을 내장된 `XCTNSNotificationExpectation` 을 사용하여 대체 할 수 있다. `XCTNSNotificationExpectation` 은 기대되는 NSNotification을 수신하면 충족된다. 코드의 양을 줄일 수 있는 좋은 방법이지만 여전히 암묵적으로 `NotficationCenter.default` 를 사용해서 post를 한다는 문제가 있다. 

따라서 NotficiationCenter를 분리하여 새로 생성하도록 아래와 같이 수정해보자. 

![23](/assets/images/WWDC18TestingTipsTricks/23.png)

Notification observe 테스트를 할 때 작성했던 것처럼 Notification을 분리하는 작업을 그대로 해준다. 여기서도 마찬가지로  `init(notificationCenter: NotificationCenter =.default)` 로 작성하여 **기존 코드 변경을 피할 수 있게 해준다.** 

그리고 이를 테스트 하는 코드는 다음과 같다. 

![24](/assets/images/WWDC18TestingTipsTricks/24.png)

기존 코드에 notificationCenter를 새롭게 생성하여 CurrentLocationProvider의 `init()` 의 매개변수로 전달하고, 이를 `XCTNSNotficationExpectation` 에서도 사용하는 방법으로, 이를 통해서 Notification이 생성한 notificationCenter로만 보내지며, 독립적으로 테스트가 가능해졌다. 또한 여기서 중요한 것은 바로 `wait()` 의 timeout의 값은 0인데, 이는 `notifyAuthChanged()` 가 호출되고 return을 할 때는 **이미 notification이 post 되었기 때문**이다.

위에서 설명한 Notification을 테스트 하는 기술들을 통해서 기존 코드를 변경하지 않고, 완전히 독집적으로 테스트를 진행할 수 있다. 



## 3️⃣ Mocking with protocols

다음은 이제 이번 세션의 세 번째 주제인 Mocking with protocols이다. 이번에는 Unit Test를 작성할 때, 외부 class와 상호작용하는 경우에 어떻게 해결해야하는지에 대해서 다루게 된다. 

앱을 개발하다보면 앱 내부의 다른 클래스나 SDK에 의해서 제공되는 클래스들과 상호작용해야 할 때가 있다. 이런 경우에는 테스트 코드를 작성하기 힘든데, 왜냐하면 **외부 클래스를 생성하기가 어렵거나 불가능하기 때문**이다. 특히나 **직접 생성하도록 설계되지 않은 API**에서는 이러한 현상이 자주 발생하게 되며, 이러한 API에 **테스트를 해야하는 delegate 메서드가 있는 경우**에는 더욱 테스트가 어려워진다. 

### 문제 발생 경우

외부 클래스의 Mock interface를 protocol을 사용해서 구현해 상호작용을 할 수 있도록 만들어서 테스트를 신뢰할 수 있도록 해보자. 

![25](/assets/images/WWDC18TestingTipsTricks/25.png)

Brian과 Sturat이 개발하는 앱에는 `CurrentLocationProvider` 클래스가 있는데, 생성자에서 정확도와 delegate를 설정해준다. 그리고 현재 위치를 얻고나서 해당 위치가 interest 장소인지 검사하는 `checkCurrentLocation()` 함수를 가지고 있다. 해당 함수 내부에서 `locationManager.requestLocation()` 은 현재 위치를 얻기 위해서 CLLocationManagerDelegate를 사용하게 된다. 그리고 해당 부분은 extension에서 `location(_ manager: didUpdateLocations locs:)` 내부에 구현한 함수를 실행하게 된다. 

이제 해당 코드로 테스트 코드를 작성해보자.

![26](/assets/images/WWDC18TestingTipsTricks/26.png)

`CurrentLocationProvider` 인스턴스를 생성하고 제대로 `init()` 되었는지 검증한다. 그리고 현재 위치가 interest 장소인지 확인하는 `checkCurrentLocation()` 를 호출하는데, 이때 문제가 발생하게 된다. 먼저 내부에서 우리가 작성한 코드가 아닌 CLLocationManger의 함수인 `requestLocation()` 이 언제 호출되는지 알 수 없으며,  또한 기기의 위치 동의 권한 상태에도 영향을 받게 되는 문제가 발생한다. 이로 인해서 테스트는 기기의 상태에 의존하게 된다. 게다가 테스트를 진행할 때 현재 위치를 mock data로 제공할 수도 없다. 



### Mocking Using a Subclass

예전에는 이러한 문제를 외부 클래스인 `CLLocationManager` 클래스에 호출하는 함수인 `requestLocation()` 을 overriding해서 해결하려고 했었다. 물론 정상적으로 작동할 수는 있지만 이는 위험하다. 왜냐하면 몇몇의 SDK들은 상속을 할 수 없도록 설계되어있는 경우가 있어 다르게 동작할 수 있기 때문이다. 이 중에서 가장 큰 문제는 바로 **CLLocationManager에서 다른 메서드를 호출하도록 코드를 수정하는 경우에는 테스트 코드 내부에서도 상속 클래스에서도 해당 메서드를 override해주어야하는 문제**이다. 즉, 상속에 의존하여 해결하는 경우, **컴파일러는 다른 메서드를 호출하는 것에 대해서 알려주지 않게 된다.** 이는 테스트를 진행할 때 잊어버리기 쉬운 부분이고 결국 테스트에 실패하게 될 것이다.

따라서 이를 protocol을 사용해 외부 타입을 mocking 하는 방법을 사용하는 것을 추천한다. 



### Mock external Type using Protocols

![27](/assets/images/WWDC18TestingTipsTricks/27.png)

`LocationFetcher` protocol을 생성하고, `CLLocationManager` 클래스가 이를 채택하도록 한다. 그리고 기존의 `locationManager` 를 `locationFetcher` 로 대체해준다. 이때도 `init(locationFetcher: LocationFetcher = CLLocationManager())` 부분을 통해서 **기존 코드의 변경을 하지 않도록 해준다.** 

![28](/assets/images/WWDC18TestingTipsTricks/28.png)

그리고 `checkCurrentLocation()` 함수 내부에서도 `locationManager `를 `locationFetcher` 로 변경해준다. 

이제 delegate method를 변경해야하는데, 아래의 기존 코드에서 보다시피 delegate가 manager 매개변수를 새로 생성한 protocol이 아닌 실제 CLLocationManager로 생각하기 때문에 복잡하긴 하지만 다음과 같이 protocol을 적용하여 해결할 수 있다. 

![29](/assets/images/WWDC18TestingTipsTricks/29.png)

1. `LocationFetcherDelegate` 라는 `CLLocationManagerDelegate` 와 이름을 제외하고 동일한 protocol을 생성
2. `LocationFetcher` protocol 내부에서 기존의 `delegate` 라는 변수명을 `locationFetcherDelegate` 로 변경하고 타입도 새롭게 생성한 protocol로 설정
3. `CLLocationManager` 가 `LocationFetcher` protocol을 채택하도록 하며, 이때 getter와 setter를 구현하여 강제 캐스팅을 사용해 CLLocationManagerDelegate를 앞뒤로 변환하도록 구현 <br>

‼️ 이때 강제 캐스팅을 사용하는 이유는 **클래스가 기존의 CLLocationManagerDelegate protocol과 새로 생성한 LocationFetcherDelegate protocol에 모두 부합하는지 확인**하고 하나만 작성하거나 다른 것을 잊지 않도록 하기 위해서 사용한다. 

![30](/assets/images/WWDC18TestingTipsTricks/30.png)

그리고 나서 `CurrentLocationProvider` 클래스 생성자 내부에서 기존의 `delegate` 를 `locationFetcherDelegate` 로 수정해준다. 

![31](/assets/images/WWDC18TestingTipsTricks/31.png)

또한 `CurrentLocationProvider` 클래스가 기존에 채택하던 `CLLocationManagerDelegate` 를 `LocationFetcherDelegate` 로 수정하고 함수도 `locationManager()` 에서 `locationFetcher()` 로 수정한다. 

![32](/assets/images/WWDC18TestingTipsTricks/32.png)

마지막으로 `CLLocationManagerDelegate` 의 **`locationManager()` 함수 내부에서 `locationFetcher()` 함수를 호출**해주어야하는데, 이는 `CLLocationManagerDelegate` 가 새로운 mock delegate protocol인 `LocationFetcherDelegate` 에 대해서 모르기 때문에 꼭 해주어야만 한다. 



#### Test Code 작성

이제 작성한 코드들을 Unit Test 해보자. 

![33](/assets/images/WWDC18TestingTipsTricks/33.png)

테스트 코드 내부에 `LocationFetcher` 프로토콜을 채택한 `MockLocationFetcher` 구조체를 생성하여 내부에서 필요한 것들을 처리해준다. 이때 `handleRequestLocation` 은 가짜 위치를 받아오기 위한 변수이며, `requestLocation()` 함수 내부에서 가짜 위치를 받아와서 이를 delegate를 통해서 `locationFetcher()` 함수의 매개변수로 넘겨주도록 구현한다. 

이후 테스트 코드에서는 위에서 생성한 Struct의 인스턴스를 locationFetcher 변수로 생성하고, 가짜 위치를 직접 입력하여 넘겨주도록 한다. 이후 `CurrentLocationProvider` 클래스의 생성자에 locationFetcher를 넘겨주고, `checkCurrentLoation()` 함수를 호출하고 주어진 가짜 위치가 interest 장소인지 검증하도록 한다. 이를 통해서 실제 CLLocationManager를 구현하지 않았지만, 이를 사용하고 테스트할 수 있었다. 



#### Mocking with Protocols 요약

다시 한 번 요약하면 다음과 같은 단계를 따라서 진행하면 된다. 

1. 외부 클래스의 인터페이스를 나타내는 새로운 프로토콜 정의 <br>
   이때 새로운 프로토콜은 외부 클래스에서 사용하는 모든 함수와 속성을 포함해야하며, 선언이 정확하게 일치할수도 있다.
2. 기존의 외부 클래스에 대한 extension을 생성하여 새로운 프로토콜을 준수하도록 선언
3. 외부 클래스에서의 모든 사용을 새로운 프로토콜로 교체 <br>
   이때 init()에 매개변수를 추가하여 테스트에서 유형을 설정할 수 있도록 한다. 



#### Mocking Delegates with Protocols 요약

위에서 SDK의 일반적인 패턴인 delegate protocol을 mock하는 방법에 대해서도 설명했지만 이를 요약하면 다음과 같다. 

1. mocking 하는 프로토콜과 유사한 함수를 가진 mock delegate protocol을 정의 <br>
   이때 **내부의 실제 type을 새롭게 생성한 mock protocol type으로 변경**
2. mock protocol에서 delegate property의 이름을 변경
3. 실제 type의 extension에서 mock delegate property를 구현하고 변경

이러한 과정은 상속에 의존하는 것보다는 많은 코드가 필요할 수 있지만, 이러한 방법은 **더욱 신뢰성 있고 코드가 확장되어도 변경될 가능성이 낮아진다.** 왜냐하면 이러한 방법을 사용하면 **컴파일러는 코드 내에서 새로운 함수를 호출하는 어떤 경우에도 이들이 새로운 프로토콜에 포함되어야 한다고 강제**하기 때문이다. 



## 4️⃣ Testing execution speed

이제 이번 세션의 마지막 주제인 Testing execution speed이다. 테스트를 실행할 때 시간이 오래걸리는 것을 개발자의 생산성을 저하시킨다. 이렇게 테스트를 느리게 하는 요인 중에는 테스트 코드가 비동기이거나, 타이머를 사용하여 테스트를 진행하여 인위적으로 대기해야하는 경우가 있다. 따라서 이렇게 테스트가 인위적으로 지연되는 것을 피할 수 있는 방법에 대해서 다뤄볼 예정이다. 



### 잘못된 예시

![34](/assets/images/WWDC18TestingTipsTricks/34.png)`FeaturedPlaceManager` 클래스는 현재 장소를 저장하는 `currentPlace` 변수와 다음 장소를 10초 뒤에 보여주는 `scheduleNextPlace()` 함수를 가지고 있다. 이러한 클래스를 테스트하는 코드는 다음과 같다. 

![35](/assets/images/WWDC18TestingTipsTricks/35.png)

테스트 코드 내에서 `RunLoop.current.run(until: Date(timeIntervalSinceNow: 11))`  라는 코드를 사용해서 11초 동안 run loop를 돌게 되고, 그 이후에 현재 장소가 바뀌었는지 검증하게 된다. 이러한 테스트 방법은 시간이 매우 오래 걸리기 때문에 좋지 않다. 

이를 완화하는 방법으로는 다음과 같이 기존의 클래스와 테스트 코드를 수정할 수 있다.  

![36](/assets/images/WWDC18TestingTipsTricks/36.png)

기존의 `FeaturedPlaceManager` 클래스에 TimeInterval 타입인 `interval` 변수를 초기값을 10으로 하여 추가해준다. 초기 값을 10으로 설정해놓으므로서 기존의 코드의 수정을 피하게 해준다. 그리고 `scheduleNextPlace()` 함수 내부에서 `interval` 시간 뒤에 `showNextPlace()` 함수를 호출하도록 수정한다. 

그리고 이를 테스트하는 코드 내부에서 `manager.interval = 1` 를 통해서 interval을 1로 변경해줘서,  `RunLoop.current.run(until: Date(timeIntervalSinceNow: 2))`  로 수정할 수 있게 된다. 기존의 11초에 비해서 2초로 확실히 테스트 시간이 줄어들어서 효율적이긴 하지만 여전히 delay(연기)가 존재하는 테스트이다. 즉, 결국 테스트에 진행되는 시간만 단축된 것이라고 할 수 있고, 특히나 비동기 코드에서는 시간이 짧아지게 되면 테스트 결과가 항상 True가 아닐 수도 있게 된다. 



### 테스팅 속도를 개선하는 방법

더 효율적으로 개선하는 방법은 다음과 같다.  

1. delay 되는 기술들을 식별 (e.g. `Timer`, `DispatchQueue.asyncAfter` ... )
2. 테스트 할 때는 위에서 식별한 기술들을 mock 
3. delay 되는 처리를 즉시 호출하도록 처리

실제로 코드에 적용해보자. 

![37](/assets/images/WWDC18TestingTipsTricks/37.png)

위에서 본 코드에서 `Timer.schedularTimer()` 는 실제로 2가지 역할을 수행한다. 

1. Timer 생성
2. 현재 run loop 에 생성한 Timer 추가

즉, `Timer.schedularTimer()` 는 타이머를 생성하는데에 있어서는 편리하긴 하지만, 두 가지 역할을 분리한다면 더욱 쉽게 테스트 가능하다. 따라서 이를 다음과 같이 수정해보자. 

![38](/assets/images/WWDC18TestingTipsTricks/38.png)

`runLoop` 변수를 따로 생성하고 2개의 역할을 분리하였다. 이때 두 번째 역할에서 runLoop의 경우에는 외부 클래스이다. 따라서 세번 째 주제였던 mocking with protocols 기술을 사용하여 이를 테스트 할 수 있다. 

#### mocking with protocols

![39](/assets/images/WWDC18TestingTipsTricks/39.png)

runLoop API와 일치하는 하나의 `addTimer()` 함수를 포함하는 `TimerScheduler` 라는 새로운 프로토콜을 생성한다. 그리고 기존에 코드에서 `runLoop` 를 새로 만든 프로토콜인  `timerSchedular` 로 수정해준다. 

이후 테스트 코드는 다음과 같이 작성하면 된다. 

![40](/assets/images/WWDC18TestingTipsTricks/40.png)

먼저 생성한 프로토콜인 `TimeSchedular` 를 채택하는 `MockTimeSchedular` 구조체를 생성하여 실제 RunLoop를 사용하지 않고 Timer를 전달할 수 있도록 한다. 

그리고 test body인 `testSchduleNextPlace()` 내부에서는 생성한 구조체의 인스턴스인 `timerSchedular` 를 생성한다. `timerSchedular.handlerAddTimer` 에서 timer를 수신하고, 스케줄러에 추가되면 타이머의 지연을 기록한 다음 타이머를 작동시켜 지연을 우회한다. 

이후에 `FeaturedPlaceManager` 인스턴스인 `manager `를 mockTimerSchedular 인 `timerSchedular` 를 넘겨주면서 생성한다. 그리고 `manager.scheduleNextPlace()` 를 호출하여 테스트를 진행한다. 이때 지연은 존재하지 않으며 더 이상 timer에 의존적이지 않게 되어 신뢰성이 높아지게 된다. 또한 기존에는 검증하지 못했던 `timerDelay` 와 `accuracy` 도 검증할 수 있게되었다.  



#### 지연된 작업을 테스팅 하는 방법

기존에 존재하던 지연을 위와 같은 방법을 통해서 아예 제거하였으며, 이 방법으로 지연된 작업을 포함하는 코드를 테스트할 수 있다. 그러나 테스트 전체에서 실행속도를 가장 높이기 위해서는 테스트의 대부분을 직접적이고 지연된 작업을 전혀 mock 하지 않도록 구성하는 것이 가장 좋긴하다. 

### 테스팅 속도를 높이는 다른 방법

#### Expectations 의 사용

`XCTNSPredicateExpectation` 의 사용에 대해서도 생각을 해봐야한다. 해당 expectation은 직접적인 callback 메커니즘보다 polar에 의존하기 때문에 다른 expectation 클래스에 비해서 성능이 떨어지며, 주로 UI test에서 사용된다. 따라서 Unit Test에서는 `XCTestExpectation` , `XCTNSNotificationExpectation` , `XCTKVOExpectation` 과 같은 직접적인 메커니즘을 사용하는 것이 좋다. 



#### App 실행 시 최적화

대부분의 앱은 실행 시 일정량의 설정 작업을 수행해야 하며, 실제로 앱 실행을 위해서는 해당 작업이 필요하지만, **테스트를 실행할 때에는 많은 작업이 불필요할 수 있다.** 예를 들면, view controller 로딩, 네트워크 요청 시작, 분석 패키지 구성과 같은 작업들은 Unit test 시나리오에서는 일반적으로는 불필요한 작업의 예이다. `XCTest` 는 appdelegate의 `appDidFinishLaunching` 메서드의 호출까지 기다린 후에 테스트를 시작하게 된다.  

![41](/assets/images/WWDC18TestingTipsTricks/41.png)

따라서 이를 해결하는 방법으로는 사용자 지정 환경 변수나 launch argument를 설정하는 것이다. 

`Scheme editor → Test → Arguments → Environment Variables` 로 이동해서 위에서 보이는 그림처럼 `IS_UNIT_TESTING` 환경변수에 1을 추가한다. 이후 appDelegate에서 DidFinishLaunching 코드를 아래와 같이 수정해서 설정한 상태를 확인한다. 

![42](/assets/images/WWDC18TestingTipsTricks/42.png)

위의 코드에서 `isUnitTest == false` 인 경우에 Unit test에서 불필요한 set-up 코드들을 넣어주면 된다. 이때 주의해야하는 부분은 skip 하는 작업들이 Unit Test에 영향을 끼치는지 그렇지 않은지 확인해야한다. 



## 결론

이번 Session에서는 testing pyramid를 통해서 앱에서 균형 잡힌 테스트를 작성하는 방법에 대해서 언급하면서 네트워크 작업을 테스팅하는 기술들에 대해서 다뤄보았다. 이를 통해서 아래와 같은 방법들에 대해서 자세한 예시와 함께 내용들을 배워볼 수 있었다. 

1. network request에 있어서 Unit test를 위해 코드를 분할하는 방법
2. URLProtocol을 사용해 mocking network request를 생성하여 실제로 API나 network가 구현되어 있지 않아도 테스트하는 방법
3. notification의 observe나 post에 대해서도 독립적으로 테스팅하는 방법
4. 외부 클래스나 delegate가 있을 때 상속이 아닌 프로토콜을 사용해 mocking 하는 방법 
5. Testing 속도를 높이는 방법

이러한 부분은 실제로 테스트 코드를 작성하는데 있어서 큰 도움이 될 것이라고 생각한다. 



## 참고 문서

1. [WWDC18-Testing_tips_Tricks](https://developer.apple.com/videos/play/wwdc2018/417/)
2. [GDG 안드로이드 코리아 - 아장아장 안린이의 안드로이드 테스트 첫걸음](https://www.slideshare.net/WooseopKim3/gdg-and-kor/WooseopKim3/gdg-and-kor)
3. [Apple Document - XCTNSNotificationExpectation](https://developer.apple.com/documentation/xctest/xctnsnotificationexpectation)



