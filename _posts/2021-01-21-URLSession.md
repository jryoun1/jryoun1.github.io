---
title:  "[Swift] URLSession"
excerpt: "iOS에서는 네트워크 통신을 어떻게 지원하는지 알아보자"

categories:
  - Swift
tags:
  - [URLSession, URLSessionTask, URLSessionConfiguration]
last_modified_at: 2021.01.21
---
## 들어가며

iOS에서는 어떻게 네트워크 통신을 하는지에 대해서 알아보자.
이번에도 참고에 있는 관련된 공식문서들을 바탕으로 글을 작성하였습니다. 

## URL Loading System

URL Loading System은 **Foundation Framework**에 포함이 되어있다. 

> `Foundation Framework`는 데이터의 저장 및 지속, 텍스트 처리, 날짜 및 시간 계산, 정렬 및 필터링, 네트워킹을 포함한 애플리케이션과 프레임 워크의 기본 기능을 제공해준다. `Foundation`에서 정의한 클래스나 프로토콜 데이터 타입은 MacOS, iOS, WatchOS, tvOS SKD에서 사용된다. 

### URL Loading System이란
URL Loading System은 **표준 Internet protocols를 사용해 URL과 상호작용하고 서버와 통신하는 시스템**을 의미한다. URL Loading System은 표준 규약인 https나 커스텀된 규약을 사용해서 **URLs에 의해서 식별되는 자원에 접근을 제공**한다. 
이때 Loading은 **비동기적으로 수행**되기 때문에 App은 여전히 사용자에 의해 반응적으로 작동해야하고, 결과로 받아오는 데이터나 에러에 대해서 처리할 수 있다. 

`URLSession` 객체를 사용해서 하나 이상의 `URLSessionTask` 객체를 생성할 수 있는데, **URLSessionTask는 데이터를 App으로 보내주거나, 파일을 다운받거나, 원격 장소에 데이터나 파일을 업로드**하는 일들을 하게 된다. 

그리고 이러한 session을 구성하기 위해서 `URLSessionConfiguration` 객체를 사용하는데, 이 객체는 **cache나 cookies의 사용 여부나 셀룰러 상태에서 연결을 허용할 지와 같은 동작을 제어**한다. 

하나의 session을 사용해서 **여러 개의 task를 생성**할 수 있다. 예를 들어 web browser가 **일반 browsing 사용과 data를 cache하지 않는 private browsing 사용**에 대해 구별해서 session을 가져야 하는 경우가 있다. 이러한 경우에는 아래의 그림이 2개의 session이 어떻게 구성되고, 여러 개의 task들을 생성하는지에 대해서 확인해 볼 수 있을 것이다. <br>
(그림에서 보면 **private browsing에는 UploadTask가 없는 것을 확인해 볼 수 있다.**)

![](https://images.velog.io/images/minni/post/452e7ac9-fd1c-49d6-b029-738b58a7b022/image.png) 

각각의 **session**은 주기적인 업데이트나 오류를 수신하기 위해서 **delegate에 연결**되어 있다. `Default delegate`의 경우에는 작성된 completion handler block를 불러주게 되고, `커스텀한 delegate` 의 경우에는 해당 block이 호출되지 않는다.

그리고 session은 background에서 동작하도록 구성하여, App이 중지되었을 때에도 시스템이 데이터를 다운로드하고 결과를 App에 전달해주기 위해서 깨우게 할 수도 있다. 

## URLSessionConfiguration

![](https://images.velog.io/images/minni/post/ad5764ac-a3e1-4b70-bcda-06c76eb31ad4/image.png) 

URLSession의 행동과 정책들을 정의하는 Configuration(구성) 객체이다. 

URLSessionConfiguration 객체는 URLSession 객체를 사용해서 
데이터를 업로드, 다운로드 할 때 행동과 정책을 정의해준다. 따라서 uploading이나, downloading을 할때는 **반드시 항상 첫번째로 해야하는 것이 바로 configuration 객체를 만드는 것**이다. 이 객체에서 URLSession에서 사용할 
**timeout 값이나, caching 정책, connection 필요조건들과 다른 정보들을 구성**해준다. 

URLSessionConfiguration 객체를 URLSession 객체 생성 전에 적절하게 구성하는 것이 매우 중요하다. 왜냐하면 Session 객체는 **구성한 configuration 설정들을 복사**하고 **이러한 설정들로 session을 설정**해주기 때문이다. 

한 번 구성되면 session 객체는 URLSessionConfiguration 객체를 통해 **수정을 하려고 해도 모두 무시**된다. 따라서 만약 **새로운 전송 정책이 필요하다면, 반드시 session configuration 객체를 업데이트하고 새로운 URLSession 객체를 만들어야한다.** 

특정 케이스에서는 session의 정책이 특정 task의 NSURLRequest 객체에 의해 정의된 정책으로 덮어질 수 있다. **session의 정책이 더 제한적이지 않는 이상** request 객체의 정의된 정책들은 다 수용될 수 있다. 예를 들어 session의 구성에서 셀룰러 네트워킹을 허용해서 안 된다고 했다면, NSURLRequest 객체는 셀룰러 네트워킹을 요청할 수 없다. 이렇게 제한적이지 않은 경우에는 request 객체에 의해 정의된 정책들은 수용가능하다.

### URLSessionConfiguration 종류

URLSession의 동작과 기능은 **session을 생성하는데 사용되는 구성에 따라서 결정**된다.

- `Shared session` : **singleton이며, 구성 객체가 없고 기본 요청에 사용**한다. 직접 만든 session보다 커스터마이즈는 할 수 없지만 요구사항이 매우 제한적인 경우에 사용하기에 좋다. `URLSession.shared`로 session에 접근할 수 있다. 
- `Default session` : Shared session과 비슷하지만, **delegate를 사용**해서 데이터를 증진적으로 얻을 수 있게 해준다. URLSessionConfiguration class의 default method를 호출해서 생성한다.
- `Ephemeral session` : defualt session과 비슷하지만 **디스크에 caches, cookies, credential를 기록하지 않는다.** URLSessionConfiguration class의 ephemeral method를 호출해서 생성한다.
- `Background session` : App이 돌아가지 않는 **background에서 upload와 다운**로드를 수행한다. URLSessionConfiguration class의 backgroundSessionConfiguration method를 호출해서 생성한다. 

## URLSession

![](https://images.velog.io/images/minni/post/179ac3c8-95d6-4ec4-bbb0-e3e6fad61797/image.png) 

`URLSession` 은 네트워크 데이터 전송 업무와 관련된 그룹들을 모아둔 객체이다. 

`URLSession` 클래스와 관련된 클래스들은 **URL이 가르키는 endpoint에서 데이터를 다운로드 하거나 업로드를 하는 API를 제공**한다. 따라서 iOS에서는 앱이 중단되어있는 background 상태에서도 이러한 API를 사용해서 다운로드 할 수 있다. 그리고 `URLSessionDelegate`나 `URLSessionTaskDelegate`를 사용해서 인증이나 redirection 및 작업 완료와 같은 이벤트도 수신할 수 있다. 

App은 **하나 이상의 URLSession 객체를 생성할 수 있다.**
예를 들어 web browser를 만든다고 할 때 tab이나 window마다 session이 필요할 수도 있고, 하나의 session이 interactive하게 사용될 수 도 있다. 또한 background에서 다운로드를 위한 session이 존재할 수도 있다. 각각의 session에서 App은 특정 URL에 대한 요청을 보내는 업무들을 추가하게 된다. 

### URLSession 종류

URL session내의 task는 단일 호스트에 대한 최대 동시 연결 수, 셀룰러 네트워크를 사용할 수 있는지 여부 등과 같은 연결 동작을 정의하는 **common sessionConfiguration 객체를 공유**한다. 

URLSession에는 기본 요청에 대해 configuration 객체가 없는 **singleton shared session**이 있다. 직접 session을 생성하는 것보다는 커스터마이즈는 할 수 없지만, 요구사항이 **매우 간단한 경우에 사용하기 좋다.** 
`URLSession.shared` 를 통해서 해당 객체에 접근할 수 있고, 이외의 다른 session들의 경우 아래의 세 가지 구성 중 하나를 사용해서 URLSession을 생성한다.

- `Default session` : shared session처럼 행동하지만 **직접 구성할 수 있다.** 또한 data를 점진적으로 획득할 수 있도록 delegate를 할당할 수 있다.
- `Ephemeral session` : default session과 비슷하지만 **디스크에 caches, cookies, credential 를 기록하지 않는다.** 
- `Background session` : App이 동작하지 않는 **background에서 업로드와 다운로드를 수행**한다. 

#### ⚠️ shared session의 한계점
shared session의 한계는 [공식문서](https://developer.apple.com/documentation/foundation/urlsession/1409000-shared)에 의하면 다음과 같다고 한다. **간단한 몇 줄의 코드로 구현하여 URL의 contents에서 데이터를 받아오는 경우**에는 shared를 사용하면 된다. 이때 shared는 singleton 객체 이므로 직접 생성하는 것이 아니라 `URLSession.shared`로 접근하면 되며, delegate나 configuration 객체는 생성할 수 없다. 

delegate나 configuration 객체가 없으므로 당연히 다음과 같은 한계점이 발생하게 된다. 
- 서버로 부터 데이터를 받아올 때 data를 점차 증가시면서 받아올 수 없다.
- `connection behavior`를 defualt에서 더 이상 **커스터마이즈 할 수 없다**.
- **authentication(인증)** 을 수행하는 기능은 사용할 수 없다.
- App이 실행되지 않을 때, **background에서 download나 upload를 수행할 수 없다.**

shared session은 `URLCache`, `HTTPCookieStorage`, `URLCredentialStorage` 객체들을 공유하게 된다. 
또한 `registerClass(_:)`, `unregisterClass(_:)` 에 의해서 구성되는 
shared custom networking protocol list를 공유하게 된다. 
그리고 이러한 것들은 모두 default configuration에 기반을 두고 있다. 

따라서 shared session을 사용하는 경우에는 **cache, cookie storage, credential storage를 커스트마이징** 하는 것을 피해야한다. <br>(NSURLConnection을 사용하는 것이 아니라면!!) 

💡 결론은 만약 **cache, cookies, authentication, custom networking protocols을 사용하는 것**이라면 shared session이 아닌 **default session나 custom session을 사용**해야 한다!!

### URLSession의 결과 반환

대부분의 networking APIs처럼 **URLSession API는 매우 비동기적**이다. 
호출 방법에 따라서 아래의 두 가지 방법 중에 하나로 데이터를 앱에 반환해준다. 

- **completionHandler block을 호출하여** 결과나 error를 반환한다.
- **session delegate의 메서드를 호출**하여 데이터가 도착했음을 알려준다. 

URLSession은 **데이터를 delegate에게 전달할 뿐만 아니라**, 작업의 현재 상태에 따라 프로그래밍 방식으로 결정해야 하는지를 물어볼 수 있도록 **상태와 진행 현황 프로퍼티를 제공**한다. 

### Protocol Support

URLSession 클래스는 기본적으로 **데이터, 파일, ftp, http, https URL 체계를 지원**하며 사용자 시스템 구성 환경에 따라 proxy server나 SCOKS gateway를 지원한다. URLSession은 **HTTP/1.1 과 HTTP/2 프로토콜을 지원**한다. 

RFC 7540에 나온것처럼 HTTP/2를 지원에는 
APLN(Application-Layer Protocol Agreement)를 지원하는 서버가 필요하다.

**URLProtocol**을 상속해서 앱에서 private하게 사용할 수 있는 
URL 체계와 networking protocol들을 커스텀할 수 있다. 


### AppTransportSecurity(ATS)

iOS 9.0 과 macOS 10.11 이후는 **URLSession에 의해 만들어지는 모든 HTTP 연결에 ATS를 사용한다.**
ATS는 **HTTP connection이 HTTPS를 사용해야한다는 것을 요청한다.** 


### Foundation Copying Behavior

session과 task객체는 다음과 같은 NSCopying 프로토콜을 따른다.

- 앱이 **session이나 task 객체를 copy**하면 **동일한 객체를 가져온다.** 
- 앱이 **configuration 객체를 copy**하면, 독립적으로 수정할 수 있는 **새로운 copy가 생성**된다.❗️

### Thread Safety

URLSession API는 **thread safe**하다. 따라서 session은 자유롭게 생성될 수 있고 어떠한 thread context에서도 업무를 할 수 있다. 

Delegate 메서드들이 completion handler들을 호출하면 **작업은 자동적으로 delegate queue에 옳바르게 스케줄링 될 것이다.** 

## URLSessionTask

![](https://images.velog.io/images/minni/post/9607c185-a7bc-4f27-9622-8f3eceeb4f92/image.png) 

URLSession에서 수행하는 자료를 다운로드 받는 것과 같은 작업을 의미한다.

### URLSessionTask의 종류
- `DataTask` : NSData objects를 사용하며 **데이터를 보내고 받는다.** 
**주로 짧고 빈번하게 서버에 요청하는 경우에 사용**된다. (background 지원 x)
- `UploadTask` : DataTask와 비슷하지만, **file 형태로 데이터를 보내거나** 
App이 실행되지 않을 때 background upload를 지원한다. 
- `DownloadTask` : 파일의 형태에서 데이터를 찾고, App이 실행되지 않는 background upload, download를 지원한다. 
- `WebSocketTask` : RFC 6455에 정의된 웹 소켓 규약을 사용하여 TCP와 TLS 간의 메세지를 교환한다. 

### URLSessionTask 상태
- `suspended` : task가 생성될 때 초기 상태
- `active` : task가 진행 중이 상태
- `canceled` : task가 취소 요청 받았거나, 취소된 상태
- `completed` : task가 완료된 상태

### URLSessionTask 생성 함수
URLSessionTask 클래스는 URLSession 클래스를 상속 받은 클래스이다. 
Task는 항상 session의 일부분이기 때문에 **URLSession 객체를 통해서 task를 생성하는 메서드를 호출하여 Task를 생성할 수 있다.** 그리고 그러한 method에는 다음과 같은 메서드들이 있다. 

- `dataTask(with: )` : DataTask 객체를 생성하기 위해 호출하며, 이때 Data tasks는 **자원을 요청하고, 서버의 응답인 하나 이상의 NSData 객체를 메모리에 반환**해준다. Default, Ephemeral, Shared session에서는 지원하지만, Background session에서는 지원하지 않는다.
- `uploadTask(with: ) ` : UploadTask 객체를 생성하기 위해 호출하며, Upload tasks는 DataTask와 비슷하지만, **서버의 응답을 찾기 전에 데이터를 업로드 할 수 있는 request body를 제공**하는 점이 다르며 Background session에서도 지원한다. 
- `downloadTask(with: ) ` : DownloadTask 객체를 생성하기 위해 호출하며, DownloadTask는 **디스크 안에 파일에 직접적으로 자료를 다운로드**해준다. 모든 session에서 지원한다. 
- `streamTask(withHostName:port:), streamTask(with:) ` : StreamTask 객체를 생성하기 위해 호출하며, StreaTask는 호스트 이름과 포트 또는 네트워크 서비스 객체에서 TCP/IP 연결을 설립해준다. 

이러한 task를 작성한 다음에는 `resume()` 메서드를 아래와 같이 호출해야한다. 
```swift
let task = URLSession.shared.dataTask(with: url) { 
    //completionHandler      
}
task.resume()
```
task는 처음 생성될 때 suspend 상태를 가지게 되는데, 이때 `resume()` 메서드를 통해서 suspend 상태의 task를 실행시키게 된다. 그러면 **session은 strong reference를 해당 task가 끝나거나 실패할 때까지 유지해준다.** 

⚠️ 모든 task 프로퍼티들은 key-value 검색을 지원한다. 

## SessionDelegate 사용하기

session안에 있는 task들은 또한 common delegate 객체를 공유하는데, 
다양한 이벤트들이 발생하였을 때 정보를 획득하거나 제공하기 위해서 delegate를 구현해야한다. 

또한 아래의 경우들도 필수로 구현해야한다. 
- 인증에 실패하였을 때
- 서버로 부터 데이터가 도착했을 때
- 데이터가 cahcing을 이용 가능해 졌을 때

만약 delegate가 제공하는 **이러한 특징들이 필요없다면 session을 생성할 때 nil로 설정하여 해당 기능을 제외하고 사용**할 수 있다.

⚠️ session 객체는 App이 종료되거나 session이 명시적으로 무효화 될 때까지 **delegate와 strong reference를 유지한다.** 따라서 만약 session을 무효화시키지 않는다면 app이 종료 될 때까지 **메모리에 낭비가 발생***할 수 있다.❗️

## URLRequest

![](https://images.velog.io/images/minni/post/73fa1564-9fe8-44ba-a880-a6ab3c85a192/image.png) 

URLReqeust는 프로토콜 또는 URL 체계와 **독립적인 URL load 요청이다.** 

URLRequest는 load request의 2가지 중요 프로퍼티인 **load를 요청할 URL과 로드하는데 사용되는 정책**을 캡슐화한다. HTTP나 HTTPS 요청의 경우 URLRequest에는 **HTTP 메서드인 GET, POST와 같은 메서드들과 HTTP 헤더를 포함할 수 있다.** 

URLRequest는 **단지 요청에 대한 정보만 나타낸다**. URLSession과 같은 다른 클래스를 사용하여 요청을 서버로 보낼 수 있다. **Swift 코드를 작성할 때** NSURLRequest나 NSMutableURLRequest 클래스보다 **이 구조를 더 선호한다.** 
특정 header 필드는 예약되어있으므로 [Reserved HTTP Headers](https://developer.apple.com/documentation/foundation/nsurlrequest#1776617)문서를 확인해 보는 것이 좋다. 

## URLResponse

![](https://images.velog.io/images/minni/post/097bc6cb-9b9b-4031-b094-21bcdb4403e6/image.png) 

URLResponse는 프로토콜이나 URL체계에 독립와는 **독립적인 URL load 요청에 대한 metadata와 관련된 응답**이다. 

HTTPURLResponse는 URLResponse를 상속한 것으로 일반적으로 자주 사용되며, 
URLResponse의 객체는 **HTTP URL 로드 요청에 대한 응답을 나타내고 응답 헤더와 같은 추가적인 프로토콜 정보를 저장**한다. HTTP request을 할 때마다 **반환되는 URLResponse 객체는 사실 HTTPURLResponse 클래스의 객체**이다. 

⚠️ URLResponse 객체는 실제로는 **URL 내용을 나타내는 바이트가 없다.** 
대신에 데이터는 요청을 시작하는데 사용되는 방법과 클래스에 따라서 **한 번에 한 개씩 delegate 호출을 통해서 반환되거나 요청이 완료되면 전체 데이터가 반환**되는 것이다. 


## 메모리로 웹사이트의 데이터 가져오기

URLSession로 부터 생성된 data task를 이용해서 memory로 바로 데이터를 받아보자.

원격 서버들과 규모가 작은 소통을 할 때는 **URLSessionDataTask** 클래스를 사용해서 **메모리에 바로 응답 데이터를 받아올 수 있다.** DataTask는 web 서비스의 endpoint를 호출하는 것과 같은 용도로 적합하다. <br>(URLSessionDownloadTask 클래스는 **파일시스템으로 바로 데이터를 저장**할 때 사용). 

URLSession 객체를 사용해서 task를 생성할 수 있다. 만약 간단한 일이라면 URLSession 클래스의 shared 객체를 사용할 수 있다. **그러나 만약 delegate를 사용해서 콜백을 받고 소통하려면** shared 객체를 사용하기 보다는 **session을 생성해야한다.** 

**URLSessionConfiguration 객체를 사용해서 session을 생성**하고, 
**URLSessionDelegate나 서브 프로토콜을 구현하여 사용**할 수 있다. 
Session은 **여러 개의 task에 재활용 될 수 있으며**, 각각 특별하게 configuration을 해야할 필요가 있다면 **session을 생성하고 프로퍼티로서 저장할 수 있다.** 

⚠️ 이때 필요한 session보다 더 많은 session을 만들지 않도록 주의해야한다.예를 들어, App에 똑같이 구성된 session을 필요로 하는 여러부분이 있다면 하나의 session을 만들고 다 같이 공유해서 사용할 수 있도록 해라.

한 번 session이 생성되면, **dataTask() 메서드를 사용해서 data task를 생성**해라. **Task는 중지된(suspended) 상태로 생성**되며, **resume()이 호출되면 시작**된다. 

### CompletionHandler로 결과 받기

Data를 fetch하는 **가장 쉬운 방법은 바로 completion handler를 사용하는 data task를 생성하는 것**이다. 이를 사용하면 task는 server로부터의 응답과, 데이터, 그리고 발생가능한 에러를 구현한 completionHandler block으로 전달해준다. 

![](https://images.velog.io/images/minni/post/e6507dbc-3092-4c97-9e59-a412d1a47c88/image.png) 

위의 그림을 보면 session과 task의 관계와 결과가 어떻게 completionHandler로 가는지에 대해서 알 수 있다. 

completionHandler를 사용하는 dataTask를 생성하는 것은 **URLSession의 dataTask(with:) 호출**하면 된다. 이때 completion handler는 아래의 3가지에 대해서 구현해야한다. 

- `error` : nil인 것을 검증해야한다. 만약 전송 에러가 발생하면 핸들링 해야한다. 
- `response` : 검사해서 status code가 성공인지 확인하고, MIME 타입이 기댓값인지 확인해야한다. 그렇지 않은 경우 핸들링이 필요하다. 
- `data` : 필요하다면 data 객체를 사용하면 된다. 

아래의 코드에서 `startLoad()` 함수는 URL의 컨텐츠를 fetching한다. 
URLSession 클래스의 shared 객체를 사용해서 data task를 만들고, 이 data task가 결과를 completionHandler에게 전달해준다. 

local과 server 에러에 대해서 검사를 한 다음에, 이 핸들러는 data를 string으로 변환하고 WKWebView 아웃렛을 채우는데 사용된다. 물론 App에서는 fetch 된 데이터를 데이터 모델로 파싱해주는 용도로도 사용이 가능하다. 

```swift
func startLoad() {
    let url = URL(string: "https://www.example.com/")!
    let task = URLSession.shared.dataTask(with: url) { data, response, error in 
        // 전송 에러가 있는지 검사
        if let error = error {
            self.handleClientError(error)
            return 
        }
        // 상태 코드가 200~299사이에 있는 성공과 관련된 코드인지 확인
        guard let httpResponse = response as? HTTPURLRespionse,
             (200...299).contains(httpResponse.statusCode) else {
             self.handleServerError(response)
             return 
        }
        // 결과 타입이 기대한 타입과 같은지 체크하고 맞다면 UI 관련된 작업은 dispatchQueue.main에서 실행
        if let mimeType = httpResponse.mimeTpye, mimeType == "text/html",
           let data = data,
           let string = String(data: data, encoding: .utf8) {
                DispatchQueue.main.sync {
                     self.webView.loadHTMLString(string, baseURL: url)
                }
           }                                       
     }
     task.resume()
}
```

⚠️ completionHandler의 경우에는 해당task를 만든 queue가 아닌 다른GCD queue에서 호출된다. 따라서 webView를 업데이트와 같이 데이터나 오류를 사용하여 
**UI를 업데이트 하는 작업들은 main queue에 명시적으로 작성**해애한다. 

### Delegate로 결과 받기

작업이 진행될 때 **작업 활동에 대한 접근 수준을 높이려면** dataTask를 생성할 때 completionHandler를 제공하는 것보다 **delegate를 session에 제공하는 것이 더 좋다.**  

![](https://images.velog.io/images/minni/post/d03da2a2-9681-443a-b969-99a1afab2a79/image.png) 

위의 그림과 같이 접근을 한다면 **data의 일부**는 **전송이 완료되거나 오류와 함께 실패할 때까지** URLSessionDataDelegate의 **`urlSession(_:dataTask:didReceive:) `메서드에 제공**이 된다. 또한 delegate는 전송이 진행됨에 따라 다른 종류의 이벤트들도 수신한다. 

**delegate를 사용하는 방법을 사용할 때**는 URLSession 클래스의 shared 객체를 사용하는 것보다는 **URLSession 객체를 직접 만드는 것이 더 좋다.** 직접 새로운 session을 생성하는 것이 아래의 코드와 같이 해당 클래스 자체에 session delegate를 설정할 수 있다. 

URLSessionDelegate, URLSessionTaskDelegate, URLSessionDataDelegate 프로토콜들 중에서 하나 이상의 **delegate protocol을 해당 클래스에 선언하고 구현**한다. 그리고 **URLSession 객체를 `init(configuration:delegate:delegateQueue:)` 이니셜라이져를 사용해서 생성**하면된다. 또한 이때 initializer에서 configuration을 커스터마이즈 할 수도 있다.

예를 들어 `waitsForConnectivity`를 true로 설정해놓을 수 있다. 
그렇게 되면 session은 연결이 불가능할 때 즉시 실패하지 않고, 적합한 연결이 될때까지 기다릴 것이다. 

```swift
private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: configuration, 
                     delegate: self, delegateQueue: nil)
}()
```

아래의 코드는 위에서의 startLoad()가 바로 위에서 구현한 session을 사용해서 data task를 시작하고, data와 error를 받기 위해서 delegate를 사용하는 것을 보여줄 것이다. 

아래 3개의 목록은 delegate callback()을 구현해준다. 
- `urlSession(_:dataTask:didReceive:completionHandler:) `: 응답이 **HTTP 성공 상태코드와 기대한 MIME 타입인지 검증**한다. 
만약 둘 중 하나라도 다르다면 task는 cancel되고 그렇지 않으면 진행된다. 
- `urlSession(_:dataTask:didReceive:)` : task에 의해서 받은 각각의 **Data 객체들을** 가지고 **receiveData라고 불리는 buffer에 넣어준다**. 
- `urlSession(_:task:didCompleteWithError:)` : 먼저 **전송 수준에서의 에러가 발생했는지 확인**한다. 만약 에러가 발생하지 않았다면 receiveData buffer에 들어있는 Data를 String으로 변환해주고 webView 컨텐츠에 표시해준다. 

```swift
var receivedData: Data?

func startLoad() {
    loadButton.isEnable = false
    let url = URL(string: "https://www.example.com/")!
    receivedData = Data()
    let task = session.dataTask(with: url)
    task.resume()
}

//delegate method
// HTTP 상태 코드와 MIME 타입 검증
func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse, 
               completionHandler: @escaping (URLSession.ResponseDispositon) -> Void) {
     guard let httpResponse = response as? HTTPURLRespionse,
             (200...299).contains(httpResponse.statusCode), 
             let mimeType = response.mimeType,
             mimeType == "text/html" else {
             completionHandler(.cancel)
             return 
        }
        completionHandler(.allow)
}

// task에 의해서 받은 data를 receivedData라는 변수에 저장
func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
     self.receivedData?.append(data)
}

// 전송수준에서 에러가 있으면 핸들링하고 그렇지 않으면 웹뷰 업데이트
func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
     DispatchQueue.main.async {
         self.loadButton.isEnable = true
         if let error = error {
             handleClientError(error)
         } else if let receivedData = self.receivedData,
             let string = String(data: receivedData, encoding: .utf8) {
             self.webView.loadHTMLString(string, baseURL: task.currentRequest?.url)
         }
     }
}
```

위에서 구현한 delegate 프로토콜 외에도 다양한 delegate 프로토콜들은 인증에 관련된 문제나, redirect 및 특수한 케이스들을 해결할 수 있다. 

## 참고

[Apple Document Foundation](https://developer.apple.com/documentation/foundation) <br>
[Apple Document URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system) <br>
[Apple Document URLSession](https://developer.apple.com/documentation/foundation/urlsession) <br>
[Apple Document URLSessionConfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration) <br>
[Apple Document URLSessionTask](https://developer.apple.com/documentation/foundation/urlsessiontask) <br>
[Apple Document URLRequest](https://developer.apple.com/documentation/foundation/urlrequest) <br>
[Apple Document URLResponse](https://developer.apple.com/documentation/foundation/urlresponse) <br>
[Apple Document Fetching Website Data into Memory](https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory) <br>
[Apple Document Type Property shared URLSession](https://developer.apple.com/documentation/foundation/urlsession/1409000-shared) <br>

