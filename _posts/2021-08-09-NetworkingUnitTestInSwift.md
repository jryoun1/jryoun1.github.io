---
title:  "[Swift] Unit Test Networking"
excerpt: "서버와 상호작용하는 작업들을 단위 테스트 하는 방법"

categories:
  - Swift
tags:
  - [Mock, Dependency Injection, ]
last_modified_at: 2021.08.09
---

## 들어가며
대부분의 앱은 networking을 하게 된다. 그렇기에 API 추가가 쉽고 변경이 용이한 네트워킹 모듈을 설계하고 개발하는 것이 매우 중요하다. 

그리고 이렇게 개발한 네트워킹 모듈은 test가 가능해야 한다. 하지만 실제로 네트워크 호출이 이뤄진다면, 호출에 따른 결과 값을 예상해야한다. 그리고 이때 네트워크 호출에 따라 결과 값은 **네트워크의 상황** 혹은 **서버에 저장되어 있는 데이터**에 따라 매번 달라질 수 있다.

즉, 해당 네트워킹 모듈만 가지고 테스트를 하는 것이 아닌, **네트워크 상황에 “의존적”**이게 된다는 것이다. 이는 항상 같은 결과 값을 보장하지 않기 때문에 유닛테스트 작성 시 문제가 발생하게 된다. 

따라서 이러한 문제를 해결하기 위해 기존에 적용하는 방법과 최근에 Apple에서 소개한 방법에 대해서 알아보도록 하자.

## Test Double (테스트 더블)
네트워킹을 유닛 테스트 하는 방법에 대해서 소개하기 전에 먼저 Test Double이 무엇인지 알 필요가 있다. 
> 테스트를 진행하기 어려운 경우 **이를 대신해 테스트할 수 있도록 만들어주는 객체** <br>예를 들면 db로부터 조회한 값을 가져와서 연산하는 로직을 테스트하려고 한다. 이때 항상 db의 영향을 받게 되므로 db에 상황에 따라 다른 결과를 반환할 수 있다. 따라서 이렇게 테스트하려는 객체와 연관된 객체의 사용이 어렵고 모호할 때 대신해 주는 객체를 `테스트 더블`이라고 한다. 

이러한 Test Double에는 `Dummy`, `Fake`, `Stub`, `Spy`, `Mock`이 있다. <br>이에 대한 자세한 내용은 [여기](https://woowacourse.github.io/tecoble/post/2020-09-19-what-is-test-double/)를 참고하면 된다.

### Mock
> 호출에 대한 예상하는 결과를 받을 수 있도록 미리 프로그래밍 된 오브젝트 

위에서 언급한 테스트 더블 중에서 `Mock`을 사용해서 네트워킹을 유닛 테스트 진행할 것이다. 

## Dependency Injection(의존성 주입)
> 표준을 정의할 수 있고, 정의된 표준을 바탕으로 같은 설계를 하게 해주는 것

### 의존성 주입의 장점
- 재사용성 증가
- 테스트에 용이
- 코드 단순화
- 종속적인 코드의 수 감소 (구성 요소의 종속성 감소) -> 변경 용이
- 결합도(coupling)은 낮추면서 유연성과 확장성은 향상
- 객체간의 의존관계를 설정가능

### 용어 이해

#### 의존성
**의존성**은 아래와 같이 B 클래스에서 A 클래스를 B 클래스 내부 변수를 사용하게 될 때, **B 클래스는 A 클래스에 의존 관계가 생기게 된다.**
```swift
class AClass {
    var number: Int = 1
}

class BClass {
    var value = AClass()
}

let bClass = BClass()
print(b.value.number) // 1
```

#### 주입
**주입**은 내부가 아니라 **외부에서 객체를 생성해서 넣어주는 것**을 의미한다. 
```swift
class CClass {
    var number: Int
    
    init(number: Int) {
        self.number = number
    }

    func setNumber(number: Int) {
        self.number = number
    }
}

let cClass = CClass(number: Int(3))
print(cClass.number) // 3

cClass.setNumber(number: Int(7))
print(cClass.number) // 7
```
위 코드처럼 3과 7을 외부에서 생성하여 함수를 통해서 넣어주는 것을 **주입**한다고 한다. 

#### 의존성 주입
**의존성**과 **주입**을 합친 의존성 주입은 내부에서 만든 변수를 외부에서 넣어주게 하는 것이다.  
```swift
class AClass {
    var number: Int
}

class BClass {
    var value: AClass

    init(value: AClass) {
        self.value = value
    }
}

let bClass = BClass(value: AClass())
print(bClass.value.number)
```
즉, 위와 같이 클래스 생성에서 주입을 하거나, 함수를 이용해서 외부에서 의존성 있는 클래스의 오브젝트를 넣어주면 된다.

⚠️ 하지만 단지 의존성을 주입하는 것만으로는 의존성 주입이라고 하지는 않는다. 

#### 의존성 분리
의존성 주입의 경우에는 의존성을 분리시켜 사용하는 반면, 의존성 분리는 **의존 관계 역전의 원칙으로 의존 관계를 분리**시켜준다. 

즉, Swift에서는 **protocol을 사용하여** 상위 계층이 하위 계층에 의존하는 관계를 역전시켜서 하위 계층의 구현으로부터 독립하게 만들어주는 것이다. 
```swift
protocol DependencyInjectionProtocol: AnyObject {
    var number: Int { get set }
}

// 프로토콜에 의존 관계가 있는 있는 클래스
class AClass: DependencyInjectionProtocol {
    var number = 1
}

// AClass와 의존 관계가 있는 클래스
class BClass { 
    var value: DependencyInjectionProtocol

    init(value: DependencyInjectionProtocol) {
        self.value = value
    }
}

// 외부에서 AClass를 BClass에 주입하는데 
// 이때 제어의 주체(DependencyInjectionProtocol)는 외부에 있다 ⭐️
let bClass = BClass(value: AClass())
print(bClass.value.number)
```
위와 같은 코드가 의존 관계가 독립이 되는 상황이며, 이때 프로토콜에 number가 없다면 에러가 발생하게 된다. 즉, 제어의 주체가 프로토콜에게 있다는 것이다. 

다시 본론으로 돌아와서 네트워킹 작업에서 실제로 통신을 하는 `Product code`와 네트워크가 연결되지 않아도 동작할 수 있는 `Test code`에서 **네트워크를 호출하는 관련 타입(URLSession, URLProtocol)을 공통 프토토콜(ULRSessionProtocl, MockURLProtocol)로 채택**하고, **구체적인 구현부를 구현할 때 변화를 준다면** 기존의 URLSession, URLProtocl에게 있던 의존성이 새로 생성한 프로토콜로 의존성이 넘어가게 된다. 따라서 자신이 구현한 mock을 사용해서 의존성을 주입하여 네트워크가 없을 때에도 유닛 테스트를 진행할 수 있도록 하는 것이다. 

## 1️⃣ URLSession protocol 사용하는 방법
URLSession을 사용하는 network layer에서 unit test를 진행하는 방법을 찾아보면 대부분이 이 방법을 사용한다. 

1. 새로운 Protocol 생성
2. URLSession을 extension하여 생성한 프로토콜을 따르게 함
3. Test할 때 1번의 프로토콜을 따르는 mock class를 생성하여 의존성 주입

따라서 URLSession을 사용하여 네트워킹을 진행하는 실제 코드(Product Code)와 유닛 테스트에서 네트워크가 연결되지 않은 상황에서도 테스트할 수 있도록 작성한 코드(Test Code)사이에 URLSessionProtocol을 구현하여 이를 채택하고, 구체적인 구현부만 변화를 주어 해당 프로토콜로 의존성을 주입하는 방법이다. 

기존의 다음과 같은 네트워킹을 진행하는 클래스가 있다고 하자.
```swift 
final class NetworkService {
    private let session: URLSession

    func load(url: URL, 
    completionHandler: @escaping (Data?, Error?) -> Void) {
        var request = URLRequest(url: url)
        request.httpMethod = "GET"

        let task = session.dataTask(with: request) { (data,response, error) in 
            completionHandler(data, error)
        }
    }
}
```

### 새로운 Protocol 생성
```swift
  protocol URLSessionProtocol { 
      func dataTask(with request: URLRequest, 
      completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) 
      -> URLSessionDataTaskProtocol
  }

  protocol URLSessionDataTaskProtocol { func resume() }
```
`URLSessionProtocol`과 `URLSessionDataTaskProtocol`을 구현한다.
- 기존의 `URLSession`의 dataTask()는 `URLSessionDataTask`를 반환한다.
- `URLSessionProtocol`은 dataTask() 메서드를 가지며 `URLSessionDataTaskProtocol`을 반환하도록 한다.
- `URLSessionDataTask`는 resume()을 실행하므로, `URLSessionDataTaskProtocol`도 resume() 메서드를 가지게 한다.

### 기존 것 extension하여 생성한 Protocol 채택
```swift
extension URLSession: URLSessionProtocol {
      func dataTask(with request: URLRequest, 
      completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) 
      -> URLSessionDataTaskProtocol {
          return dataTask(with: request, 
                        completionHandler: completionHandler) as URLSessionDataTask
      }
  }

// 기존의 URLSessionDataTask의 resume() 수행하도록함
extension URLSessionDataTask: URLSessionDataTaskProtocol {} 
```

### 테스트 코드에 mock class 생성
```swift
// MockURLSession 구현하여 test에서는 URLSession 대신 사용
final class MockURLSession: URLSessionProtocol {
    var mockURLSessionDataTask = MockURLSessionDataTask()
    var mockData: Data?
    var mockError: Error?
    private (set) var mockURL: URL?
    
    func successHttpURLResponse(request: URLRequest) 
    -> URLResponse {
        return HTTPURLResponse(url: request.url!, 
                            statusCode: 200, 
                            httpVersion: "HTTP/1.1", 
                            headerFields: nil)!
    }
    
    func dataTask(with request: URLRequest, 
                completionHandler: @escaping DataTaskResult) 
                -> URLSessionDataTaskProtocol {
        mockURL = request.url
        completionHandler(mockData, 
                        successHttpURLResponse(request: request), 
                        mockError)

        return mockURLSessionDataTask
    }
}

// MockURLSessionDataTask 구현
final class MockURLSessionDataTask: URLSessionDataTaskProtocol {
    private (set) var resumeWasCalled = false
    
    func resume() {
        resumeWasCalled = true
    }
}
```
이렇게 테스트 코드에 mock class들을 생성하여 test를 할 때는 실제 `URLSession`, `URLSessionDataTask`가 아닌 mock을 사용하도록한다. 
그리고 기존의 `NetworkService` 클래스를 아래와 같이 수정해준다. 
```swift
final class NetworkService {
    // session의 타입과 init() 함수가 바뀐 부분
    private let session: URLSessionProtocol

    init(session: URLSessionProtocol) {
        self.session = session
    }

    func load(url: URL, 
    completionHandler: @escaping (Data?, Error?) -> Void) {
        var request = URLRequest(url: url)
        request.httpMethod = "GET"

        let task = session.dataTask(with: request) { (data,response, error) in 
            completionHandler(data, error)
        }
    }
}
```
결국 해당 네트워킹 작업을 어떠한 `URLSessionProtol`을 주입하느냐에 따라서 네트워크의 유무에 따른 코드를 작성할 수 있게 되는 것이다. 

### 실제 테스트 작성
```swift
import XCTest
@testable import NetworkingUnitTest
final class NetworkingUnitTest: XCTestCase {
    var networkService: NetworkService!
    let session = MockURLSession()

    override func setUpWithError() throws {
        super.setUpWithError()
        networkService = NetworkService(session: session)    
    }

    func testGetRequestURL() {
        guard let url = URL(string: "https://[serverURL]") else { return }

        networkService.load(url: url) { (data, response) in 
            // 
        }

        XCTAssertEqual(session.mockURL, url)
    }

    func testResumeCalled() {
        let dataTask = MockURLSessionDataTask()
        session.mockDataTask = dataTask
        guard let url = URL(string: "https://[serverURL]") else { return }

        networkService.load(url: url) { (data, response) in 
            // 
        }

        XCTAssertTrue(dataTask.resumeWasCalled)
    }

    override func tearDownWithError() throws {
        super.tearDownWithError()
        networkService = nil
    }
}
```
다음과 같이 실제로 네트워크가 없어도 유닛 테스트를 진행할 수 있게 된다. 

## 2️⃣ URLProtocol 사용한 방법
Apple에서는 [WWDC18 Testing Tips and Tricks](https://developer.apple.com/videos/play/wwdc2018/417/)에서 URLProtocol을 사용해 네트워크 요청을 가로채는 방법을 소개하여 더 이상 기존의 위에서 접근한 방식을 사용하지 않아도 된다. 

기존의 방법에 비해서 이 방법의 장점은 바로 **기존 프로젝트에서의 변경을 최소화하면서 사용할 수 있다는 것**이다. 

사용하는 방법은 네트워크 작업을 수행하는 클래스에 URLSession 의존성을 주입하여, 유닛 테스트를 할 때는 URLSession을 자신만의 MockURLProtocol로 구성(configuration)하는 것이다.

### URLProtocol 클래스 상속하기
> URLProtocol 클래스는 추상(abstract)클래스로 프로토콜별 URL 데이터 load를 처리한다

```swift
class MockURLProtocol: URLProtocol {
  
  // 매개변수로 받는 request를 처리할 수 있는 프로토콜인지 검사하는 함수
  override class func canInit(with request: URLRequest) -> Bool {
    return true
  }

  // Request를 canonical(표준)버전으로 반환하는데, 
  // 거의 매개변수로 받은 request를 그대로 반환
  override class func canonicalRequest(for request: URLRequest) -> URLRequest {
    return request
  }

  override func startLoading() {
      // test case 별로 mock response를 생성하고, 
      // URLProtocolClient로 해당 reponse를 보내는 부분
  }

  override func stopLoading() {
      // request가 취소되거나 완료되었을 때 호출되는 부분
  }
}
```
request 요청을 시작하면, 시스템 내부적으로 해당 request를 처리할 수 있는 등록되어 있는 프로토콜 클래스가 있는지 검사하고, 만약 클래스가 있다면 네트워크 작업을 완료할 책임을 해당 클래스에게 부여한다. 

따라서 여기서 network layer를 가로챌 수 있다. `URLProtocol`을 extension하여 mock class를 생성하고, 유닛 테스트에 필요한 모든 메서드를 재정의하면 된다. <br>따라서 아래와 같이 `startLoading()`메서드를 재정의한다. 
```swift
// 1️⃣ request를 받아서 mock response(HTTPURLResponse, Data?)를 넘겨주는 클로저 생성
static var requestHandler: ((URLRequest) throws -> (HTTPURLResponse, Data?))?

override func startLoading() {
  guard let handler = MockURLProtocol.requestHandler else {
    fatalError("Handler is unavailable.")
  }
    
  do {
    // 2️⃣ 받은 request를 매개변수로 전달하며 handler를 호출하고, 
    // 반환 되는 response와 data를 저장
    let (response, data) = try handler(request)

    // 3️⃣ 저장한 response를 client에게 전달
    client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
    
    if let data = data {
      // 4️⃣ 저장한 data를 client에게 전달
      client?.urlProtocol(self, didLoad: data)
    }
    // 5️⃣ request가 완료되었음을 알린다
    client?.urlProtocolDidFinishLoading(self)
  } catch {
    // 6️⃣ 만약 handler가 request를 받아서 에러가 발생한다면 발생한 에러를 알린다
    client?.urlProtocol(self, didFailWithError: error)
  }
}
```

### API 설정
테스트를 위해서 아래와 같은 타입과 클래스를 생성한다. 
```swift
struct Item: Decodable {
    let id: Int
    let title: String
    let descriptions: String
    let price: Int
}

class ItemDetailAPI {
    let urlSession: URLSession

    init(urlSession: URLSession = .shared) {
        self.urlSession = urlSession
    }

    func loadItemDetail(completion: @escaping (_ result: Result<Item, Error>) -> Void) {
        let url = URL(string: "https://[serverURL]")!
        let dataTask = urlSession.dataTask(with: url) { (data, response, error) in 
            do {
                if let _ = error {
                    completion(Result.failure(.error))
                }

                guard let httpResponse = response as? HTTPURLResponse,
                      (200...299).contains(httpResponse.statusCode) else {
                    return completion(Result.failure(APIResponseError.network))
                } 

                guard let data = data else {
                    return completion(Result.failure(APIResponseError.unwrapping))
                }
                
                if let responseData = data, 
                    let object = try? JSONDecoder().decode(Item.self, from: responseData) {
                        completion(Result.success(object))
                } else {
                    completion(Result.failure(APIResponseError.parsing))
                }
            } catch {
                completion(Result.failure(error))
                }
            }
        dataTask.resume()
    }
}
```
이때 `ItemDetailAPI` 클래스의 init() 메서드에서 **기본 값으로 shared instance를 설정**해주어 **기존의 Product Code는 수정하지 않게 만들어준다**. 그리고 또한 이 부분에서 Test code에서 MockURLProtocol로 구성한 URLSession 을 의존성 주입해주는 부분이다. 

### 실제 테스트 작성
```swift
class ItemAPITest: XCTestCase {
    var itemDetailAPI: ItemDetailAPI!
    var expectation: XCTestExpectation!
    let apiURL = URL(string: "https://[serverURL]")!
  
    override func setUpWithError() throws {
        super.setUpWithError()
        let configuration = URLSessionConfiguration.ephemeral
        configuration.protocolClasses = [MockURLProtocol.self]
        let urlSession = URLSession(configuration: configuration)
    
        itemDetailAPI = ItemDetailAPI(urlSession: urlSession)
        expectation = expectation(description: "Expectation")
    }

    func testSuccessResponse() {
        let id = 5
        let title = "Macbook Pro 2019"
        let descriptions = "2019 맥북 프로 스페이스 그레이 색상입니다."
        let price = 365000
        let mockJsonData = """
        {
            "id": "\(id)",
            "title": "\(title)", 
            "descriptions": "\(descriptions)",
            "price": "\(price)"
        }
        """.data(using: .utf8)!
        
        MockURLProtocol.requestHandler = { request in 
            guard let url = request.url, url == self.apiURL else {
                throw APIResponseError.request
            }
        
            let response = HTTPURLResponse(url: self.apiURL, 
                                            statusCode: 200,
                                            httpVersion: nil,
                                            headerField: nil)!
            return (response, mockJsonData)
        }

        itemDetailAPI.loadItemDetail { result in 
            switch result {
            case .success(let item):
                XCTAssertEqual(item.id, id)
                XCTAssertEqual(item.title, title)
                XCTAssertEqual(item.descriptions, descriptions)
                XCTAssertEqual(item.price, price)
            case .failure(let error):
                XCTFail("Error occured: \(error)")
            }
            self.expectation.fulfill()
        }
        wait(for: [expectation], timeout: 1.0)
    }

    func testParsingFailure() {
        let data = Data()
        MockURLProtocol.requestHandler = { request in
            let response = HTTPURLResponse(url: self.apiURL, 
                                            statusCode: 200, 
                                            httpVersion: nil, 
                                            headerFields: nil)!
            return (response, data)
        }

        itemDetailAPI.fetchPostDetail { (result) in
            switch result {
            case .success(_):
                XCTFail("Success response was not expected.")
            case .failure(let error):
                guard let error = error as? APIResponseError else {
                XCTFail("Incorrect error received.")
                self.expectation.fulfill()
                return
                }
            XCTAssertEqual(error, APIResponseError.parsing)
            }   
        self.expectation.fulfill()
        }
        wait(for: [expectation], timeout: 1.0)
    }

    override func tearDownWithError() throws {
        super.tearDownWithError()
        itemDetailAPI = nil
    }
}
```
위와 같이 테스트 코드를 작성하여 성공하는 테스트 케이스와 실패하는 테스트 케이스를 작성할 수 있다. 

## Alamofire
실제 많이 사용하는 서드파티 라이브러리들에서는 mock을 사용하여 테스트를 하기 어려운 경우가 많다. 그러나 Alamofire는 URLSession과 같이 `SessionManager`를 사용하여 네트워크 호출을 관리한다. 

이때 URLSessionConfiguration을 사용하여 URLSession을 구성할 수 있다. 따라서 URLProtocol을 사용하는 방법을 사용한다면 단지 아래의 코드와 같이 SessionManager의 의존성을 구현한 class에 주입하면 된다. 
```swift
let configuration = URLSessionConfiguration.default
configuration.protocolClasses = [MockURLProtocol.self]
let sessionManager = SessionManager(configuration: configuration)
```
따라서 이를 통해서 네트워킹을 하는데 많이 사용하는 서드파티 라이브러리인 Alamofire를 사용할 때에도 유닛 테스트를 할 수 있게 된다.  

## 마치며
이번 포스팅을 작성하면서 기존에 네트워킹 작업을 테스트하는 방법에서, 지난 포스팅인 [WWDC18 Testing Tips and Tricks 정리]({{site.url}}{{site.baseurl}}/wwdc/TestingTipsTricks/)에서 소개한 방법을 사용하여 네트워킹 작업을 테스트 하는 방법을 잘 이해할 수 있게 되었다. 

또한 테스트 더블이 무엇이며, 그 중에서 mock을 사용하는 방법에 대해서 알 수 있었다. 그리고 의존성 주입(Dependency Injection)에 대한 개념을 다시 한 번 확인하는 좋은 계기가 되었던 것 같다. 


## 참고 
[Test Double을 알아보자](https://woowacourse.github.io/tecoble/post/2020-09-19-what-is-test-double/) <br>
[Clint Jang Dependency Injection이란](https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f) <br>
[The complete guide to Netowkr Unit Testing in Swift](https://medium.com/flawless-app-stories/the-complete-guide-to-network-unit-testing-in-swift-db8b3ee2c327) <br>
[WWDC Testing Tips and Tricks 정리]({{site.url}}{{site.baseurl}}/wwdc/TestingTipsTricks/) <br>
[innie Networking Unit Test in Swift](https://innieminnie.github.io/xctest/tdd/testdouble/dependencyinjection/2021/07/13/NetworkingUnitTestinSwift.html) <br>
[Unit Testing URLSession using URLProtocol](https://medium.com/@dhawaldawar/how-to-mock-urlsession-using-urlprotocol-8b74f389a67a)