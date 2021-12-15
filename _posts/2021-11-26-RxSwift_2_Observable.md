---
title:  "[RxSwift] Observable"
excerpt: "Observable에 대해서 알아보자"

categories:
  - RxSwift
tags:
  - []
last_modified_at: 2021.11.26
--- 

# Observable이란

- `observable` = `observable sequence` = `sequence` = 다 같은 의미
- 비동기적임
- 일정 기간 동안 계속해서 **event를 방출함 (emit)**
- 각 event는 `숫자`나 `커스텀한 인스턴스` 등과 같은 값을 가질 수 있고, 탭과 같은 `제스처`를 인식할 수도 있다

## 생명주기

<img src="https://user-images.githubusercontent.com/45090197/146013029-4756774f-22f7-4abc-b6e4-9e170901fef6.png"/>

세 개의 구성 요소가 있음

Observable은 `next` 이벤트를 통해 각각의 요소들을 방출함

<img src="https://user-images.githubusercontent.com/45090197/146013258-f94bd887-0618-40a6-bc57-e96d04f20a8b.png"/>

이 observable은 세 개의 tap 이벤트를 방출 한 뒤 종료됨. 이는 `completed` 이벤트라고 함

<img src="https://user-images.githubusercontent.com/45090197/146013378-6e80136e-19f1-47b3-8a66-11034db003cd.png"/>

이 observable은 에러가 발생하였다.

Observable이 종료되었다는 점에서는 일치하지만, `error` 이벤트를 방출하고 이에 의해 종료된 것

```swift
Observable은
- 어떤 구성 요소 가지는 `next` 이벤트를 계속 방출 가능
- `error` 이벤트를 방출하여 완전 종료 될 수 있음
- `completed` 이벤트를 방출하여 완전 종료 될 수 있음

public enum Event<Element> {
    case next(Element)
    case error(Swift.Error)
    case completed
}
```

- `next event` : **최신(또는 "다음") 데이터 값을 "전달"하는 이벤트**이다. observer들이 값을 "수신" 하는 방법이다. Observable은 종료 이벤트가 방출될 때까지 이러한 값을 무한정으로 방출할 수 있다.
- `completed event` : 이 이벤트는 success와 함께 이벤트 순서를 종결한다. Observable은 **lifecycle을 성공적으로 마쳤고, 더 이상 추가적으로 이벤트를 방출하지 않는다.**
- `error event` : **에러와 함께 Observable이 종료**되고, 더 이상 추가적인 이벤트를 방출하지 않는다.

## just

`just` 메서드는 **하나의 element를 포함하는 observable sequence를 생성하는 역할**을 한다.

Observable 타입의 **static method**이다. Rx에서 메서드는 "연산자"로 언급할 수 있다.

```swift
/**
Returns an observable sequence that contains a single element.
- seealso: [just operator on reactivex.io](http://reactivex.io/documentation/operators/just.html)
- parameter element: Single element in the resulting observable sequence.
- returns: An observable sequence containing the single specified element.
*/
public static func just(_ element: Element) -> Observable<Element> {
   return Just(element: element)
}
```

just 메서드를 살펴보니 위와 같았다. element 타입을 매개변수로 받아서 반환형으로 Just 타입을 반환하는데, Just는 무엇인지 알아보았다.

```swift
final private class Just<Element>: Producer<Element> {
   private let _element: Element

   init(element: Element) {
       self._element = element
   }

   override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
       observer.on(.next(self._element))
       observer.on(.completed)
       return Disposables.create()
   }
}
```

오호 이번에는 Just는 Producer를 상속받고 있다. 그럼 Producer는 무엇인지 알아보겠습니다.

```swift
class Producer<Element> : Observable<Element> {
   override init() {
       super.init()
   }

   override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
       if !CurrentThreadScheduler.isScheduleRequired {
           // The returned disposable needs to release all references once it was disposed.
           let disposer = SinkDisposer()
           let sinkAndSubscription = self.run(observer, cancel: disposer)
           disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

           return disposer
       }
       else {
           return CurrentThreadScheduler.instance.schedule(()) { _ in
               let disposer = SinkDisposer()
               let sinkAndSubscription = self.run(observer, cancel: disposer)
               disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

               return disposer
           }
       }
   }

   func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) whereObserver.Element == Element {
       rxAbstractMethod()
   }
}
```

Producer는 Observable<Element> 클래스를 상속받고 있네요. 지금봐서는 뭐가 뭔지 모르겠으나 일단 Producer가 Observable을 상속받고 있으므로 Just도 Observable을 상속받고 있어서 `just` 메서드의 반환타입이 Observable 타입인 것은 확인할 수 있었다.

## of

```swift
let observable2 = Observable.of(one, two, three)
let observable3 = Observable.of([one, two, three])
```

`of` 의 경우에는 `just` 와는 다르게 **타입을 명시하지 않았다**. 그러나 첫 번째 `of` 와 두 번째 `of` 에서의 차이는 다음과 같다. 첫 번째 `of` 의 타입을 확인해보면 Observable<Int>로 나오고 두 번째 `of` 의 타입은 Observable<[Int]> 로 나오게 된다.

그 이유는 `of` 를 살펴보면 알 수 있다.

```swift
/**
    This method creates a new Observable instance with a variable number of elements.
- seealso: [from operator on reactivex.io](http://reactivex.io/documentation/operators/from.html)
- parameter elements: Elements to generate.
- parameter scheduler: Scheduler to send elements on. If `nil`, elements are sent immediately on subscription.
- returns: The observable sequence whose elements are pulled from the given arguments.
*/
public static func of(_ elements: Element ..., scheduler: ImmediateSchedulerType = CurrentThreadScheduler.instance) ->Observable<Element> {
   return ObservableSequence(elements: elements, scheduler: scheduler)
}
```

`of` 연산자의 경우에는 **매개변수로 가변 매개변수를 받기 때문에 Swift가 타입 추론을 할 수 있게 된다**. 따라서 observable array를 생성하고 싶다면  두 번째 `of` 처럼 배열을 매개변수로 넘겨주면 된다.

```swift
final private class ObservableSequence<Sequence: Swift.Sequence>: Producer<Sequence.Element> {
   fileprivate let _elements: Sequence
   fileprivate let _scheduler: ImmediateSchedulerType

   init(elements: Sequence, scheduler: ImmediateSchedulerType) {
       self._elements = elements
       self._scheduler = scheduler
   }

   override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable)where Observer.Element == Element {
       let sink = ObservableSequenceSink(parent: self, observer: observer, cancel: cancel)
       let subscription = sink.run()
       return (sink: sink, subscription: subscription)
   }
}
```

`just` 연산자도 배열을 넘길 수 있지만 차이가 있다. 배열의 원소 하나 하나를 가져가는 것이 아니라 전체 배열 자체를 하나의 element로 가져가는 것이다. 따라서 이러한 결과가 나오게 된다.

```swift
   // 1
   let one = 1
   let two = 2
   let three = 3
   let array = [1,2,3]

   // 2
   let observable_just = Observable<[Int]>.just(array)
   let observable = Observable<Int>.just(one)
   let observable2 = Observable.of(one, two, three)
   let observable3 = Observable.of([one, two, three])

   observable.subscribe() { element in
       print(element)
   }
/* Result
 next(1)
 completed
*/

   observable_just.subscribe() { element in
       print(element)
   }
/* Result
next([1, 2, 3])
completed
*/

   observable2.subscribe { element in
       print(element)
   }
/* Result
next(1)
next(2)
next(3)
completed
*/

   observable3.subscribe { element in
       print(element)
   }
/* Result
next([1, 2, 3])
completed
*/
```

이 결과를 보면서 과연 `of` 연산자를 사용해서 배열을 매개변수로 주는 것과 `just` 연산자에 배열을 넣는 결과가 같아서 둘 의 차이점이 무엇일까 고민해봤는데, 내린 결론은 `just` 는 하나의 값만 보내고 종료되는 반면에 `of` 는 2차원 배열이라면 1차원 배열이 하나씩 나올 수 있다라는 생각이 들었다. 이것이 정확한 차이인지는 모르겠지만, 누구라도 알려주시면 감사하겠습니다.

## from

`from` 연산자는 **입력된 요소의 배열을 개별적인 observable한 요소로 만들어준다**. 따라서 아래와 같은 결과가 나온다.

```swift
let one = 1
let two = 2
let three = 3

let observable4 = Observable.from([one, two, three])
observable4.subscribe { element in
   print(element)
}
/* Result
next(1)
next(2)
next(3)
completed
*/
```

이때 observable4의 타입을 확인해보면 [Int]가 아닌 Int로 나온다.

`from` 연산자는 **매개변수로 배열만을 받을 수 있다.**

# Subscribe

- Observable은 sequence의 정의일 뿐이다.
    
    ⇒ Obervable은 subscriber, 즉 `구독되기 전에는 아무런 이벤트도 보내지 않는다`
    
- Observable이 방출하는 각각의 이벤트 타입에 대해 handler 추가 가능

## subscribe()

```swift
example(of: "subscribe") {
     let one = 1
     let two = 2
     let three = 3
     
     let observable = Observable.of(one, two, three)
     observable.subscribe({ (event) in
    	 print(event)
 	})
 	
 	/* Prints:
 	 next(1)
 	 next(2)
 	 next(3)
 	 completed
 	*/
 }
```

- .subscribe 는 escaping closure로 Int 타입을 이벤트로 가짐
    - 이때 escaping closure의 반환 값은 없음
    - 그리고 .subscribe는 `Disposable` 을 반환함
    - 각각의 요소들에 대해서 `.next` 이벤트를 방출
    - 최종적으로 `completed` 방출

## subscribe(onNext:)

위의 코드에서는 next() 안에 요소들이 포함되어서 출력되었다. 

이때 각 요소 값만 얻고 싶다면 다음과 같이 수정하면 된다.

```swift
observable.subscribe { event in
 	if let element = event.element {
 		print(element)
 	}
 }
 
 /* Prints:
  1
  2
  3
 */
```

이는 Optional처럼 unwrapping 해주는 과정이므로 이를 더 간단하게 축약하면 아래와 같이 고칠 수 있다.

```swift
observable.subscribe(onNext: { (element) in
 	print(element)
 })
 
 /* Prints:
  1
  2
  3
 */
```

`.onNext` 클로저는 `.next` 이벤트만을 매개변수로 취한 뒤 핸들링하고 다른 것들은 모두 무시한다.

## empty()

요소를 하나도 가지지 않는 Observable 은 empty 연산자를 통해 `.completed` 이벤트만 방출시킬 수 있음

```swift
example(of: "empty") {
     let observable = Observable<Void>.empty()
     
     observable.subscribe(
         
         // 1
         onNext: { (element) in
             print(element)
     },
         
         // 2
         onCompleted: {
             print("Completed")
     }
     )
 }
 
 /* Prints:
  Completed
 */
```

Observable은 반드시 특정 타입으로 정의되어야함 

empty의 경우에는 가지고 있는 요소가 없으므로 `Void` 타입으로 설정

🤔 언제 사용할까?

1. 즉시 종료할 수 있는 Observable을 리턴하고 싶을 때
2. 의도적으로 0개의 값을 가지는 Observable 리턴하고 싶을 때

## never()

```swift
example(of: "never") {
     let observable = Observable<Any>.never()
     
     observable
         .subscribe(
             onNext: { (element) in
                 print(element)
         },
             onCompleted: {
                 print("Completed")
         }
     )
 }
```

이렇게 하면 Completed 조차 프린트 되지 않음

⇒ 제대로 동작하는 것이 맞을까..? Debugging을 찍어보자

## range()

```swift
example(of: "range") {
     
     //1
     let observable = Observable<Int>.range(start: 1, count: 10)
     
     observable
         .subscribe(onNext: { (i) in
             
             //2
             let n = Double(i)
             let fibonacci = Int(((pow(1.61803, n) - pow(0.61803, n)) / 2.23606).rounded())
             print(fibonacci)
         })
 }
```

`range` 연산자를 사용해 start부터 count크기 만큼 값을 가지는 observable 생성

방출된 요소의 대한 n번째 피보나치 수를 계산하고 출력

# Disposing

⭐️ 다시 한번 Observable은 subscription을 받기 전까지 아무것도 하지 않음!!

즉, subscription이 Observable이 이벤트를 방출하는 방아쇠 역할을 해줌

그렇다면 **subscription을 취소하면 Observable도 수동적으로 취소할 수 있지 않을까?**

## Dispose()

```swift
example(of: "dispose") {

     // 1
     let observable = Observable.of("A", "B", "C")

     // 2
     let subscription = observable.subscribe({ (event) in

         // 3
         print(event)
     })

     subscription.dispose()
 }
```

1. String의 Observable을 생성
2. Observable를 구독하고 subscribe를 이용해 `Disposable`을 리턴하게 함
3. 출력된 이벤트 출력

이때 구독 취소하려면 `dispose()` 호출 하면됨. 구독 취소나 dispose 이후에는 이벤트 방출이 정지됨

## DisposeBag()

각각의 구독에 대해 일일히 관리하는 것은 비효율적임 

따라서 RxSwift에서 제공하는 `DisposedBag` 타입을 이용할 수 있음

⇒ `DisposeBag` 은 disposables를 가지고 있음

```swift
example(of: "DisposeBag") {
     
     // 1
     let disposeBag = DisposeBag()
     
     // 2
     Observable.of("A", "B", "C")
         .subscribe{ // 3
             print($0)
         }
         .disposed(by: disposeBag) // 4
 }
```

1. dispose bag를 생성
2. observable 생성
3. 방출하는 이벤트 프린트
4. subscribe로부터 방출되는 리턴 값을 disposeBag에 추가

왜 하는것인가?

⇒ dispose bag를 subscription에 추가하거나, 수동적으로 `dispose` 를 호출하지 않으면 memory leak발생


# 참고

1. [raywenderlich chapter 2](https://www.raywenderlich.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/2-observables)