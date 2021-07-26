---
title:  "[Swift] Core Location"
excerpt: "앱 중에서는 사용자의 위치를 추적하거나, 현재 위치를 기반으로 동작하는 앱들이 있다. 이러한 기능에 대해서 알아보자."

categories:
  - Swift
tags:
  - [CoreLocation, CLLocationManager, CLLocationManagerDelegate]
last_modified_at: 2021.01.21
---

## 들어가며
우리가 사용하는 앱 중에서는 **사용자의 위치를 추적**하거나, **현재 위치를 기반으로 동작**하는 앱들이 있다. 
예를 들면 NikeRun과 같이 사용자의 위치를 추적하는 앱, 지도 앱에서 검색을 하고 현재 위치 근처에서 결과 보기와 같이 현재 위치를 사용하는 앱이 있다.

그렇다면 이러한 기능을 Apple에서는 어떻게 제공하는지 알아보자. <br>
해당 문서는 참고에 있는 공식문서들을 기반으로 작성했습니다. 

## Core Location 프레임워크

Core Location은 Framework로 **기기의 지리적인 location이나 orientation을 획득하는데 사용**한다. 이를 통해 기기의 지리적인 **위치, 고도, 방향 그리고 주변의 iBeacon 기계의 위치까지 제공**해준다. 이 Framework는 Wi-Fi, GPS, Bluetooth, magneotmeter, barometer나 핸드폰의 하드웨어를 이용하여 데이터를 얻게된다.

그리고 우리는 **CLLocationmanger** 클래스 객체를 활용하여 Core Location 서비스를 구성하고, 시작하고 중지할 수 있다. 
**location manager** 객체를 통해 아래와 같은 활동들을 확인할 수 있다. 
- `Standard and significant location updates` : 사용자의 현재 위치에서 크고 작은 사항들을 각도 단위의 정확도로 추적한다.
- `Region monitoring` : 관심있는 특별한 지역을 모니터링 하다가 해당 지역을 벗어나거나 들어갈 때 이벤트를 발생시킬 수 있다.
- `Beacon ranging` : 근처에 있는 비콘을 탐지하고 찾는다.
- `Compass headings` : 나침반의 가르키는 방향의 변화를 보고해준다. 

그러나 이러한 위치 기반 서비스를 활용하기 위해서는 **사용자한테 사용 권한을 물어봐야한다.** 즉, 우리가 네이버나 카카오맵에 들어가게 되면 항상 "현재 위치를 가져올 수 있도록 허가하겠습니까?" 라는 경고창이 뜨는게 바로 우리에게 위치 사용 권한을 물어보는 것이다. 앱을 처음 실행할 때 권한을 설정하는 것 외에도 `Setting(환결설정)`에서 모든 앱들의 위치 정보 허가 관련 설정을 변경할 수 있다. 

그리고 만약 사용자가 앱의 권한 설정을 중간에 변경하는 경우에는, 개발자는 CLLocationManagerDelegate 프로토콜을 채택한 location manager의 delegate 객체에 의해서 앱의 권한이 변경되었다는 이벤트를 받아 권한이 변경되었다는 사실을 알고 이를 대비할 수 있다. 

## App에 위치 서비스 적용하기

그렇다면 앱에 location service를 적용하려면 어떻게 해야할까?

먼저 CLLocationManager와 delegate를 사용해야하며 **앱에 필요한 권한 모드를 결정**해야한다. 그리고 나서 앱이 실행되면 **해당 기기가 location service를 지원하는지 검사**하고, **지원한다면 location service를 구성하고 시작**하게 된다. 

또한 이때 사용자로부터 사용자의 위치를 사용해도 괜찮냐는 권한을 물어보게 된다. 
승인을 받는다면 이제 앱은 Core Location service에 필요한 location event들을 받기 시작하게 된다. 

### 사용자의 기기가 위치 서비스를 지원하는지 확인

CoreLocation 서비스의 경우에는 모든 기기에서 지원하는 것이 아닌 **특정 기기에서만 지원**하게 된다. 

따라서 먼저 해당 기기가 CoreLocation Service를 지원하는지에 대한 검사를 진행해야한다. 만약 아래 함수의 **return 값이 거짓**이라면, 해당 기기는 해당 서비스를 지원받지 못하는 것이다.

- `SignificantLocationChangeMonitoringAvailable()` : significant-change location service를 활용해서 사용자 위치를 얻을 수 있는지 판단
- `isMonitoringAvailable(for:)` : 특별 지역이나 비콘의 범위에서 들어왔다, 나갔다를 인식할 수 있는지 판단
- `headingAvailable()` : 나침반을 사용하여 가르킬 수 있는지에 대한 여부를 판단
- `isRangingAvailable()` : 근처의 iBeacon 기기 위치를 찾을 수 있는지 판단

> 만약 앱이 하드웨어적인 성능이 부족한 기기에 대해서 서비스를 지원하지 못한다면 
**해당 서비스를 Info.plist의 필수 조건에 넣어두면 된다.** 그러면 성능이 안되는 기기들은 사용이 불가능하게된다. 

### LocationManager, Delegate 생성

CLLocationManager 클래스 객체를 생성하고 **앱의 어딘가에 strong reference 상태로 저장**해야한다. 해당 객체가 수행해야하는 업무가 끝날 때까지 location manager 객체는 `strong reference`를 유지하고 있어야만 한다. 
왜냐하면 대부분의 location manager 업무들은 **비동기적**으로 실행되기 때문에 **지역변수로 location manager를 가지고 있는것은 비효율적**이다. 

CLLocationManagerDelegate 프로토콜을 준수하는 delegate에는 커스텀 객체를 할당해야한다. 그리고 delegate는 location service들을 시작하기 전에 할당 되어야한다.

그럼 시스템은 시작한 스레드에서 해당 위치 서비스를 delegate의 메서드를 통해 호출한다. 이때 스레드는 앱의 main thread와 같이 스스로 active한 run loop를 가져야한다. 

### Delegate 메서드의 에러 처리

기기에서 location service가 실행 안 될 경우를 대비해서 우리는 항상 **delegate에 실패에 관련된 메서드들을 구현해야 한다.** 

만약 사용할 수 없는 서비스를 시작하려고 한다면 CLLocationManager 객체는 
delegate 실패에 관련된 메서드를 호출하게 된다. 예를 들어 region monitoring이 불가능하다면, 객체는 `locationManager(_:monitoringDidFailFor:withError:)`메서드를 호출한다. 

따라서 앱이 특정 location service를 사용하지 못한다면, delegate 실패 함수를 통해 이에 대응할 수 있게 UI에 알맞은 업데이트를 해주는 것이 좋다. 

### 위치 사용 권한 요청 및 권한 변경 대응 
> 위치 사용 권한 요청

앱은 기능에 따라서 필요한 권한 상태를 가져야한다. 그리고 사용자가 앱에서 위치 관련 기능에 접근할 때 위치 서비스를 사용할 수 있는 권한을 요청해야 한다. 

이러한 요청에는 다음과 같은 종류들이 있다.
- `WhenInUse`
- `Always`

> 권한이 변경되었음을 감지하는 방법

delegate의 `locationManager(_:didChangeAuthorization:)` 메서드를 구현해놓으면 location manager가 생성된 직후부터 앱의 권한 상태가 변경되면 알림을 주게 된다. 

따라서 이 메서드를 사용해서 각각의 권한 상태에 알맞은 기능을 하도록 구현하면된다. 
예를 들어서 권한이 바뀌었을 때 location service를 on/off 하도록 할 수 있을 것이다. 

### 위치 서비스를 실행하고 이벤트 수신하기

CLLocationManager 객체의 메서드를 사용하기 전에 **반드시 delegate 객체를 설정해야한다.** 
그리고 나서 아래의 사항들을 수행하면 된다. 
- 이벤트를 받기 위해서는 **적절한 CLLocationManager의 함수를 호출**해야함
- location을 받고 delegate 객체에서 관련된 업데이트를 해주어야함
- 더 이상 앱에서 위치 정보를 받을 필요가 없다면 **service를 중지해주는 CLLocationManager 메서드를 호출**해야함

사용하는 위치 서비스는 해당 서비스와 관련된 속성을 정확하게 구성해야하며, CoreLocation은 필요하지 않을 때는 **hardware를 껴서 전력 관리를 하는 것이 좋다.** 예를 들어 location event의 정확도를 1km로 설정해놓는다면, location manager는 GPS hardware를 끄고 WiFi에 의존할 수 있는 유연성을 가지게 되어 상당한 전력 절감 효과를 얻을 수 있을 것이다. 

## 요청할 위치 서비스 권한 선택하기

이제 **앱이 필요한 location data 권한**에 대해서 알아보자. <br>
앱에게는 2가지 요청할 수 있는 권한이 있다. 

- `WhenInUse` : app이 **사용되고 있는 동안**에 location service와 이벤트들을 받을 수 있다. 일반적으로 iOS 앱들은 foreground에 있거나 background에서 위치 사용 표시가 활성화 된 상태로 사용되고 있다면 `In Use` 상태에 있다고 한다. 
- `Always` : 사용자가 app이 **실행 되는지 몰라도** location service와 이벤트들을 받을 수 있다. 만약 **app이 실행되지 않았어도** 시스템이 앱을 실행시켜 event를 전달해준다. 

### WhenInUse 권한
`WhenInUse` 권한은 다음과 같은 특징들을 가진다. 

- 사용자가 **앱을 사용하는 동안 사용 가능한 모든 location service에 접근**한다. 
그리고 만약 사용자가 **앱의 사용을 멈추면, 다시 앱을 시작할 때 까지 요청들은 모두 중단**된다. 
- 만약 Xcode project에서 **Background에서도 location updates를 가능하게 설정**해놓는다면 **background 상태에 들어가도 location update**를 받아온다.
- **location notification을 앱 시작 트리거로 사용**할 수 있다. 

만약 앱이 사용자와의 상호작용에 의지할 수 있다면, 즉 권한을 사용자가 중간에 허가해 주거나 할 수 있다면, `UNLocationNotificationTrigger` 를 사용해서 특정 region에 들어갔을 때 알림을 줄 수 있다. 그리고 유저가 알림을 클릭하게 되면 시스템이 앱을 실행시키고 위치 이벤트들을 받아올 수 있다. 

이러한 방법을 통해서 **특정 순간에 사용자에게 그들의 위치를 앱에 공유할 지, 하지 않을지에 대해서 결정할 수 있게 해준다.** 

위치 서비스는 앱이 `InUse` 상태일 때  `CLAuthorizationStatus.authorizedWhenInUse` 를 가진 앱들만 사용이 가능하다. 또한 WhenInUse 권한을 지원하는 모든 플랫폼에서 앱은 다음과 같은 상황을 `InUse` 상태라고 한다. 

-  앱이 foreground에서 동작할 때
- app이 foreground에서는 사라졌지만, **이미 사용자가 요청한 현재 위치 업무에가 짧은 시간 내에 끝나는 경우**
- app이 **background location usage indicator를 보여줄 때**, 이때 iOS에서 indicator는 스크린의 맨 위에 파란색 bar나 pill이다. 


### Always 권한

Always 권한은 다음과 같은 상황에 필요하다.
- 앱이 **신속하게 처리할 필요가 없는 일**들에 대해서 업무를 자동으로 실행할 때 <br>
예를 들어 사용자가 특정 위치를 들어가면 자동적으로 휴대폰 조명을 켜야하는 경우에는
**Always** 권한이 필요할 것이다. 
- 다이어리 앱처럼 **하루 종일 수 많은 위치를 기록할 때** <br>
사용자는 **Always** 권한을 주어서 사용을 하지 않을 때에도 사용자에게 알림을
띄워서 권한을 물어보는 것보다는, 물어보지 않고 기록 하는 것을 선호할 것이다. 

물론 명심해야할 것은 `Always` 권한을 요청한다고 **사용자로부터 승인이 되는 것이 아니다.** 즉, 사용자는 자신의 위치 정보를 제공하는 것에 꺼려할 수도 있기 때문에 **항상 `WhenInUse` 권한을 선택하였을 경우에 대해서도 준비를 해야한다.** 

> 사용자가 위치 정보 권한 요청에 대해서 `WhenInUse` 권한을 선택한 경우에는 추후에 앱을 켤 때 다시 `Always` 권한에 대해서 요청을 해볼 수 있다. 반면에 사용자가 `Always` 권한을 선택한 경우에는 처음 단 한 번만 위치 정보 권한 요청을 물어본다.<br>(이후에 앱을 다시 켜도 물어보지 않는다. )

### iOS12 이전 버전의 경우
iOS 12와 그 이전에 Request Authorization에 대해서는 [해당 사이트](https://developer.apple.com/documentation/corelocation/choosing_the_location_services_authorization_to_request#3376460)로 들어가서 표를 한번 확인해보자.

## CLLocationManager

![](https://images.velog.io/images/minni/post/67783801-4996-4fe1-a8b8-a2467a171371/image.png) 

CLLocationManager는 **클래스타입**이다. <br>
앱의 **location과 관련된 이벤트들의 전달을 시작하고 멈출 수 있게 하는 객체**이다. <br>
[👉🏻 Apple Document CLLocationManager](https://developer.apple.com/documentation/corelocation/cllocationmanager)



## CLLocationManagerDelegate

![](https://images.velog.io/images/minni/post/b2a516ba-7e71-4678-afc3-730479fa6d02/image.png) 

CLLocationMangerDelegate는 **location manager객체로부터 이벤트를 받을 때 사용하는 메서드**들이다. location manager는 이 delegate의 메서드들을 앱의 location 관련된 이벤트들을 보고할 때 사용하게 된다. 

따라서 **앱의 특정 객체에 이 프로토콜을 구현하고 메서드들을 사용해서 앱을 업데이트 할 수 있다.** 예를 들면 지도에서 사용자의 위치를 표시하기 위해서 현재 위치가 필요할 수 있고, 사용자의 현재 위치 근처에서만 관련된 검색 결과를 반환할때도 사용할 수 있다. 

> location 관련된 데이터를 받을 때 항상 실패에 대해서도 
핸들링할 수 있도록 구현하는 것이 매우 중요하다.

Delegate 객체는 어떠한 서비스를 시작하기 전에 CLLocationmanager 객체의 delegate 프로퍼티에 할당해야한다.

Core Location은 **서비스를 시작한 직후에 delegate에게 캐시에 저장된 값을 보고**하고 **이후에 더 최신 값을 보고**할 수 있다. 따라서 데이터 객체를 사용하기 전에 수신한 **모든 데이터 객체의 타임스탬프를 확인하고 사용**하는 것이 좋다. 

## Info.plist
사용자에게 권한을 요청할 때, 왜 해당 요청이 필요한지에 대해서 이유를 작성해 줄 필요가 있다. 그래야 사용자도 왜 자신의 위치 정보가 필요한지에 대해서 알 수 있기 때문이다. 종종 이러한 이유를 구체적으로 적지 않거나, 대충 적어서 Apple에 의해서 출시 거부 되는 경우가 있다고 하니까 최대한 구체적으로 작성해주자. 

Xcode에서 Info.plist의 Property List Key에 `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysAndWhenInUseUsageDescription` 에 
**왜 앱이 사용자의 위치 정보에 접근 해야하는지 원인**을 작성해 줄 수 있다. 

만약 **foreground에서만 사용자의 위치에 접근하는 경우**에는
`NSLocationWhenInUseUsageDescription` 만 작성해주면 된다.

앱이 **background에서도 사용자의 위치에 접근해야하는 경우**에는
`NSLocationAlwaysAndWhenInUseUsageDescription` 를 작성해주면 된다. 


## 참고
[Apple Document Core Location](https://developer.apple.com/documentation/corelocation) <br>
[Apple Document Adding Location Services to Your App](https://developer.apple.com/documentation/corelocation/adding_location_services_to_your_app) <br>
[Apple Document Choosing the Location Services Authorization to Request](https://developer.apple.com/documentation/corelocation/choosing_the_location_services_authorization_to_request) <br>
[Apple Document CLLocationManager](https://developer.apple.com/documentation/corelocation/cllocationmanager) <br>
[Apple Document CLLocationManagerDelegate](https://developer.apple.com/documentation/corelocation/cllocationmanagerdelegate) <br>
[Apple Document NSLocationWhenInUseUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationwheninuseusagedescription) <br>
[Apple Document NSLocationAlwaysAndWhenInUseUsageLocation](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationalwaysandwheninuseusagedescription)