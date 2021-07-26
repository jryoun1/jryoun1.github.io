---
title:  "[Swift] Escaping Closure"
excerpt: "클로저 중에서 @escaping 키워드를 붙이는 escaping 클로저에 대해서 알아보자."

categories:
  - Swift
tags:
  - [escaping closure]
last_modified_at: 2021.06.23
---
## 들어가며
네트워크 통신을 하여 비동기적으로 서버로부터 데이터를 받아오기 위해서 `dataTask(with:)` 매서드를 사용하던 도중에 메서드를 확인해보니 아래와 같이 `@escaping` 키워드를 포함하는 클로저를 가지고 있었다.  
```swift 
func dataTask(with request: URLRequest, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
``` 
이에 escaping 클로저가 정확히 무엇이고 왜 사용하는지 알아보기 위해서 이 포스팅을 작성한다. 

## escaping closure

> A closure is said to escape a function when the closure is passed as an argument to the function, but is called after the function returns.

`Escaping closure` 는 클로저가 함수의 인자로 전달됐을 때, 함수의 실행이 종료된 후에 실행되는 클로저이다.

클로저가 함수로부터 **escape**한다는 것은 해당 함수의 인자로 클로저가 전달되지만, **함수가 반환된 후 실행되는 것**을 의미한다. 함수의 인자가 함수의 영역을 탈출하여 함수 밖에서 사용할 수 있는 개념은 기존에 우리가 알고 있는 변수의 scope 개념을 무시한다. 왜냐하면 함수에서 선언된 로컬 변수가 로컬 변수의 영역을 뛰어넘어 함수 밖에서도 유효하기 때문이다.

일반 로컬 변수가 함수 밖에서 살아있는 것은 전역변수를 함수에 가져와서 사용하는 것과 크게 다르지 않다. 그러나 클로저의 escaping은 **A 함수가 마무리 된 상태에서만 B 함수가 실행되도록 함수를 작성할 수 있다는 점**에서 유용하다. <br>
즉, escaping closure를 활용하면 **함수 사이에 실행 순서를 정할 수 있다.** 

> Escaping Closure를 통해서 클로저 인자는 함수로부터 빠져나올 수(outLive) 있다. Swift3 이후부터는 기본적으로 함수의 인자로 들어오는 클로저가 함수 밖에서 사용할 수 없도록 되어 있다. 즉 기본적으로 클로저를 함수 외부의 저장소에 저장하거나, GCD를 이용하여 다른 쓰레드에서 해당 클로저를 실행시키는 것이 불가능하다. 하지만 이는 Escaping Closure를 통해 사용 가능하고, 클로저 타입 앞에 @escaping 키워드를 넣어주면 클로저는 Escaping Closure가 됩니다.

### 클로저를 함수 외부에 저장하는 예시

```swift
var completionHandlers: [() -> Void] = []

func withEscaping(completion: @escaping () -> Void) {
    completionHandlers.append(completion)
}
func withoutEscaping(completion: () -> Void) {
    completion()
}

class MyClass {
    var x = 10
    func callFunc() {
        withEscaping { self.x = 100 }
        withEscaping { self.x = 400 }
        withoutEscaping { x = 200 }
    }
}

let mc = MyClass()
completionHandlers.first?() // nil
print(mc.x) // 10

mc.callFunc()
print(mc.x) // 200

completionHandlers.first?()
print(mc.x) // 100

completionHandlers.last?()
print(mc.x) // 400
```

위의 예시에서는 MyClass의 함수 `callFunc()`는 클로저를 인자를 가지는 `withEscaping(completion:)`과 `withoutEscaping(completion:)`을 각각 호출한다. 

이 때 `withEscaping(completion:)`은 `completion`의 파라미터가 `Escaping Closure` 형태로 구현되어 있다. 위의 예제에서는 `completionHandlers.append(completion)`코드를 통해 `withEscaping(completion:)` 외부에 클로저를 저장한다. 

즉, 클로저가 함수에서 빠져나갔다. 이렇게 함수를 호출하는 도중에 **해당 함수 외부에 클로저를 저장하기 위해서는 클로저는 `Escaping Closure`이어야 한다.**


## non-escaping closure

`Non-Escaping closure` 는 이와 반대로 함수의 실행이 종료되기 전에 실행되는 클로저이다.

```swift
func runClosure(closure: () -> Void) {
  closure()
}
```

### 클로저가 실행되는 순서

1. 클로저가 `runClosure()` 함수의 `closure` 인자로 전달됨
2. 함수 안에서 `closure()` 가 실행됨
3. `runClosure()` 함수가 값을 반환하고 종료됨

이렇게 클로저가 함수가 종료되기 전에 실행되기 때문에 `closure`는 Non-Escaping 클로저 이다.


## 두 개의 차이점

`@escaping` 이 붙은 클로저에는 반드시 escaping 클로저만 사용하는 것이 아닌 non-escaping 클로저도 인자로 넣을 수 있다. <br>
반면에 escaping closure를 `@escaping` 없이는 사용이 불가능하다. 

> 그렇다면 다 escaping 붙이고 그냥 사용하지 뭐하러 나눠서 사용하는가?

이렇게 escaping, non-escaping 클로저를 나눠서 사용하는 이유는 컴파일러의 **퍼포먼스와 최적화 때문**이다.

non-escaping 클로저는 컴파일러가 클로저의 실행이 언제 종료되는지 알기 때문에, 때에 따라 클로저에서 사용하는 특정 객체에 대한 retain, release 등의 처리를 생략해 객체의 라이프 사이클(life cycle)을 효율적으로 관리할 수 있다.

반면 esacping 클로저는 함수 밖에서 실행되기 때문에 클로저가 함수 밖에서도 적절히 실행되는 것을 보장하기 위해, 클로저에서 사용하는 객체에 대한 추가적인 참조 사이클(reference cycles) 관리 등을 해줘야한다. 이 부분이 컴파일러의 퍼포먼스와 최적화에 영향을 끼치기 때문에 Swift에서는 필요할 때만 escaping 클로저를 사용하도록 구분한다. 

