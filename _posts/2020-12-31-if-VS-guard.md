---
title:  "[Swift]if VS guard"
excerpt: "Swift로 개발하는 사람이면 if와 guard의 사용에 대해서 한 번쯤은 고민을 해봤을 것 같다."

categories:
  - Swift
tags:
  - [guard, if, optional binding]
toc: true
toc_sticky: true
last_modified_at: 2020.12.31
---

## 들어가며
Swift로 개발하는 사람이면 if와 guard의 사용에 대해서 한 번쯤은 고민을 해봤을 것 같다.
언제 if를 사용하고 언제 guard를 사용할 지에 대한 고민을 말이다.
나는 Swift 이전에 C++을 주 언어로 사용했었기 때문에 guard 보다는 익숙한 if 를 자주 사용했던 것 같다.
하지만 이제 Swift를 주 언어로 사용하는 이상 어느 시점에 무엇을 써야할 지 스스로 기준을 세워보려고 한다. 

먼저 기준을 세우기 전에 if와 guard에 대해서 간단히 알아보자!

## if
Apple의 공식 문서 중 하나인 Swift Langauage Guideline의 Control Flow에 다음과 같이 나와있다. 

![](https://images.velog.io/images/minni/post/1367b1ad-a3da-49e4-b4cf-0381eab499ea/image.png) 

Swift는 2가지 방법으로 우리의 코드에서 조건들이 여러 개인 경우 처리할 수 있는 방법을 제공해 준다고 한다. 하나는 `if` 문 다른 하나는 `switch` 문이라고 한다. 이 중에서 오늘은 `if` 문에 대해서 알아보자.

위의 문서에서는 `if` 문은 `switch` 문에 비하면 **가능한 결과의 개수가 많지 않은 조건들**에서 주로 사용한다고 한다. 또한 `if` 문은 특정 조건이 참이거나 거짓인 경우에 사용하게 된다. 
예를 들어 `if` 문이 한 개인 경우에는 조건이  **항상 참인 경우에만** 실행이된다.

```swift
var age = 25
if age < 30 {
   print("you are still very young🔥!!")
}
// you are still very young🔥!!
```

반면에 만약 `if - else` 문이라면 해당 조건이 **참인 경우와 거짓인 경우에도 실행**이 된다. 

```swift
var age = 32
if age < 30 {
   print("you are still very young🔥!!")
}
else {
   print("it's sad but you are old😢")
}
// it's sad but you are old😢
```
또한 `if - else if - else` 문을 사용해서 조건을 여러 개 사용할 수도 있다. 
```swift
var age = 80
if age < 30 {
   print("you are still very young🔥!!")
}
else if age < 60 {
   print("it's sad but you are old😢")
}
else {
   print("now, you might become grandfather🧙‍♂️")
}
// now, you might become grandfather🧙‍♂️
```
이때 마지막 `else` 는 있어도 되고 없어도 되는데, 즉 optional한데 만약 특정 조건이 꼭 수행되지 않아야하는 경우 생략할 수도 있다.

### Optional binding using if

```swift
func printName() {
    var value: String?
    value = "jake"
    print(value) // Optional("jake")
    if let name = value {
        print(name) // "jake"
    }
}
```
위의 코드에서처럼 `if let` 을 통해 옵셔널 바인딩 된 상수의 경우에는 해당 `if` 블럭 안에서만 상수를 사용할 수 있다. 

## guard
guard에 관한 내용도 공식문서에는 다음과 같이 나와있다. 
![](https://images.velog.io/images/minni/post/1fbebb2c-3f28-4f33-9e83-2a3d0e817124/image.png) `guard` 문은 `if` 문과 비슷하게 참인지 거짓인지, 즉 `Boolean` 값에 따라서 실행이 된다. 
그러나 `guard` 문은 그 다음 문장들을 실행하기 위해서는 조건이 반드시 참이어야만 한다. 
`if` 문과는 다르게 `guard` 문은 항상 `else` 구문이 있고, `else` 구문 안에서는 조건이 거짓일 때 실행되는 코드들이 들어가 있어야한다. 위에 제목처럼 참이 아니면 빠르게 탈출한다는 의미인 것 같은데 알아보자.
```swift
func buyFood(with money: Int?) {
    guard let moneyInPocket = money, moneyInPocket > 5000 else {
    	print("I have not enough money in my pocket😢")
    	return 
    }
    print("I can buy 🥐🥖 with \(moneyInPocket)")
}

buyFood(with: 10000)
// I can buy 🥐🥖 with 10000
buyFood(with: 5000)
// I have not enough money in my pocket😢
```
위의 코드를 보면 매개변수로 `Int` 타입이 들어오면 옵셔널인지 `guard let` 구문에서 확인하고, 5000원 보다 큰 돈 인지 확인한다. 이때 만약 매개변수가 nil,즉 돈이 없거나 매개변수가 5000보다 작은 경우에는 돈이 부족하다고 출력되고`Int` 타입의 돈이 5000보다 많다면 빵을 살 수 있다고 출력 된다. 

즉, `if` 문과는 다르게 해당 조건을 만족해야지만 아래의 코드를 실행하게 되고, 조건을 만족하지 못하면 `else` 구문안에 코드를 실행한다. 이때 `else` 문 안에는 `break, continue, return, throw` 와 같이 제어를 넘겨주거나 종료 시켜주는 keyword가 포함 되어야하며 만약 제어를 넘길 함수나 return 값이 없는 경우에는 `fatalError(_:file:line:)`를 호출 해야 한다.

### Optional binding using guard
```swift
func printName() {
    var value: String?
    value = "jake"
    print(value) // Optional("jake")
    guard let name = value else {
        return
    }
    print(name)
}
```
`Boolean` 타입의 값으로 `guard` 문을 사용할 수도 있지만 `guard let` 구문을 통해서 `옵셔널 바인딩`을 할 때도 사용이 가능하다. `if let` 과의 차이점은 `guard let` 을 통해서 바인딩 된 상수 값을 `guard` 구문 실행 코드 아래부터 **함수 내부의 지역상수 처럼 사용이 가능**하다. 

또한 위에 빵을 사먹는 예시처럼 `,` 를 통해서 추가로 조건을 나열 할 수 있고 이때 조건은 `Boolean` 타입 값이어야 한다. `,`는 AND 논리연산자와 같은 결과이고, 앞에서 바인딩 된 상수의 값에 조건을 지정할 수도 있다.  


## if VS guard

### 에러처리 및 코드의 가독성 측면
> Using a guard statement for requirements improves the readability of your code, compared to doing the same check with an if statement. It lets you write the code that’s typically executed without wrapping it in an else block, and it lets you keep the code that handles a violated requirement next to the requirement.

Apple 공식 문서에 의하면 **`if` 보다 `guard` 를 사용하면 요구사항을 처리하는 코드와 에러를 처리하는 코드가 분리가 되어서 읽기가 더 수월**하다고 한다. `guard` 같은 경우에는 항상 `else` 문을 동반하며 `else` 문 안에는 에러 처리를 하는 경우가 많다. 즉, **어떤 값이 의도한 것처럼 기능하길 원하도록 표현할 때 사용**하며 만약 의도한 경우가 아니라면 실패하는 느낌을 준다. 따라서 `guard` 는 어떻게 보면 assert인데 crush를 내지 않고 break, return을 사용하거나 error handling을 통해 탈출하는 약한 assert 라고 생각할 수도 있을 것 같다.

반면에 if 의 경우에는 모든 경우에 대해서 조건을 확인하지 않을 수도 있고, 원하는 조건만 확인할 수도 있다. 따라서 맞고 틀림의 느낌이 아닌 A 라는 경우와 B 라는 경우를 단지 나눠서 처리하는 기능이라고 생각한다. 그렇기 때문에 앞으로의 코드 작성에 있어서의 if와 guard의 사용해야 할 흐름을 한 번 쯤은 생각해보고 코드를 작성하는 것이 좋을 것 같다. 

### 코드의 depth 측면
먼저 해당 코드를 리뷰해주고 조언을 해준 **도미닉**에게 고마움을 전한다🙏
```swift
private func transformToPostfix(_ infix: [String]) throws -> [String] {
        var postfix = [String]()
        for element in infix {
            if binaryOperator.contains(element) {
                if !postfixStack.isEmpty {
                    if let top = postfixStack.peek() {
                        var stackTopOperatorType = try getOperatorType(of: top)
                        let currentOperatorType = try getOperatorType(of: element)
                        while(!postfixStack.isEmpty && precedence(stackTopOperatorType) 
                              >= precedence(currentOperatorType)) {
                            guard let top = postfixStack.pop() else {
                                throw CalculatorError.stackIsEmpty
                            }
                            postfix.append(top)
                            if let topAfterPop = postfixStack.peek() {
                              stackTopOperatorType = try getOperatorType(of: topAfterPop)
                            }
                        }
                    }
                }
                postfixStack.push(element)
            }
            else {
		postfix.append(element)
            }
        }
        while !postfixStack.isEmpty {
            if let top = postfixStack.pop() {
                postfix.append(top)
            }
        }
        return postfix
}
```
위와 같은 함수가 있다고 하자. 이 함수는 전위 표기로 되어있는 `[String]` 타입을 매개변수로 받아서 후위 표기로 바꿔서 `[String]` 타입으로 반환해주는  함수이다. 
이 함수에 대해서 **도미닉**으로부터 리뷰를 받은 부분은 바로 code depth가 너무 깊다는 것이다. 
code의 depth를 길게 하지 않는 것이 가독성 측면에서 좋다는 것을 알면서도 `stack`의 값을 확인하는 과정에서 optional인지를 확인해 주어야 했기에 `if let`을 많이 쓰다 보니까 코드의 depth가 엄청 깊어지게 되었다. 

이때까지만 해도 `guard`는 보다 쉽게 에러를 처리하기 위해서 사용한다는 생각을 가지고 있어서 `if let` 을 사용했었다. 그래서 **도미닉**에게 방법을 물어보게 되었고 **도미닉**은 depth를 줄이는 방법으로는 `if`를 `guard`로 바꿔서 작성하거나 **depth가 깊어지는 부분을 따로 함수로 떼어내서 작성**하는 것이 하나의 방법이라고 힌트를 주었다. 
```swift
 private func transformToPostfix(_ infix: [String]) throws -> [String] {
        var postfix = [String]()
        for element in infix {
            guard binaryOperators.contains(element) else {
                postfix.append(element)
                continue
            }
            guard !postfixStack.isEmpty else {
                postfixStack.push(element)
                continue
            }
            guard let top = postfixStack.peek() else {
                throw CalculatorError.stackIsEmpty
            }
            var stackTopOperatorType = try getOperatorType(of: top)
            let currentOperatorType = try getOperatorType(of: element)
            while(!postfixStack.isEmpty && precedence(stackTopOperatorType) 
                  >= precedence(currentOperatorType)) {
                guard let top = postfixStack.pop() else {
                    throw CalculatorError.stackIsEmpty
                }
                postfix.append(top)
                if let topAfterPop = postfixStack.peek() {
                    stackTopOperatorType = try getOperatorType(of: topAfterPop)
                }
            }
            postfixStack.push(element)
        }
        
        for _ in 0..<postfixStack.size {
            if let top = postfixStack.pop() {
                postfix.append(top)
            }
        }
        
        return postfix
    }
```
그래서 다음과 같이`if`를 `guard`로 변환하니까 확실히 depth가 줄어들고 코드를 읽기가 편해지는 것을 느낄 수 있었다. 

이로써 if와 guard를 사용해야 할때 생각해 볼 기준이 하나 더 생겼다. 그 전에는 요구사항의 처리하는 코드와 에러를 처리하는 코드를 분리할 때에 guard를 사용하고 if 의 경우에는 모든 경우에 대해서 조건을 확인하지 않을 수도 있고, 원하는 조건만 확인할 수도 있다. 따라서 맞고 틀림의 느낌이 아닌 A 라는 경우와 B 라는 경우를 단지 나눠서 처리하는 기능이라고 생각했지만 이제는 또 하나의 기준인 **code의 depth의 측면**에서도 고려해서 사용해야 할 것 같다.

## 결론
지금까지 `if`, `guard` 에 대해서 길게 말했지만 내가 내린 결론은 결국 다음과 같다. 

1. 어떤 값이 의도한 것처럼 작동하기 원할 때, 만약 그 값이 아니라면 error가 발생한다는 느낌이라면 `guard`을 사용하고 error 처리를 해주자.
2. 옵셔널 바인딩을 하는 경우에는 code의 depth가 깊어질 것 같은 경우 `guard`를 사용해서 code의 depth를 줄여보자.
3. 간단한 처리라면 `if`를, 한 개 이상의 변수 연산이 필요한 경우에는 `guard`를 사용하자.

물론 `if` 를 사용하나 `guard` 를 사용하나 둘 다 코드를 작성하는데에는 큰 문제가 없다.
그러나 코드의 가독성 측면에서는 `guard` 를 사용하는 것이 좀 더 바람직하다고 생각한다. 
"좋은 코드는 다른 사람들이 읽기 쉬워야한다" 말처럼 `guard` 를 사용함으로서 가독성을 높일 수 있다면 `if` 보다는 `guard` 를 사용하는 것이 좋은 것 같다. 
앞으로도 `if` 와 `guard`를 사용하는데 고민할 만한 기준들이 생긴다면 추가해 볼 예정이다. 


## 참고
1. [Apple 공식문서 : Language Guide - Control Flow](https://docs.swift.org/swift-book/LanguageGuide/ControlFlow.html)
2. [Swift: When to use guard vs if](https://www.natashatherobot.com/swift-when-to-use-guard-vs-if/)

