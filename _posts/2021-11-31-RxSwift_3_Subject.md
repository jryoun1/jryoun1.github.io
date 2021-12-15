---
title:  "[RxSwift] Subject"
excerpt: "Subject에 대해서 알아보자"

categories:
  - RxSwift
tags:
  - []
last_modified_at: 2021.11.31
--- 

# Subject

실제로 프로젝트를 진행할 때는 실시간으로 Observable 에 값을 추가하고, Subscriber를 할 수 있는 것이 필요하다.

이때 `Observable` 이자 `Observer` 인 것이 바로 `Subject` 이다.<br>
`Observable + Observer = Subject라는 의미를 알아야함` <br>
이벤트를 외부에 전달해 줄 경우에 사용해, delegate 대신 사용하기도 함

Observable은 observer 하나만을 subscribe할 수 있고 (unicast)
Subject는 여러 개의 observer를 subscribe할 수 있다 (multicast)

<img src="https://user-images.githubusercontent.com/45090197/146187498-5ef894f1-d148-467b-b5d1-d401899695d1.png">

## Unicast

```swift
// --- Observable ---
    let randomNumGenerator1 = Observable<Int>.create{ observer in
        observer.onNext(Int.random(in: 0 ..< 100))
        return Disposables.create()
    }
    
    randomNumGenerator1.subscribe(onNext: { (element) in
        print("observer 1 : \(element)")
    })

    randomNumGenerator1.subscribe(onNext: { (element) in
        print("observer 2 : \(element)")
    })
    
//--------------------print------------------  
//observer 1 : 54
//observer 2 : 69
```

다른 숫자가 출력된다. observer가 해당 observable에 대해 독자적인 실행을 가지기 때문에 비록 동일한 observable 구독을 통해서 생성된 두 개의 observer라고 해도 각각 실행되면서 observer에게 서로 다른 값이 전달되는 것이다. 

## Multicast

```swift
// ------ BehaviorSubject/ Subject
    let randomNumGenerator2 = BehaviorSubject(value: 0)
    randomNumGenerator2.onNext(Int.random(in: 0..<100))
    
    randomNumGenerator2.subscribe(onNext: { (element) in
        print("observer subject 1 : \(element)")
    })

    randomNumGenerator2.subscribe(onNext: { (element) in
        print("observer subject 2 : \(element)")
    })

//--------------------print------------------
//observer subject 1 : 92
//observer subject 2 : 92
```

하나의 observable 실행이 여러 subscriber에게 공유된다.<br>
따라서 구독해서 생성되는 observer에게 observable의 동일한 실행이 전달된다. 

⇒ 즉 Observable에서 subscribe를 하면 이벤트로 전달되는 것은 항상 새롭게 실행된 결과 <br>
⇒ 반면 Subject에서 subscribe하면 이벤트로 전달되는 것은 subscribe전에 수행된 onNext 등의 하나의 공통된 것

<img src="https://user-images.githubusercontent.com/45090197/146188245-89687dba-5701-4749-9e15-d3ad1293d79f.png">

# Observer

음.. Observable은 알겠는데 갑자기 Observer는 뭐지..? 

## 정의

```markdown
# reactiveX 홈페이지에서 Observable 정의
In ReactiveX an observer subscribes to an Observable. 
Then that observer reacts to whatever item or sequence of items the Observable emits.

ReactiveX에서 observer는 observable을 구독한다
observer는 observable이 방출하는 모든 아이템들에 대해 반응한다
```

정의는 이렇다고 한다.<br>
이번 개념에서 나올 키워드는 `observable` , `subscribe` , `observer` 이다.

```swift
observable.subscribe(onNext: { element in 
			print(element)
	})
```

RxSwift에서 자주 사용되는 코드이다. <br>
`observable`, `subscribe`는 있는데 `observer`는 뭐지..?

## subscribe

`subscribe`의 정의는 다음과 같다.

```swift
public func subscribe(onNext: ((E) -> Void)? = nil, 
                    onError: ((Swift.Error) -> Void)? = nil, 
                    onCompleted: (() -> Void)? = nil,
                    onDisposed: (() -> Void)? = nil) -> Disposable
```

구현부는 아래와 같다.

```swift
public func subscribe(onNext: ((E) -> Void)? = nil, 
                    onError: ((Swift.Error) -> Void)? = nil, 
                    onCompleted: (() -> Void)? = nil,
                    onDisposed: (() -> Void)? = nil) -> Disposable {
            let disposable: Disposable            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? 
														Hooks.customCaptureSubscriptionCallstack() : []
            
            //1) 안에서 자체적으로 observer 생성
            let observer = AnonymousObserver<E> { event in
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
                // 2) 'observer'에 대한 subscription. 
                // 이렇게 해서 해당 옵저버블에 대해 옵저버가 관찰을 할 수 있다.
                self.asObservable().subscribe(observer),
                disposable
            )
    }
```

1. 내부에서 자체적으로 observer 생성 <br>
    ⇒ observable을 `subscribe` 하면 그 구현부에서 observer 가 생성되고 있음
2. Observer에 대한 subscription

먼저 asObservable은 다음과 같다.

```swift
extension ObservableType {
    /// Default implementation of converting `ObservableType` to `Observable`.
    public func asObservable() -> Observable<E> {
        // temporary workaround
        //return Observable.create(subscribe: self.subscribe)
        return Observable.create { o in
            return self.subscribe(o)
        }
    }
}
```

즉, ObservableType을 Observable로 변환해주는 것!! <br>
그리고 뒤에 붙는 `.subscribe(observer)` 는 Observable이 채택한 ObservableType protocol을 보자

```swift
public protocol ObservableType : ObservableConvertibleType {
   func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E
}
```

내부에 `subscribe` 함수는 다음과 같다고 한다.

```swift
이 시퀀스에서의 이벤트를 받기 위해 observer를 구독한다.

[문법]
시퀀스는 0개 이상의 요소들을 생산할 수 있고, `.next` 를 통해서 이벤트들이 observer로 전달된다.

`error`, `completed` 이벤트가 발생했을 때, 해당 시퀀스는 종료되고 더 이상 이벤트 방출 ❌

이벤트들이 각기 다른 스레드에서 전달될 수는 있지만, 
동시에 두개의 이벤트가 observer로 전달되는 것은 불가능하다

[리스소 관리]
시퀀스가 `completed`, `error` 이벤트를 보내면 
해당 시퀀스의 요소를 다뤘던 모든 내부적 자원은 해제된다

시퀀스의 요소 방출을 막고, 자원 즉각 해제하기 위해서 반환되는 subscription에서 dispose 호출가능

[반환값]
시퀀스가 요소 생산을 취소하고, 자원을 해제하는데 쓰일 수 있는 `observer`에 대한 subscription

```

즉, observer를 받아서 그에 대한 subscription을 반환한다. <br>
그리고 이는 자원을 해제할 수 있는 형태(Disposable)이라고 나와 있다

⭐️ **결국 observable.subscribe 할 때마다, 각각 만들어지는 observer들에 대한 Disposable 타입의 subscription이라는 것이다.** 

## 결론

observer가 observable을 관찰할 수 있다는 것 <br>
⇒ observable을 subscribe할 때, subscribe 구현부에서 각각의 observer를 생성하고 그 observer에 대한 subscription을 만들기 때문에 가능하다

# Subject 종류

## PublishSubject

`.completed` , `.error` 가 발생하기 전까지, <br>
즉 종료될 때까지 **subscribe 한 이후부터 이벤트를 방출**

⇒ 아무것도 없는 빈 상태로 subscribe를 시작하고, 오직 새로운 elements만 subscriber에게 방출

<img src="https://user-images.githubusercontent.com/45090197/146189136-66e75137-fe87-4c88-8ef4-2173c96f7bca.png">


```swift
let subject = PublishSubject<Int>()

let subjectOne = subject
        .subscribe(onNext: { (num) in 
            print("subjectOne:" , num)
		})

subject.onNext(1)
subject.onNext(2)

let subjectTwo = subject
        .subscribe(onNext: { (num) in 
            print("subjectTwo:" , num)
		})

subject.onNext(3)
subject.onNext(4)

/* 
subjectOne: 1
subjectOne: 2
subjectOne: 3
subjectTwo: 3
subjectOne: 4
subjectTwo: 4
*/
```

⚠️ subject가 종료된 이후에, **새로운 subscriber가 생긴다고 다시 subject가 작동하지 않는다** <br>
subject 종료될 때, subscriber에게 종료 이벤트를 줄 뿐만 아니라 이후 구독한 subscriber에게도 종료이벤트를 알려준다

```swift
let subject = PublishSubject<Int>()

let subjectOne = subject
        .subscribe(onNext: { (num) in 
            print("subjectOne:" , num)
		})
		

subject.onNext(1)
subject.onNext(2)
subject.onCompleted()

let subjectTwo = subject
        .subscribe(onNext: { (num) in 
            print("subjectTwo:" , num)
		})

subject.onNext(3)
subject.onNext(4)

/* 
subjectOne: 1
subjectOne: 2
subjectOne: onCompleted
subjectTwo: onCompleted
*/
```

사용하는 곳
- 시간에 민감한 데이터를 모델링할 때, (실시간 경매 애플리케이션)
- 만약 9시에 장이 시작된다고 할 때, 9시01분에 들어온 유저에게 8시 59분에 기존에 날린 "서두르세요. 1분남았습니다"라는 이벤트를 방출하는 것은 무의미할 것이다.

### 의문점

Q. observable이 disposed 되는 시점?

`.completed` 나 `.error` 이벤트에 의해서 diposed가 되는 것인지 혹은 이런 이벤트를 받고나서도 `.dispose(by: bag)` 와 같은 체이닝이 있어서 사이클을 타야지 되는 것인지? 

**[결과]** <br>
결국 `.completed`, `.error` 이벤트를 받고 난 이후에는 자동으로 diposed 된다. (onDisposed 내부가 호출됨)

그렇다면 `.disposed(by: )` 를 체이닝하는 이유는 혹시라도 completed, error가 호출되지 않고 다른 뷰컨으로 넘어가거나 할 때, 메모리 릭이 발생할 수 있을 것 같아서 해주는것 같다!!


## BehaviorSubject

PublishSubject과 유사하지만 `초기값` 을 가진다.

또한 PublishSubject과는 달리 항상 `직전` 의 값부터 구독한다

즉 **마지막 `.next` 이벤트를 새로운 구독자에게 반복**한다는 점만 제외하면 PublishSubject와 유사함

⇒ 항상 최근의 원소를 방출하기 때문에 초기 값을 제공하지 않으면 생성할 수 없음 

⇒ 즉, 만약 초기 값이 없다면 PublishSubject를 사용해라 or Optional을 사용하는 방법도 있긴함

<img src="https://user-images.githubusercontent.com/45090197/146189166-5a99244f-3341-4980-bf5f-e6557722b14c.png">

그러나 만약 Observable이 error로 인해서 종료되는 경우에, BehaviorSubject는 이후에 subscribe하는 observer들에게 아무런 값도 방출하지 않는다. 하지만 error notification은 전달해준다.

<img src="https://user-images.githubusercontent.com/45090197/146189777-e7cf08ad-e350-4178-8e29-cb7d73713b0e.png">

```swift
enum MyError: Error {
     case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
     print(label, event.element ?? event.error ?? event)
}

example(of: "BehaviorSubject") {

     let subject = BehaviorSubject(value: "Initial value")
     let disposeBag = DisposeBag()

    // ⚠️ 나중에 추가한다고 가정
     subject.onNext("X")

     subject
         .subscribe{
             print(label: "1)", event: $0)
         }
         .disposed(by: disposeBag)

     subject.onError(MyError.anError)

     subject
         .subscribe {
             print(label: "2)", event: $0)
         }
         .disposed(by: disposeBag)
}
```

이때 처음에 출력 결과는 아래와 같다.

```swift
--- Example of: BehaviorSubject ---
1) Initial value 
```

이때 위의 주석 부분을 추가해준다고 하면, 결과는 바뀌게 됨

```swift
--- Example of: BehaviorSubject ---
1) X 
```

이후 에러를 발생시키고, 이후에 subject를 subscribe 하면 결과는 다음과 같이 발생하게 된다.

```swift
1) anError
2) anError
```

즉, 위에서 말했듯이 에러가 발생하고 나면 어떠한 값도 방출하지 않지만, 새로 추가된 subscriber에게도 에러가 발생한 사실은 알려준다. 

사용하는 곳 

- 뷰를 가장 최신의 데이터로 미리 채우기에 용이하다
- 예를 들어 UserProfile 화면의 컨트롤을 BehaviorSubject에 바인드 할 수 있다 <br>
이렇게 하면 앱이 새로운 데이터를 가져오는 동안 최신 값을 사용하여 화면을 미리 채울 수 있음

## ReplaySubjects

Observer가 subscribe하는 시점과 관계없이, Observables가 방출한 모든 값들을 observer에게 방출

<img src="https://user-images.githubusercontent.com/45090197/146189933-d4294a87-0c02-45b8-b3a6-098532db6882.png">

ReplaySubject 는 초기화 된 **buffer size로 시작** <br>
→ 그 사이즈까지 buffer의 원소들을 유지하며 새로운 subscriber들에게 값을 방출 <br>
즉, buffer size에 도달할 때까지 계속 방출해준다(replay)는 의미라고 이해할 수 있다.

위의 그림의 buffer size는 몇일까? `정답은 2` <br>
왜냐하면 replay 되는 원소들이 빨간색, 초록색 두 개 이므로 buffer size는 2가 된다. <br>
이때 첫 구독자는 subject와 함께 구독하므로 subject의 값을 그대로 받는다. (2번째 줄) <br>
반면에 두 번째 구독자는 subject가 두 개의 이벤트를 받은 이후 구독하지만 버퍼사이즈 2만큼의 값도 받을 수 있다 (3번째 줄)

⚠️ ReplaySubject를 사용할 때 주의해야할 점

**버퍼들은 메모리를 가지고 있기 때문에, 이미지나 array같이 메모리를 크게 차지하는 값들을 큰 사이즈의 버퍼로 가지는 것은 메모리에 엄청난 부하를 줄 수 있다.** 

또한 ReplaySubject를 observer로 사용하는 경우에 **여러 스레드에서 `onNext()` 를 호출하지 않도록 주의해야한다.** 왜냐하면 이는 Observable의 원칙 중에 하나인 한 번에 하나의 값을 받는 다는 것을 위반하고 어떠한 것이 먼저 replay 되야하는지 모호하게 만들 수 있기 때문에 주의해야한다. 

```swift
example(of: "ReplaySubject") {
  // 1
  let subject = ReplaySubject<String>.create(bufferSize: 2)
  let disposeBag = DisposeBag()

  // 2
  subject.onNext("1")
  subject.onNext("2")
  subject.onNext("3")
	
  // 3
  subject
    .subscribe {
      print(label: "1)", event: $0)
    }
    .disposed(by: disposeBag)
	subject
    .subscribe {
      print(label: "2)", event: $0)
    }
    .disposed(by: disposeBag)
}

// 4
subject.onNext("4")

// 5
// subject.onError(MyError.anError)
// 6
// subject.dispose()

subject
  .subscribe {
    print(label: "3)", event: $0)
  }
  .disposed(by: disposeBag)

```

1. 버퍼 사이즈를 2로 가지는 String 타입을 방출하는 replaySubject를 생성
2. 이후 1,2,3 세 개의 요소들을 subject에 추가한다.
3. 그리고 2개의 구독자를 생성해본다. 

결과는 다음과 같다.

```swift
--- Example of: ReplaySubject ---
1) 2
1) 3
2) 2
2) 3
```

가장 최근의 요소 2개만 방출되고, 버퍼 사이즈인 2개를 넘는 1은 방출되지 않는다.

이때 4번 줄을 추가해보면 결과는 다음과 같다.

```swift
1) 4
2) 4
3) 3
3) 4
```

즉, 기존 구독자인 1,2 는 새롭게 subject에 추가된 값인 4를 받게 되고, 새 구독자인 3은 버퍼사이즈 2개 만큼의 최근 값을 받게 된다. 

이제 5번 주석을 추가하면 되면 다음과 같은 결과가 나타난다.

```swift
1) 4
2) 4
1) anError
2) anError
3) 3
3) 4
3) anError
```

비록 error로 인해서 **subject가 종료되었음에도 불구하고 새로운 구독자인 3번에게 버퍼에 있는 값을 보내준다**. 

⇒ subject가 종료되었어도 버퍼는 여전히 돌아다니기 때문에 이러한 결과가 발생하게 된다. 따라서 error 추가한 다음에 반드시 dispose하여 이벤트의 방출을 막아주어야한다. 

6번 주석을 추가하면 다음과 같이 종료된다. 

```swift
/* Prints:
3) Object `RxSwift.(ReplayMany in _33052C2CE59F358A8740AFDD4371DD39)<Swift.String>` was already disposed.
*/
```

- 이렇게 하면 새로운 구독자는 에러 이벤트만 받을 것이다. 왜냐하면 subject 자체가 구독 전에 이미 dispose 되었기 때문이다
- 다만, `ReplaySubject`에 명시적으로 `dispose()`를 호출하는 것은 적절하지 않다. 왜냐하면 만약 subjuect의 구독을 disposeBag에 넣고, 이 subject의 소유자(보통은 ViewController나 ViewModel)가 할당 해재되면 모든 것들이 dispose 될 것이기 때문이다.
- 참고로 상기 에러메시지에 표시된 `ReplayMany`는 `ReplaySubject`를 생성할 때 사용되는 내부 유형이다.

사용하는 곳

- BehaviorSubject처럼 최근의 값 외에도 더 많은 것을 보여주고 싶은 경우
- 예를 들어 검색창에서 최근 5개의 검색어를 보여주고 싶은 경우에 사용 가능

## Relays

`relay`가 replay behavior을 유지하면서 subject를 wrapping하는 것을 다뤘다. <br>
subjects, observables 와는 다르게 `accept(_:)` 를 통해서 값을 추가한다. <br>
즉, `onNext(_:)` 를 사용하지 않는다. <br>
왜냐하면 relay는 값을 받을 수만 있고, **error나 completed event를 보낼 수 없다.** <br>
아래의 publishRelay, subjectRelay는 둘 다 subject를 wrap 한 것이며, <br>
이때 **절대 종료되지 않는다는 것을 보장**한다. 

### PublishRelay

```swift
import RxSwift

/// PublishRelay is a wrapper for `PublishSubject`.
///
/// Unlike `PublishSubject` it can't terminate with error or completed.
public final class PublishRelay<Element>: ObservableType {
    private let subject: PublishSubject<Element>
    // Accepts `event` and emits it to subscribers
    public func accept(_ event: Element) {
        self.subject.onNext(event)
    }
    /// Initializes with internal empty subject.
    public init() {
        self.subject = PublishSubject()
    }
    /// Subscribes observer
    public func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        self.subject.subscribe(observer)
    }
    /// - returns: Canonical interface for push style sequence
    public func asObservable() -> Observable<Element> {
        self.subject.asObservable()
    }
}
```
위의 코드를 보다시피 PublishRelay는 PublishSubject를 래핑해서 가지고 있다.

그리고 `accept()` 를 보면 내부에서 `onNext()` 를 호출하고 있는 것을 확인할 수 있다.

```swift
example(of: "PublishRelay") {
  let relay = PublishRelay<String>()
  let disposeBag = DisposeBag()

  relay.accept("Knock knock, anyone home?")

  relay
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)

  relay.accept("1")
}
 
// --- Example of: PublishRelay ---
// 1
```

위의 예시를 보면 아직 subscriber가 없기 때문에 첫 accept인 knock은 출력되지 않는다. 
이후에 subscriber를 subscribe하고 나서는 accept 한 값이 제대로 나오는 것을 확인할 수 있다.

이 예시의 출력을 보면 relay가 아니라 마치 subject 만든 것과 같이 동일하게 작동한다. <br>
왜냐하면 위에서 봤듯이 PublishRelay는 결국 PublishSubject를 wrap 한 것이고, `accept()` 함수를 사용한다는 것과 error, completed 이벤트를 보낼 수 없다는 점을 제외하고는 동일하게 동작하기 때문이다.

### BehaviorRelay

BehaviorRelay도 마찬가지로 BehaviorSubject를 wrapping 한 것이다. <br>
따라서 completed, error event를 보내서 종료할 수 없다는 점은 동일하다. <br> 
그러나 BehaviorRelay는 초기 값과 함께 생성되고, `가장 최신의 값이나 초기 값을 새로운 subscriber들에게 replay`하게 된다. 
또한 behavior relay는 `언제든지 현재 값을 요청`할 수 있다. 

```swift
/// BehaviorRelay is a wrapper for `BehaviorSubject`.
///
/// Unlike `BehaviorSubject` it can't terminate with error or completed.
public final class BehaviorRelay<Element>: ObservableType {
    private let subject: BehaviorSubject<Element>

    /// Accepts `event` and emits it to subscribers
    public func accept(_ event: Element) {
        self.subject.onNext(event)
    }

    /// Current value of behavior subject
    public var value: Element {
        // this try! is ok because subject can't error out or be disposed
        return try! self.subject.value()
    }

    /// Initializes behavior relay with initial value.
    public init(value: Element) {
        self.subject = BehaviorSubject(value: value)
    }

    /// Subscribes observer
    public func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        self.subject.subscribe(observer)
    }

    /// - returns: Canonical interface for push style sequence
    public func asObservable() -> Observable<Element> {
        self.subject.asObservable()
    }
}
```

BehaviorRelay는 BehaviorSubject를 같은 방식으로 가지고 있다.

마찬가지로 `accept()` 내부에서는 `onNext()` 를 호출하고 있다.

```swift
example(of: "BehaviorRelay") {
  // 1
  let relay = BehaviorRelay(value: "Initial value")
  let disposeBag = DisposeBag()

  // 2
  relay.accept("New initial value")

  // 3
  relay
    .subscribe {
      print(label: "1)", event: $0)
    }
    .disposed(by: disposeBag)

  // 1
  relay.accept("1")

  // 2
  relay
    .subscribe {
      print(label: "2)", event: $0)
    }
    .disposed(by: disposeBag)

  // 3
  relay.accept("2")
}
```
초기 값과 함께 BehaviorRelay를 생성 (이때 타입을 명시할 수도 있음 BehaviorRelay<String>(value:))

그리고 accept 이후에 subscribe 하게 되면 결과는 

```swift
1) New initial value 
```

가 된다. 

이후에 다시 1을 accept 하고, 또 다른 subscriber를 붙이고 이후 accept을 하면 결과는 다음과 같다.

```swift
1) 1  // 기존의 relay가 1을 받음
2) 1  // 새로운 subscriber는 가장 최신값인 1을 받음
1) 2  // 기존의 relay는 2를 받음
2) 2  // 새로운 subscriber도 2를 받음
```

마지막으로 가장 최신 값을 확인할 수 있다.

```swift
print(relay.value)

2 // 가장 최근 값인 2가 출력된다.
```

BehaviorRelay의 현재 값을 확인하는 방법은 명령형과 반응형을 연결할 때 도움이 된다. 

또한 BehaviorRelay는 다용도이다.

- 다른 subject들처럼 **새로운 event가 방출될 때마다 반응**하도록 subscribe를 할 수 있고
- 업데이트를 받기 위해서가 아니라 **현재 값 만을 확인하기 위한 일회성**

으로도 사용할 수 있다. 

### Relays와 Variable의 공통점

- ObservableType을 채택함
- error, completed event를 통해서 종료될 수 없다는 점
- `onNext()` 를 사용하지 않는다는 점
- relay를 observable이나 subject처럼 구독하고 싶을 때 사용할 수 있는 `asObservable` 메서드가 있다

### 두 relay의 차이점

- `BehaviorRelay`는 `PublishRelay`에는 없는 `value`라는 프로퍼티가 있다.

### Variable (Deprecated)

RxSwift 4.0 이전에는 Variable이 있었다. 그러나 이는 deprecated 되고 현재는 BehaviorRelay가 이를 대체한다고 보면 된다. 

```swift
public final class Variable<Element> { 
		public typealias E = Element 
		private let _subject: BehaviorSubject<Element> 
		private var _lock = SpinLock() 
			
		// state 
		private var _value: E 
		#if DEBUG 
		fileprivate let _synchronizationTracker = SynchronizationTracker() 
		#endif 

	/// Gets or sets current value of variable. 
	/// Whenever a new value is set, all the observers are notified of the change. 
	/// Even if the newly set value is same as the old value, observers are still notified for change. 

	public var value: E { 
		get { 
			self._lock.lock(); defer { self._lock.unlock() } 
			return self._value 
		} 
		set(newValue) { 
			#if DEBUG 
					self._synchronizationTracker.register(synchronizationErrorMessage: .variable) 
					defer { self._synchronizationTracker.unregister() } 
			#endif self._lock.lock() 
			self._value = newValue 
			self._lock.unlock() 
			self._subject.on(.next(newValue)) 
	} 
} 

	/// Initializes variable with initial value. 
	/// - parameter value: Initial variable value. 
	public init(_ value: Element) { 
			#if DEBUG 
				DeprecationWarner.warnIfNeeded(.variable) 
			#endif 
			self._value = value 
			self._subject = BehaviorSubject(value: value) 
	} 

	/// - returns: Canonical interface for push style sequence 
	public func asObservable() -> Observable<E> { 
			return self._subject 
	} 

	deinit { 
		self._subject.on(.completed) 
	} 
}
```
Variable은 BehaviorSubject를 한 번 wrapping 한 것이다. 

이때 value는 public property로 되어있고, setter에서 _value에 대입을 하고 `subject.on(.next(newValue))` 로 이벤트를 생성하고 있기 때문에 아래코드와 같이 사용할 수 있다.

```swift
let disposeBag = DisposeBag() 

let intPropertyVariable = Variable(1) 
func changeProperty() { 
		// subscribe로 property를 구독한다 
		intPropertyVariable.asObservable() 
				.subscribe(onNext: { newValue in 
							print("newValue - \(newValue) ") 
				}).disposed(by: disposeBag) 
		
		// value 2로 변경 
		intPropertyVariable.value = 2 
}

// 결과
// newValue - 1
// newValue - 2
```

# 참고
1. [Subject naljin's 블로그](https://sujinnaljin.medium.com/rxswift-subject-99b401e5d2e5)
2. [raywenderlich chapter 3](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/3-subjects)