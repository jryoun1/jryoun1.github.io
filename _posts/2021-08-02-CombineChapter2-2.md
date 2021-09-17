---
title:  "[Combine] Chapter2 Publisher와 Subscriber - 2"
excerpt: "Combine에서 Publisher와 Subscriber에 대해서 자세히 알아보자"

categories:
  - Combine
tags:
  - [Custom Subscribers, Future, Subject, Demand]
last_modified_at: 2021.08.02
---

## 들어가며 
지난 [Chapter2]({{site.url}}{{site.baseurl}}/combine/CombineChapter2-1)에서는 Publisher, Subscriber와 Sink, Assign operator, Cacenllable에 대해서 알아보았다. 

이번 포스팅에서는 이제 Custom Subscriber의 작성과 Future, Subject, Demand의 변경 등에 대해서 알아볼 것이다. <br>

## 사용자 지정 subscriber 생성하기

```swift
example(of: "Custom Subscriber") {
    // 1️⃣
    let publisher = (1...6).publisher
    // 2️⃣
    final class IntSubscriber: Subscriber {
        // 3️⃣
        typealias Input = Int
        typealias Failure = Never
        // 4️⃣
        func receive(subscription: Subscription) {
            subscription.request(.max(3))
        }
        // 5️⃣
        func receive(_ input: Int) -> Subscribers.Demand {
            print("Received value", input)
            return .none
        }
        // 6️⃣
        func receive(completion: Subscribers.Completion<Never>) {
            print("Received completion", completion)
        }
    }
}
```

1. 범위 내의 정수 값을 가지는 pulisher를 생성
2. Custom subscriber 클래스 생성
3. Int 타입을 수신할 것이며, Error는 수신하지 않는다고 설정
4. publisher에 의해서 호출되며, subscriber가 최대 3개까지 값을 수신하겠다는 것을 의미하도록 subscription을 설정
5. 수신받은 값을 print하고, `return .none` 을 하여 subscriber가 demand를 조정하지 않는다는 것을 나타낸다. 그리고 `.none = .max(0)` 과 동일하다
6. completion event를 print

이제 publisher가 내보내기 위해서는 subscriber가 필요하다. <br>따라서 subscriber를 추가해준다. 

```swift
let subscriber = IntSubscriber()
publisher.subscribe(subscriber)
```

이때 publisher와 Output, Failure 타입이 일치하는 subscriber 생성하고, publisher에 subscriber를 붙여준다. 

![7](/assets/images/Combine/Chapter2/7.png)

다음과 같은 결과가 나오는데, 이때 completion은 수신하지 못했는데, 왜냐하면 max를 3으로 제한했기 때문이다. 

따라서 `receive(_:)` 에 반환 값을 `.none` 에서 `.unlimited` 로 수정하면 completion까지 제대로 출력된다. 

```swift
func receive(_ input: Int) -> Subscribers.Demand {
    print("Received value", input)
    return .unlimited
}
```

![8](/assets/images/Combine/Chapter2/8.png)

그리고 이때 `.unlimited` 를 `.max(1)` 로 수정해도 위와 같은 결과가 나오는데, 이는 매번 이벤트를 수신할 때, max를 1씩 증가시킨다는 뜻이기 때문이다. 

```swift
func receive(_ input: Int) -> Subscribers.Demand {
    print("Received value", input)
    return .max(1)
}
```
그리고 만약 처음에 선언한 publisher를 Int 타입에서 String 배열로 바꾸고 실행해보면 에러가 발생하게 된다. 

```swift
let publisher = ["A", "B", "C", "D", "E", "F"].publisher
```

이는 `subscribe()` 메서드는 String 타입을 필요하는 반면에, IntSubscriber.Intput은 Int 타입이기 때문이다. 즉, Publisher의 **Output, Failure 타입**은 subscription을 위해서는 항상 Subscriber의 **Input, Failure 타입과 일치**시켜야만 한다. 

## Future

`Just` 는 **하나의 값을 subscriber에 내보내고, complete** 하는 publisher를 생성한다. <br>
`Future` 은 **비동기적으로 하나의 결과를 생성하고 complete** 하는 publisher를 생성한다.

```swift
var subscriptions = Set<AnyCancellable>()
example(of: "Future") {
    func futureIncrement(
        integer: Int,
        afterDelay delay: TimeInterval) ->  Future<Int, Never> { 
            Future<Int, Never> { promise in
            DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
              promise(.success(integer + 1))
            }
          }
    }
}
```

Int, Never 타입의 future를 반환하는 factory function인 `futureIncrement()` 를 생성한다. 그리고 subscriptions 변수에 future에 subscription을 저장하게 된다. 길게 동작하는 비동기 작업의 경우에는 subscription을 저장하지 않으면, 코드의 범위가 끝날 때 cancelation이 발생하게 된다. (Playground에서는 바로 cancelation 발생)

따라서 함수의 body를 위와 같이 채워준다. <br>
future를 정의해서 caller에 의해 명시된 값을 사용해 integer를 delay이후에 증가시킬 것이라는 promise를 생성한다. 

즉, `Future` 는 결국 하나의 value를 만들고 끝나는 publisher이며, 값이나 에러가 발생할 때 promise라고도 불리는 클로저를 호출한다. 

```swift
final public class Future<Output, Failure> : Publisher where Failure : Error {
    public typealias Promise = (Result<Output, Failure>) -> Void
	...
}
```

`Future` 의 정의에서 `Promise` 는 Future에 의해서 내보내진 값이나 에러를 가지는 `Result<Output, Failure>` 를 수신하는 클로저의 별칭이다. 

```swift
// 1️⃣
let future = futureIncrement(integer: 1, afterDelay: 3)
// 2️⃣    
future
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
```

1. 3초 뒤에 전달한 정수 값을 증가시키는 future를 factory function을 통해서 생성
2. subscribe 하고 값과 completion event를 수신하고 결과를 subscriptions set에 저장한다

실행 결과 3초 뒤에 결과가 나오게 된다. 

![9](/assets/images/Combine/Chapter2/9.png)

```swift
future
    .sink(receiveCompletion: { print("Second", $0) },
          receiveValue: { print("Second", $0) })
    .store(in: &subscriptions)

// futureIncrement 함수의 DispatchQueue 블록 직전에 아래 print 추가
print("Original")
```

실행하면, 특정 delay이후에 **두 번째 subscritpion이 같은 값을 수신**하는 걸 확인할 수 있다. future은 **promise를 다시 실행시킨것이 아니라 결과 값을 share하거나 replay한 것**이다. 

![10](/assets/images/Combine/Chapter2/10.png)

그리고 Original은 subscription이 시작되자마자 바로 출력되는데 왜냐하면 **future는 다른 publisher들과는 다르게 subscriber를 필요로 하지 않기 때문**이다.


위의 예제들로 제한된 갯수만큼 내보내는 publisher에 대해서 알아봤고, 내보내지는 값은 순차적이고 동기적으로 내보내졌다. 처음에 시작한 NotificatioCenter 예시는 비동기적이고 무제한으로 값을 내보낼 수 있는 publisher의 예시이다. 

- notification sender는 notification을 내보낸다
- 특정 notification에는 구체적인 subscriber가 있다

위와 같은 작업을 할 수 있는 subject에 대해서 알아보자.

## Subject

`Subject` 는 **Combine이 아닌 코드에서 combine subscriber에게 value를 전송할 수 있게** 해준다.

```swift
example(of: "PassthroughSubject") {
    // 1️⃣
    enum MyError: Error {
        case test
    }
    // 2️⃣
    final class StringSubscriber: Subscriber {
        typealias Input = String
        typealias Failure = MyError
        
      	func receive(subscription: Subscription) {
      		subscription.request(.max(2))
    		}
 
        func receive(_ input: String) -> Subscribers.Demand {
            print("Received value", input)
            // 3️⃣
            return input == "World" ? .max(1) : .none
        }
        
        func receive(completion: Subscribers.Completion<MyError>) {
            print("Received completion", completion)
        }
    }
    
    // 4️⃣
    let subscriber = StringSubscriber()
}
```

1. 사용자 지정 에러 타입 정의
2. MyError와 String을 수신하는 사용자 지정 subscriber 클래스 정의
3. 수신된 값에 대한 요구사항을 반영
4. 사용자 지정 subscriber 객체 생성

입력 값이 "World"일 때 `.max(1)` 을 `receive(_:)` 에 반환하면 새로운 max 는 3이 된다. (원래 max 값 + 1) 

```swift
// 5️⃣
let subject = PassthroughSubject<String, MyError>()
// 6️⃣
subject.subscribe(subscriber)
// 7️⃣
let subscription = subject
    .sink(
    receiveCompletion: { completion in
        print("Received completion (sink)", completion)
    },
    receiveValue: { value in
        print("Received value (sink)", value)
    })
```

5. String, MyError 타입을 가지는`PassthroughSubject`  객체 생성
6. subscriber를 subject에 subscribe한다
7. sink를 사용해 다른 subscription을 생성

`Passthrough` subject는 **새로운 값을 내보낼 수 있게 해준다.** 다른 publisher와 마찬가지로 **publisher가 내보낼 수 있는 값과 오류 타입을 미리 선언**해야하고 subscriber는 해당 `Passthrough` **subject를 subscribe하려면 입력과 실패 타입을 일치**시켜야한다. 

```swift
subject.send("Hello")
subject.send("World")
```

값을 보내보면 각각의 subscriber는 내보내지는 값을 수신한다. 

![11](/assets/images/Combine/Chapter2/11.png)

```swift
subscription.cancel()
subject.send("Still there?")
```

이후 subscription을 취소하고 subject에 또 보내면 첫 번째 subscriber만 값을 수신하고, 두 번째 subscriber는 수신하지 못한다. 

![12](/assets/images/Combine/Chapter2/12.png)

```swift
subject.send(completion: .finished)
subject.send("How about another one?")
```

위의 코드를 추가하고 실행하면 해당 값은 수신되지 않는데, 첫 번째 subscriber는 보내기 전에 completion 이벤트를 수신하기 때문이고 두 번째 subscriber는 이미 전에 cancel되었기 때문에 다음과 같은 결과가 나온다. 

![13](/assets/images/Combine/Chapter2/13.png)

```swift
subject.send(completion: .failure(.test))
```

그리고 만약 completion을 보내기 전에 에러를 먼저 보낸다면 첫 번째 subscriber는 에러를 수신하고 이후의 값은 수신되지 않는것을 확인할 수 있다. 이를 통해 **publisher가 하나의 completion event(정상적인 completion, Error)를 보내면 끝난다는 것을 확인**할 수 있다. 

`PassthroughSubject` 로 값을 전달하는 방법은 명령헝 코드와 선언형 세계인 Combine을 이어주는 다리의 역할이다. 그러나 가끔 명령형 코드에서 publisher의 현재 값을 보고 싶은 경우가 있는데, 그럴 때는 `CurrentValueSubject` 를 사용하면 된다. 

하나의 subscription을 각각 저장하는 대신 `AnyCancellable` 콜렉션에 subscription을 여러 개를 저장할 수 있다. 이 콜렉션은 더해진 subscription들을 자기 자신이 deinit 될 때 함께 cancel한다. 

```swift
example(of: "CurrentValueSubject") {
  // 1️⃣
  var subscriptions = Set<AnyCancellable>()
  // 2️⃣
  let subject = CurrentValueSubject<Int, Never>(0)
  // 3️⃣
  subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions) // 4️⃣
}
```

1. subscription set 생성
2. Int, Never 타입을 가지는`CurrentValueSubject` 생성 (초기값은 0)
3. subject를 sink를 통해 subscribe하고 값을 출력
4. subscriptions set에 subscription을 저장하고, inout 파라미터`&`를 통해 copy 대신 업데이트 되도록 한다

실행하면 초기값 0을 출력한다. 

```swift
subject.send(1)
subject.send(2)

print(subject.value)
```

값을 두 개 더 보내면 1,2가 출력된다. 그러나 PassthroughSubject와는 다르게 current value를 `.value` 프로퍼티를 통해서 확인할 수 있다. 

```swift
subject.value = 3
print(subject.value)
```

`send()` 를 통해서 새로운 값을 보낼 수도 있지만, **다른 방법으로 value 프로퍼티에 값을 할당**하는 것도 가능하다. 바로 **직접 value에 할당**하는 방법이다. 그러나 이때 프로퍼티에 **`.finished` 와 같은 event는 할당할 수 없으므로 `send()` 를 통해서만 보내야한다.** 

```swift
subject
    .sink(receiveValue: { print("Second subscription:", $0) })
    .store(in: &subscriptions)
```

현재 subject 값으로 새로운 subscription을 생성하고 수신된 값을 print하고 subscriptions set에 subscription을 저장한다. 그러나 위에서 subscriptions set이 자동적으로 추가된 subscription을 cancel 한다고 했는데 이를 어떻게 확인할 수 있을까?

바로 print()를 subject와 sink 사이에 넣어주는 것이다. 

```swift
subject
    .print()
    .sink(receiveValue: { print("Second subscription:", $0) })
    .store(in: &subscriptions)
```

그럼 다음과 같은 결과가 나오게 된다. 

![14](/assets/images/Combine/Chapter2/14.png)

모든 event는 subscription handler에 있는 값과 함께 subject의 value를 print할 때 마다 print된다. 마지막에  receive cancel 이벤트가 출력되는데, 이는 **범위 내에서 subscriptions set이 정의되었고 범위가 끝났기에 deinit되면서 포함되어있는 subscription을 cancel**하게 되는 것이다. 

```swift
subject.send(completion: .finished)
```

이때 completion 이벤트를 보내주면 cancel event가 아닌 completion event를 수신하고, 그럼 끝났으므로 더 이상  cancel될 필요가 없다. 

## Dyanamically adjusting demand

`Subscriber.receive(_:)` 에서 demand를 동적으로 바꾸는 방법에 대해서 알아보자.

```swift
example(of: "Dynamically adjusting Demand") {
    final class IntSubscriber: Subscriber {
        typealias Input = Int
        typealias Failure = Never
        
        func receive(subscription: Subscription) {
            subscription.request(.max(2))
        }
        
        func receive(_ input: Int) -> Subscribers.Demand {
            print("Received value", input)
            
            switch input {
            case 1:
                return .max(2) // 1️⃣
            case 3:
                return .max(1) // 2️⃣
            default:
                return .none // 3️⃣
            }
        }
        
        func receive(completion: Subscribers.Completion<Never>) {
            print("Received completion", completion)
        }
    }
    
    let subscriber = IntSubscriber()
    
    let subject = PassthroughSubject<Int, Never>()
    
    subject.subscribe(subscriber)
    
    subject.send(1)
    subject.send(2)
    subject.send(3)
    subject.send(4)
    subject.send(5)
    subject.send(6)
}
```

1. 새로운 max 값은 4 (기존의 max 2 + 새로운 max 2)
2. 새로운 max 값은 5 (기존의 max 5 + 새로운 max 1)
3. max는 5로 남아있음 (기존의 max 5 + 새로운 max 0)

따라서 실행결과는 다음과 같다. 

![15](/assets/images/Combine/Chapter2/15.png)

예상한대로 max 의 값이 5로 되어있기 때문에 6번째 값은 수신하지 못한 것을 확인할 수 있다. 

## Type erasure

**subscriber로부터 publisher의 상세 정보를 숨기는 방법**이 필요한 경우가 있다. 즉, publisher의 추가 정보에 접근하지 않고 subscriber가 publisher를 subscribe 해서 이벤트를 수신하는 경우다. 

```swift
example(of: "Type erasure") {
    // 1️⃣
    let subject = PassthroughSubject<Int, Never>()
    // 2️⃣
    let publisher = subject.eraseToAnyPublisher()
    // 3️⃣
    publisher
        .sink(receiveValue: { print($0) })
        .store(in: &subscriptions)
    // 4️⃣
    subject.send(0)
}
```

1. `PassthroughSubject` 생성
2. subject로부터 type-erased publisher 생성
3. sink를 사용해서 type-erased publisher를 subscribe
4. Passthrough subject를 통해서 새로운 값 보내기

`Publisher` 살펴보면 `AnyPublisher<Int, Never>` 라는 타입이 있다.

```swift
let publisher: AnyPublisher<Int, Never>
```

`AnyPublisher` 는 `Publisher` 프로토콜을 따르는 type-erased 구조체 타입이다. Type erasure은 **subscriber에게 노출하고 싶지 않은 publisher에 대한 상세 정보들을 숨길 수 있게 해준다.**

이는 이전에 `Cancellable` 에서 `AnyCancellable` 과 비슷하다. 실제로 `AnyCancellabe` 은 `Cancellable` 을 따르는 type-erased 클래스이며, caller가 더 많은 요청을 하기 위해서 subscription에 접근할 수 없이 subscription을 cancel하는 것이다. 

publisher에서 type-erased 타입을 사용하는 경우는 접근제한자로 public, private 쌍을 프로퍼티로 사용하는 경우이다. 이러한 속성의 소유자가 private publisher에 값을 보내기위해서 사용하고, caller는 public publisher만 subscribe하고 이때 값은 보낼 수 없는 형태일때이다. 

`AnyPublisher` 는 `(send_:)` operator가 없으므로, 새로운 값을 publisher에 추가할 수 없다.

`eraseToAnyPublisher()` operator는 **제공된 publisher를 `AnyPublisher` 객체로 래핑하여 publisher가 실제로 `PassthroughSubject` 타입인 것을 숨긴다**. 게다가 Publisher **protocol을 지정할 수 없다**. 즉, Publisher<UIImage, Never> 라고 정의할 수 없다는 것이다. 

이에 위의 코드에서 publisher가 type-erased 타입임을 확인하려면 아래의 코드를 추가하고 실행하면 `Value of type 'AnyPublisher<Int, Never>' has no member 'send'` 오류가 발생하는 것을 확인할 수 있다. 

```swift
publisher.send(1)
```

## 마치며
이번 포스팅에서는 Custom Subscriber, Future, Subject, Demand, Type erasure에 대해서 알아보았다. 이제 [Chapter3]({{site.url}}{{site.baseurl}}/combine/CombineChapter3/)에서는 본격적으로 Operator에 대해서 알아 볼 것이다. 


## 참고
[Raywenderlich Combine Chapter2: Publisher & Subscriber](https://www.raywenderlich.com/books/combine-asynchronous-programming-with-swift/v2.0/chapters/2-publishers-subscribers#toc-chapter-005-anchor-002)