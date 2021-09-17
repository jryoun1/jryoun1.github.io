---
title:  "[Combine] Chapter5 Operator - Combining"
excerpt: "Combine의 operator 중에서 combining에 관련된 operator를 알아보자"

categories:
  - Combine
tags:
  - [Operator]
last_modified_at: 2021.09.01
---

## 들어가며 
[Chapter4]({{site.url}}{{site.baseurl}}/combine/CombineChapter4)에서는 filtering operator에 대해서 알아보았다.

Chapter5에서는 **combining과 관련된 operator**에 대해서 알아보자

## Combining Operator
`Combining Operator` : 다른 publisher들로부터 방출되는 이벤트들을 combine하고 의미있는 data로 만들어 내는 연산자이다. 

예를 들면 login 화면에서 사용자로부터 여러 입력을 받아야한다고 해보자. 이때 이러한 데이터들을 combine해서 의미있는 관계를 만들어야한다. 예를 들면 필수로 입력되어야하는 것이라던가 password, check password의 관계라던가..

### Prepending

> 원래 publisher의 요소보다 먼저 방출될 요소들을 추가하는데 사용한다

#### predend(Output...)

> variadic(가변) 리스트를 input으로 받게 된다. 즉, publisher의 original `Output` 타입과 일치하는 어떠한 요소를 가변 매개변수로 넣어줄 수 있다.

![1](/assets/images/Combine/Chapter5/combining1.png)

```swift
example(of: "prepend(Output...)") {
  // 1
  let publisher = [3, 4].publisher

  // 2
  publisher
    .prepend(1, 2)
    .prepend(-1, 0)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: prepend(Output...) ———
-1
0
1
2
3
4
```

이때 last prepend가 upstream first에 영향을 미치기 때문에, -1, 0 이 prepend되고 그리고 다시 1, 2, 3, 4 로 결과가 출력되는 것이다.



#### prepend(Sequence)

> prepend(Output...)과 비슷하지만 `Sequence` 프로토콜을 채택하는 객체를 input으로 가진다는 점에서 다르다. 예를 들면 Set, Array, Dictionary와 같은 것이 있다. 

![2](/assets/images/Combine/Chapter5/combining2.png)

```swift
example(of: "prepend(Sequence)") {
  // 1
  let publisher = [5, 6, 7].publisher
  
  // 2
  publisher
    .prepend([3, 4])
    .prepend(Set(1...2))
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: prepend(Sequence) ———
1
2
3
4
5
6
7
```

 ⚠️이때 Array와는 다르게 Set의 경우에는 순서가 정해지지 않기 때문에 1,2나 2,1 순으로 나올 수 있다는 점을 알아야한다. 

```swift
.prepend(stride(from: 6, to: 11, by: 2))

// Result
——— Example of: prepend(Sequence) ———
6
8
10
1
2
3
4
5
6
```

위의 코드를 추가하면 `Strideable` 을 생성하는데, `Strideable` 은  `Sequence` 를 채택하므로 input으로 사용할 수 있다. 따라서 6과 11사이에서 2만큼씩 건너뛰는 숫자들을 prepend하게 된다. 



#### prepend(Publisher)

> 앞의 두 연산자가 원소들의 리스트를 publisher에 붙이는 것이였다면, 이 연산자는 2개의 publisher가 있을 때 하나의 publisher를 다른 하나의 publisher에 붙일 때 사용할 수 있다

![3](/assets/images/Combine/Chapter5/combining3.png)

```swift
example(of: "prepend(Publisher)") {
  // 1
  let publisher1 = [3, 4].publisher
  let publisher2 = [1, 2].publisher
  
  // 2
  publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: prepend(Publisher) ———
1
2
3
4
```

기대했던 대로 publisher2를 publisher1 앞에 붙여서 결과가 출력된다. 

```swift
example(of: "prepend(Publisher) #2") {
  // 1
  let publisher1 = [3, 4].publisher
  let publisher2 = PassthroughSubject<Int, Never>()
  
  // 2
  publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

  // 3
  publisher2.send(1)
  publisher2.send(2)
  publisher2.send(completion: .finished)
}

// Result
——— Example of: prepend(Publisher) ———
1
2
3
4
```

위와 같이 PassthroughSubject의 경우에는 만약 `send()` 값만 보내고 `send(completion:_)` 을 보내지 않는다면 끝났는지 안끝났는지 알 수 없기 때문에 **prepend당하는 publisher는 반드시 complete 를 보내야만 한다.** 



### Appending

> prepend와 비슷하지만 앞이 아닌 뒤에 이벤튼나 요소들을 붙여줄 때 사용한다 



#### append(Output...)

> prepend(Output...)과 비슷하게 가변 리스트를 input으로 받는데 대신에 기존의 publisher가 `.finished` omplete 된 시점에 추가된다

![4](/assets/images/Combine/Chapter5/combining4.png)

```swift
example(of: "append(Output...)") {
  // 1
  let publisher = [1].publisher

  // 2
  publisher
    .append(2, 3)
    .append(4)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: append(Output...) ———
1
2
3
4
```

이때 각각의 `append` operator는 upstream이 complete 되기를 기다렸다가 이후에 동작되며, **upstream publisher는 반드시 complete하는 publisher**여야만 한다. 

```swift
example(of: "append(Output...) #2") {
  // 1
  let publisher = PassthroughSubject<Int, Never>()

  publisher
    .append(3, 4)
    .append(5)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
  
  // 2
  publisher.send(1)
  publisher.send(2)
  publisher.send(completion: .finished)
}

// Result 
——— Example of: append(Output...) #2 ———
1
2
3
4
5
```

이때 만약 `send(completion: .finished)` 가 없다면 뒤에 append 연산자들은 동작되지 못한다. 이러한 점은 모든 append 연산자에서 동일하게 적용되므로 upstream publisher는 반드시 complete 되어야만 한다. 



#### append(Sequence)

> Publisher의 output 타입과 일치하는 `Sequence` 프로토콜을 따르는 모든 object는 이 연산자를 사용해서 append할 수 있다

![5](/assets/images/Combine/Chapter5/combining5.png)

```swift
example(of: "append(Sequence)") {
  // 1
  let publisher = [1, 2, 3].publisher
    
  publisher
    .append([4, 5]) // 2
    .append(Set([6, 7])) // 3
    .append(stride(from: 8, to: 11, by: 2)) // 4
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: append(Sequence) ———
1
2
3
4
5
7
6
8
10
```

이때도 역시 Set은 Array와 다르게 순서가 일정하지 않으므로 6,7 이나 7,6으로 나올 수 있고 `Strideable` 도 `Sequence` 프로토콜을 따르기 때문에 사용할 수 있다. 



#### append(Publisher)

> 다른 Publisher를 original publisher 뒤에 붙일 때 사용한다

![6](/assets/images/Combine/Chapter5/combining6.png)

```swift
example(of: "append(Publisher)") {
  // 1
  let publisher1 = [1, 2].publisher
  let publisher2 = [3, 4].publisher
  
  // 2
  publisher1
    .append(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result 
——— Example of: append(Publisher) ———
1
2
3
4
```

 

### Advanced combining

다른 publisher들을 combining 하는 복잡한 방법에 대해서 알아볼 것이다.



#### switchToLatest

> 전체 publisher chain들을 바꾸는데, 이때 pending되어있는(현재 들어가있는) publisher를 취소하고 최신의 것으로 switch한다. publisher를 방출하는 publisher들에서만 사용할 수 있다

![7](/assets/images/Combine/Chapter5/combining7.png)


```swift
example(of: "switchToLatest") {
  // 1
  let publisher1 = PassthroughSubject<Int, Never>()
  let publisher2 = PassthroughSubject<Int, Never>()
  let publisher3 = PassthroughSubject<Int, Never>()

  // 2
  let publishers = PassthroughSubject<PassthroughSubject<Int, Never>, Never>()

  // 3
  publishers
    .switchToLatest()
    .sink(receiveCompletion: { _ in print("Completed!") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)

  // 4
  publishers.send(publisher1)
  publisher1.send(1)
  publisher1.send(2)

  // 5
  publishers.send(publisher2)
  publisher1.send(3)
  publisher2.send(4)
  publisher2.send(5)

  // 6
  publishers.send(publisher3)
  publisher2.send(6)
  publisher3.send(7)
  publisher3.send(8)
  publisher3.send(9)

  // 7
  publisher3.send(completion: .finished)
  publishers.send(completion: .finished)
}

// Result
——— Example of: switchToLatest ———
1
2
4
5
7
8
9
Completed!
```

1. 3개의 Int, Never 타입을 가지는 PassthroughSubject 타입의 변수 생성
2. 위에서 생성한 타입, Never 타입을 가지는 PassthroughSubject 타입의 변수 생성
3. `switchToLatest` operator를 사용하여 최신의 것이 들어오면 pending 된 것을 취소하고 switch하도록함

결과를 보면 publisher2가 send된 이후에 publisher1으로 보내는 값이나, publisher3가 send된 이후 publisher2로 보내지는 값들은 받아지지 않는 것을 확인해 볼 수 있다. 그리고 마지막에는 publisher 3에 대해서 finished를 보내고 전체에 대해서 finished를 보내준다.

💡 이러한 연산자는 실제로 다음과 같은 시나리오에서 사용할 수 있다. 예를 들어 네트워킹을 하는 버튼을 유저가 눌르고 난 뒤, 네트워킹이 진행되는 도중에 다른 버튼을 눌렀을 때 해당 버튼이 동작하게 하려면 pending 된 네트워킹을 취소하고 새롭게 하도록 하기 위해서는 이러한 연산자를 사용해야한다. 

```swift
example(of: "switchToLatest - Network Request") {
  let url = URL(string: "https://source.unsplash.com/random")!

  // 1
  func getImage() -> AnyPublisher<UIImage?, Never> {
      URLSession.shared
                .dataTaskPublisher(for: url)
                .map { data, _ in UIImage(data: data) }
                .print("image")
                .replaceError(with: nil)
                .eraseToAnyPublisher()
  }

  // 2
  let taps = PassthroughSubject<Void, Never>()

  taps
    .map { _ in getImage() } // 3
    .switchToLatest() // 4
    .sink(receiveValue: { _ in })
    .store(in: &subscriptions)

  // 5
  taps.send()

  DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    taps.send()
  }
  DispatchQueue.main.asyncAfter(deadline: .now() + 3.1) {
    taps.send()
  }
}

// Result
——— Example of: switchToLatest - Network Request ———
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x600000f702d0 anonymous {1080, 1350}>))
image: receive finished
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive cancel
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x600000f6c120 anonymous {1080, 1617}>))
image: receive finished
```

따라서 결과를 보면 맨 처음 tap은 정상적으로 작동하지만, 3초 뒤에 tap과 3.1초 이후에 tap은 앞서 pending된 3초에 시작된 tap이 cancel되고 3.1초에 시작된 tap이 작동하는 것을 확인할 수 있다.



#### merge(with:)

> 같은 타입의 서로 다른 publisher들의 방출되는 값들을 사이사이에 끼워서 합쳐준다 

![8](/assets/images/Combine/Chapter5/combining8.png)

```swift
example(of: "merge(with:)") {
  // 1
  let publisher1 = PassthroughSubject<Int, Never>()
  let publisher2 = PassthroughSubject<Int, Never>()

  // 2
  publisher1
    .merge(with: publisher2)
    .sink(receiveCompletion: { _ in print("Completed") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)

  // 3
  publisher1.send(1)
  publisher1.send(2)

  publisher2.send(3)

  publisher1.send(4)

  publisher2.send(5)

  // 4
  publisher1.send(completion: .finished)
  publisher2.send(completion: .finished)
}

// Result
——— Example of: merge(with:) ———
1
2
3
4
5
Completed
```

따라서 중간중간에 다른 publisher에서 보내는 값들이 합쳐져서 출력되는 것을 확인할 수 있다.



#### combineLatest

> 같은 타입뿐만 아니라 다른 타입의 publisher끼리도 combine할 수 있게 해준다. 그러나 Publisher의 방출되는 값들 사이사이에 끼워서 합쳐주는 것이 아니라, 해당 publisher의 **가장 최신의 것과 지금 방출되는 값을 tuple형태로 반환**해준다. 그리고 이때 original publisher이외의 **다른 모든 publisher들이 적어도 하나의 값을 방출해야지** combineLatest가 동작한다

![9](/assets/images/Combine/Chapter5/combining9.png)

```swift
example(of: "combineLatest") {
  // 1
  let publisher1 = PassthroughSubject<Int, Never>()
  let publisher2 = PassthroughSubject<String, Never>()

  // 2
  publisher1
    .combineLatest(publisher2)
    .sink(receiveCompletion: { _ in print("Completed") },
          receiveValue: { print("P1: \($0), P2: \($1)") })
    .store(in: &subscriptions)

  // 3
  publisher1.send(1)
  publisher1.send(2)
  
  publisher2.send("a")
  publisher2.send("b")
  
  publisher1.send(3)
  
  publisher2.send("c")

  // 4
  publisher1.send(completion: .finished)
  publisher2.send(completion: .finished)
}

// Result
——— Example of: combineLatest ———
P1: 2, P2: a
P1: 2, P2: b
P1: 3, P2: b
P1: 3, P2: c
Completed
```

즉, 여기서 보면 1은 분명히 publisher1을 통해서 방출되었지만, combineLatest에 push through하지 못했다. 왜냐하면 combineLatest가 위에서도 말했듯이 **original publisher를 제외한 다른 모든 publisher들이 적어도 하나의 값을 방출한 이후에 동작**하기 때문이다. 따라서 publisher2가 "a"를 send한 시점부터 조건이 true가 된다고 할 수 있다. 



#### zip

> Swift 라이브러리의 zip과 비슷하게 동작한다. 같은 index의 방출되는 값들을 pair하게 tuple 형태로 방출한다. 이때 만약 2개의 publisher를 zip한다면 다른 하나의 publisher가 해당 index의 값을 방출할 때 까지 기다렸다가 방출되면 tuple 형태로 내보낸다

![10](/assets/images/Combine/Chapter5/combining10.png)

```swift
example(of: "zip") {
  // 1
  let publisher1 = PassthroughSubject<Int, Never>()
  let publisher2 = PassthroughSubject<String, Never>()

  // 2
  publisher1
    .zip(publisher2)
    .sink(receiveCompletion: { _ in print("Completed") },
          receiveValue: { print("P1: \($0), P2: \($1)") })
    .store(in: &subscriptions)

  // 3
  publisher1.send(1)
  publisher1.send(2)
  publisher2.send("a")
  publisher2.send("b")
  publisher1.send(3)
  publisher2.send("c")
  publisher2.send("d")

  // 4
  publisher1.send(completion: .finished)
  publisher2.send(completion: .finished)
}

// Result
——— Example of: zip ———
P1: 1, P2: a
P1: 2, P2: b
P1: 3, P2: c
Completed
```

이를 통해서 zip된 Publisher들끼리는 서로 해당 index의 값이 방출되기까지를 기다리고, 모든 publisher가 해당 Index의 값을 방출했을 때 tuple형태로 보내는 것을 확인할 수 있다.


## 마치며
이번 포스팅에서는 Operator 중에서 Publisher로부터 방출된 값들을 combining 할 수 있는 operator들에 대해서 알아볼 수 있었다. 

[Chapter6]({{site.url}}{{site.baseurl}}/combine/CombineChapter6)에서는 timing과 관련된 operator에 대해서 알아보자.


## 참고
[Raywenderlich Combine Chapter5: Combining Operators](https://www.raywenderlich.com/books/combine-asynchronous-programming-with-swift/v2.0/chapters/5-combining-operators)