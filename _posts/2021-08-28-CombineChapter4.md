---
title:  "[Combine] Chapter4 Operator - Filtering"
excerpt: "Combine의 operator 중에서 filtering에 관련된 operator를 알아보자"

categories:
  - Combine
tags:
  - [Operator]
last_modified_at: 2021.08.28
---

## 들어가며 
[Chapter3]({{site.url}}{{site.baseurl}}/combine/CombineChapter3)에서는 transforming operator에 대해서 알아보았다.

Chapter4에서는 **filtering과 관련된 operator**에 대해서 알아보자

## Filtering Operator
`Filtering Operator` : publisher에 의해서 방출되는 값이나 이벤트들을 제한하고 싶은 경우에 사용

⚠️ 대부분의 Operator는 `try` 접두사가 붙을 수 있다. 예를 들면 `filter` , `tryFilter` 와 같이 사용할 수 있다. 이 둘의 유일한 차이점은 `try` 가 붙은 operator는 throwing 클로저를 제공한다는 것이다. 따라서 클로저 내부에서 Error를 던지면 해당 error와 함께 publisher를 종료시킨다. 

### filter

> Bool 값을 반환하는 클로저를 가지고 있고, 주어진 조건에 만족하는 값들만 방출된다

![1](/assets/images/Combine/Chapter4/filtering1.png)

```swift
example(of: "filter") {
  // 1
  let numbers = (1...10).publisher
  // 2
  numbers
    .filter { $0.isMultiple(of: 3) }
    .sink(receiveValue: { n in
      print("\(n) is a multiple of 3!")
    })
    .store(in: &subscriptions)
}

// Result
——— Example of: filter ———
3 is a multiple of 3!
6 is a multiple of 3!
9 is a multiple of 3!
```

1. 1부터 10까지의 수를 방출하는 Publisher 생성
2. filter로 3의 배수인 것만 출력하도록 함



### removeDuplicates

> Equatable을 준수하는 값들에 대해서 연속적인 값들 중에서 중복되는 값을 제거하고 방출
>
> ⚠️ 이때 흔히 아는 set과 다르게 모든 중복 값을 제거하는 것이 아니라 연속되는 값들 중에서 중복을 제거하는 것



![2](/assets/images/Combine/Chapter4/filtering2.png)

```swift
example(of: "removeDuplicates") {
  // 1
  let words = "hey hey there! want to listen to mister mister ?"
                  .components(separatedBy: " ")
                  .publisher
  // 2
  words
    .removeDuplicates()
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: removeDuplicates ———
hey
there!
want
to
listen
to
mister
?
```

1. 주어진 문장을 공백에 의해서 분리해서 배열로 가지는 publisher 생성
2. removeDuplicates를 적용해서 값을 출력 



💡 만약 Equatable을 준수하지 않는 값의 경우에는, 클로저에 overload하여 비교할 수 있다. 



### Compacting and ignoring

> publisher가 nil 값을 방출하는 경우에는 이를 처리해야하는데 이때 사용할 수 있다



#### compactMap

> Swift library에 있는 compactMap과 유사하게 nil인 값들은 제외한다

![3](/assets/images/Combine/Chapter4/filtering3.png)

```swift
example(of: "compactMap") {
  // 1
  let strings = ["a", "1.24", "3",
                 "def", "45", "0.23"].publisher
  
  // 2
  strings
    .compactMap { Float($0) }
    .sink(receiveValue: {
      // 3
      print($0)
    })
    .store(in: &subscriptions)
}

// Result 
——— Example of: compactMap ———
1.24
3.0
45.0
0.23
```

1. 문자열을 가지는 publisher 생성

2. Float로 변환하는 compactMap operator를 사용하여 값을 출력하도록함 <br>

   이때 만약 Float로 initializer가 변환할 수 없는 값들은 nil을 반환하게 되고 compactMap에 의해서 nil은 방출되지 않는다



#### ignoreOutput

> publisher가 방출하는 값들은 중요하지 않고, 값을 다 방출하고 끝났는지를 알고 싶을 때 사용

![4](/assets/images/Combine/Chapter4/filtering4.png)

그림처럼 어떠한 값이 방출되어도 상관하지 않고, completion event만 consumer에게 push 해준다.

```swift
example(of: "ignoreOutput") {
  // 1
  let numbers = (1...10_000).publisher
  
  // 2
  numbers
    .ignoreOutput()
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: ignoreOutput ———
Completed with: finished
```

1. 1 - 10,000까지의 값을 방출하는 publisher 생성
2. ignoreOutput operator를 사용해서 값들은 무시하고 completion event만 출력하도록 함



### Finding values

> 조건에 만족하는 값을 찾을 때 사용

#### first(where:)

> 조건에 만족하는 첫 번째 값을 찾으면 방출하며 publisher 종료

![5](/assets/images/Combine/Chapter4/filtering5.png)

`first(where:)` operator는 **lazy** 라고 할 수 있는데, 왜냐하면 조건에 만족하는 값을 찾을 때까지 많은 값들을 필요로하며 조건에 만족하는 경우에는 전체 stream을 cancel하고 complete한다. 

```swift
example(of: "first(where:)") {
  // 1
  let numbers = (1...9).publisher
  
  // 2
  numbers
    .print("numbers")
    .first(where: { $0 % 2 == 0 })
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: first(where:) ———
numbers: receive subscription: (1...9)
numbers: request unlimited
numbers: receive value: (1)
numbers: receive value: (2)
numbers: receive cancel
2
Completed with: finished
```

1. 1 - 9 까지 방출하는 publisher 생성
2. `print` operator를 사용해서 chain에서 어떠한 동작이 일어나는지 확인할 수 있다. 그리고 `first(where:)` operator를 사용해서 첫 번째 짝수를 찾고 출력하도록 함

이때 결과를 보면 실제로 2를 찾고나서 cancel 되고 complete 가되는 것을 확인해 볼 수 있다. 



#### last(where:)

> 조건에 만족하는 마지막 값을 찾으면 방출하며 publisher는 completion event를 받아야지만 종료

![6](/assets/images/Combine/Chapter4/filtering6.png)

`first(where:)` operator와는 다르게 `last(where:)` 은 **greedy** 하다고 할 수 있는데, 왜냐하면 방출되는 모든 값들이 조건에 매칭되는지 알아야하므로 기다려야하기 때문이다. 그렇기 때문에 **upstream은 언젠가 complete를 하는 publisher여야만 한다**. 

```swift
example(of: "last(where:)") {
  let numbers = PassthroughSubject<Int, Never>()
  
  numbers
    .last(where: { $0 % 2 == 0 })
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
  
  numbers.send(1)
  numbers.send(2)
  numbers.send(3)
  numbers.send(4)
  numbers.send(5)
  numbers.send(completion: .finished)
}

// Result 
——— Example of: last(where:) ———
4
Completed with: finished
```

```swift
let numbers = (1...9).publisher
```

⚠️ 위와 같은 publisher는 1 - 9 까지 방출하고 completion 되지만, 만약 위의 예시에서 `send(completion: .finished)` 가 없는 경우에는 `last(where:)` operator는 언제 끝날지 모르기 때문에 계속 기다리게 된다. 

따라서 `last(where:)` 은 반드시 upstream이 complete 되는 것과 사용해야한다. 



### Dropping values

> 값을 dropping 하는 것은 여러 publisher와 일할 때 유용하다. 예를 들어 2개의 publisher가 있을 때, 첫 번째 publisher가 방출하는 값을 두 번째 publisher가 방출하기 전까지나, 혹은 특정 개수를 무시하고 싶을 때 사용할 수 있다

#### dropFirst

> count 매개변수와 함께 사용되며, count의 default = 1 이고 publisher가 방출하는 count 개의 값을 무시 

![7](/assets/images/Combine/Chapter4/filtering7.png)

```swift
example(of: "dropFirst") {
  // 1
  let numbers = (1...10).publisher
  
  // 2
  numbers
    .dropFirst(8)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result 
——— Example of: dropFirst ———
9
10
```

1. 1 - 10 까지 방출하는 publisher 생성
2. `dropFirst` operator를 사용해서 8개를 무시하고 이후는 값을 출력하도록함



#### drop(while:)

> 조건을 제시하는 클로저와 함께 사용되며, 해당 조건을 만족하는 값이 나오기 전까지 publisher가 방출하는 값들을 모두 무시

![8](/assets/images/Combine/Chapter4/filtering8.png)

```swift
example(of: "drop(while:)") {
  // 1
  let numbers = (1...10).publisher
  
  // 2
  numbers
    .drop(while: {
      print("x")
      return $0 % 5 != 0
    })
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: drop(while:) ———
x
x
x
x
x
5
6
7
8
9
10
```

1. 1 - 10 까지 방출하는 publisher 생성
2. `drop(while:)` operator를 사용해서 x를 계속 프린트하며 5의 배수가 나오기 전까지는 값을 무시하고 5의 배수 이후에는 값을 출력하도록 한다



💡 `filter` vs `drop(where:)` 

두 연산자 모두 방출되는 값들을 통제하기 위한 조건을 클로저로 가지는 점은 공통점이다. 

`filter` 

- 클로저에서 true를 반환하는 경우에 값은 throuhg(통과) 된다. 
- publisher가 방출하는 모든 값들에 대해서 조건을 검사하는 것을 멈추지 않고 모두 수행한다. 즉, 앞의 값이 true를 반환했다고 해서 다음 값을 검사하지 않는 것이 아니라 다음 값도 클로저에서 검사를 하게 된다.

`drop(where:)` 

- 클로저에서 true를 반환하는 동안에는 값은 skip(건너뜀) 된다.  
- 클로저 내부에서 조건을 처음으로 만족하게 되면, 그 이후에 다시는 조건을 검사하지 않는다. <br>따라서 위의 코드 예시에서 `print(x)` 가 5 전까지만 실행되고, 조건을 만족한 이후에는 실행되지 않는 것을 확인할 수 있다.



#### drop(untilOutputFrom:)

> 가장 정교한 filtering operator라고 할 수 있다.

![9](/assets/images/Combine/Chapter4/filtering9.png)

사용자가 button을 tap할 수 있고, 이때 isReady인 상태에서만 해당 값들을 받고 싶은 경우에 `drop(untilOutputFrom:)` operator를 사용하면 된다. 

```swift
example(of: "drop(untilOutputFrom:)") {
  // 1
  let isReady = PassthroughSubject<Void, Never>()
  let taps = PassthroughSubject<Int, Never>()
  
  // 2
  taps
    .drop(untilOutputFrom: isReady)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
  
  // 3
  (1...5).forEach { n in
    taps.send(n)
    
    if n == 3 {
      isReady.send()
    }
  }
}

// Result
——— Example of: drop(untilOutputFrom:) ———
4
5
```

1. 2개의 PassthroughSubject 타입의 변수 생성
2. `drop(untilOutputFrom:)` operator를 사용해서 isReady에서 값이 나오기 전까지는 값들을 무시하고, 이후에는 값을 출력하도록 한다.
3. 1 - 5 까지 taps를 보내는데, 3 일때 isReady가 보내지도록 한다. 

따라서 유저가 5번 탭을 하고 3번째 이후에 isReady 상태가되므로 결과가 1,2,3번째 탭은 무시되고 4,5만 출력되는 것을 확인해 볼 수 있다. 

### Limiting values

> drop 하는 것이 클로저 내부의 조건을 만족하기 전까지 버리는 것이였다면, limit는 해당 조건을 만족하는 값들을 받으며 publisher가 complete되도록 만드는 것이다. 

#### prefix

> dropFirst와는 반대로 주어진 값만큼만 방출된 값을 가지고 complete

![10](/assets/images/Combine/Chapter4/filtering10.png)

`first(where:)` operator와 같이 `prefix` 도 **lazy** 라고 할 수 있는데, 필요한 만큼만 값을 받고 종료하기 때문이다. 게다가 numbers가 추가로 값을 생성하지 않도록 해주기까지 한다. 

```swift
example(of: "prefix") {
  // 1
  let numbers = (1...10).publisher
  
  // 2
  numbers
    .prefix(2)
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: prefix ———
1
2
Completed with: finished
```

1. 1 - 10 까지 방출하는 publisher 생성
2. `prefix()` operator를 사용하여 2개까지만 값을 받고 complete event를 보내도록 함



#### prefix(while:)

> 조건을 가지는 클로저를 가지고 있으며, 해당 조건을 만족하기 전까지는 upstream이 방출하는 값들을 통과하도록 하고, 만족하지 못하면 그 즉시 publisher는 complete 된다

![11](/assets/images/Combine/Chapter4/filtering11.png)

```swift
example(of: "prefix(while:)") {
  // 1
  let numbers = (1...10).publisher
  
  // 2
  numbers
    .prefix(while: { $0 < 3 })
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
}

// Result
——— Example of: prefix(while:) ———
1
2
Completed with: finished
```

1. 1 - 10 까지 방출하는 publisher 생성
2. `prefix(while:)` operator를 사용해서 3보다 작을 때까지는 값들이 통과되어 출력되고, 3 이상의 값들이 오는 순간 바로 publisher 는 complete 되도록 한다.



#### prefix(untilOutputFrom:)

> second publisher가 값을 방출 할 때까지 값들을 skip하는 drop(untilOutputFrom:) 과는 반대로 second publisher가 값을 방출하기 전까지 값들을 가지게 된다. 

![12](/assets/images/Combine/Chapter4/filtering12.png)

사용자가 딱 두 번만 tap 할 수 있는 버튼이 있다고 할 때, 빠르게 두번의 tap이 일어나면 이후의 tap들은 무시되어야하는 경우에 이 연산자를 사용할 수 있다. 

```swift
example(of: "prefix(untilOutputFrom:)") {
  // 1
  let isReady = PassthroughSubject<Void, Never>()
  let taps = PassthroughSubject<Int, Never>()
  
  // 2
  taps
    .prefix(untilOutputFrom: isReady)
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
  
  // 3
  (1...5).forEach { n in
    taps.send(n)
    
    if n == 2 {
      isReady.send()
    }
  }
}

// Result 
——— Example of: prefix(untilOutputFrom:) ———
1
2
Completed with: finished
```

1. 2개의 PassthroughSubject 타입의 변수 생성
2. `prefix(untilOutputFrom:)` operator를 사용해서 isReady가 오기전까지만 값을 받고 이후에는 publihser가 complete을 하도록 함
3. 1 - 5 까지 tap이 보내지는데, 이때 두 번째 이후에 isReady가 보내진다.

따라서 결과를 보면 2번 tap이 이뤄져서 isReady가 보내지기 전까지의 값들은 출력되고 2번 탭이 되고 isReady가 보내지는 즉시 publisher는 complete되며 종료된다. 

### Challenge

1 - 100까지 방출하는 publisher에서 다음의 조건을 만족하도록 한다.

1. Skip first 50 values emitted by the upstream publisher
2. Take next 20 values after those first 50
3. Only take even numbers
4. 결과는 다음과 같이 `52 54 56 58 60 62 64 66 68 70`  출력되어야하면 한 숫자당 한 라인에 출력되어야한다. 

```swift
import Foundation
import Combine

var subscriptions = Set<AnyCancellable>()

let numbers = (1...100).publisher

numbers
    .dropFirst(50)
    .prefix(20)
    .filter { $0 % 2 == 0 }
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```


## 마치며
이번 포스팅에서는 Operator 중에서 Publisher로부터 방출된 값들을 필터링 할 수 있는 operator들에 대해서 알아볼 수 있었다. 

[Chapter5]({{site.url}}{{site.baseurl}}/combine/CombineChapter5)에서는 combining과 관련된 operator에 대해서 알아보자.


## 참고
[Raywenderlich Combine Chapter3: Transforming Operators](https://www.raywenderlich.com/books/combine-asynchronous-programming-with-swift/v2.0/chapters/3-transforming-operators)