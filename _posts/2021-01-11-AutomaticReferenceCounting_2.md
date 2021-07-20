---
title:  "[Swift] ARC(Automatic Reference Counting) - 2"
excerpt: "closure로 발생하는 retain cycle을 해결해보자."

categories:
  - Swift
tags:
  - [ARC, retain cycle, capture list]
toc: true
toc_sticky: true
last_modified_at: 2021.01.11
---

## 들어가며
[지난 포스팅]({{site.url}}{{site.baseurl}}/swift/AutomaticReferenceCounting_1)에서는 Swift에서 두 개의 class 객체에서 발생할 수 있는 strong reference cycle에 대해서 알아보았었다. 그리고 이를 해결하는 4가지 방법에 대해서 포스팅했었다.

그러나 <br>
strong reference cycle은 class 객체의 프로퍼티에 closure를 할당하고 이때 closure의 body가 객체를 capture 한다면 발생할 수도 있다. 여기서 말하는 capture는 `self.someProperty` 나 `self.someMethod()`와 같이 **class의 객체의 프로퍼티나 함수에 접근을 할 때 발생**하게 된다. 이러한 접근은 closure가 `self`를 capture 하도록 하며 strong reference cycle을 유발한다. 

이러한 strong reference cycle은 closure, class들이 **reference type**이기 때문에 발생한다. 따라서 우리가 프로퍼티에 closure를 할당하는 것은, 해당 closure에 **reference**를 할당하는 것이다. <br>
본질적으로 지난 포스팅에서 다룬 것과 같이 **두 개의 strong references가 서로를 존속**시키는 것인데, 다른 점이라면 class 객체 두 개가 아닌 **class 객체와 closure로 인해 발생**한다는 점이다. 

## clousure로 인해 발생하는 retain cycle 예시
이에 Swift에서는 `closure capture list` 라고 알려진 해결책을 제공한다. <br>
해결책에 대해서 알아보기 전에 먼저 class 객체와 closure로 인해 발생하는 strong reference cycle의 예시를 보도록 하자.

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
    if let text = self.text {
        return "<\\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}
```

HTMLElement 클래스는 `name` 프로퍼티와 `text` 프로퍼티를 가지고 있다. <br>
`name` 프로퍼티는 "h1" : heading element, "p" : paragraph element, "br" : line break element 와 같은 element의 이름을 나타낸다. <br> 
`text` 프로퍼티는 Optional 타입으로 HTML elmenent 안에서 rendering 될 text를 나타낸다. 

또한 `asHTML`이라는 **lazy 프로퍼티**를 가지고 있다. 
`asHTML` 프로퍼티는 name과 text를 HTML string 조각으로 결합하는 **closure를 참조**한다. 
`asHTML` 프로퍼티는 **() → String**, 즉 매개변수를 사용하지 않고 String을 반환하는 타입이다. 

asHTML 프로퍼티는 기본적으로 HTMLtag를 의미하는 string을 반환하는 closure를 할당받는다. 
만약 paragraph element라면 `"<p>some text<\p>"` 나 `"<p/>"` 둘 중에 text의 값의 유무에 따라서 값을 가지게 된다. 

여기서 asHTML 프로퍼티는 instance method처럼 선언되고 사용된다. 그러나 asHTML은 실제로는 instance method가 아닌 closure이기 때문에 우리는 아래의 코드와 같이 특별한 HTML element를 랜더링하려고 할 때 custom closure를 사용해서 asHTML 프로퍼티의 기본 값을 변경할 수 있다. 

```swift
let heading = HTMLElement(name: "h1")
let defaultText = "some default text"
heading.asHTML = {
    return "<\(heading.name)>\(heading.text ?? defaultText)</\(heading.name)>"
}
print(heading.asHTML())
// Prints "<h1>some default text</h1>"
```

위의 코드는 만약 text가 nil이라면 empty HTML대신 defaultText로 선언된 값이 출력되는 것을 확인할 수 있다. 

HTLMElement class는 initailizer와 deinitializer를 가지고 있다. 

```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```

따라서 위의 코드는 실제로 해당 클래스를 사용한 것이다. 이때 HTMLElement의 타입이 optional인데 이는 나중에 nil로 만들어서 strong reference cycle의 존재를 확인하기 위해서이다. 

![](https://images.velog.io/images/minni/post/404a6760-f6fa-4425-a07e-a20d5ba47736/image.png) 

코드의 결과 위의 그림과 같은 strong reference cycle이 HTMLElment 객체와 asHTML의 기본값을 사용하는 closure 사이에 발생하게 된다. 

```swift
paragraph = nil
``` 

여기서 paragraph에 nil을 할당해서 HTMLElement와의 strong reference를 끊는다고 해도 여전히 HTMLElement 객체와 closure사이에는 strong reference가 남아있게 되고 그 결과 deinitailzer에 의한 문구가 출력되지 않는다. 

## closure에서 retain cycle 해결 방법
그렇다면 closure에서 retain cycle(=strong reference cycle)를 해결하는 방법은 다음과 같다. 

### Capture List는 무엇일까?
closure expression은 기본적으로 주변 범위에서 상수 및 변수를 해당 값에 대한 strong reference로 capture하게 된다. 

이때 capture list안에 항목들은 [] 대괄호 안에 ,로 구분되어 나열되며 parameter나 return 값에 대한 명시 전에 작성되어야한다. 그리고 parameter나 return 값이 없는 경우에도 **in keyword**를 붙여주어야한다. 

capture list안에 있는 항목들은 closure가 생성되는 시점에 initialize된다. <br>
따라서 capture list안에 있는 각각의 항목들은 주변 범위에서 이름이 같은 상수 또는 변수의 값이 상수에 저장된다. 말이 헷갈리는데 즉, 상수로 저장되는데 값은 주변 범위에서 이름이 같은 상수나 변수의 값이라는 것이다. 

다음의 예시를 확인해보자

```swift
var a = 0
var b = 0
let closure = { [a] in
    print(a, b)
}

a = 10
b = 10
closure()
// Prints "0 10"
```

이때 a는 capture list안에 포함되어있고 b는 그렇지 않다. <br>
이는 a의 outer scope에서 변경되는 값들은 inner scope(여기서는 closure 내부를 의미)에 영향을 끼치지 않는다는 것이다. 이와 반대로 내부에서 변경되는 값도 외부에 영향을 미치지 않는다. 

반대로 b라는 변수는 outer scope에 있는 b 하나뿐이므로 closure 내부 또는 외부에서의 발생하는 변화는 두 곳 모두에서 확인할 수 있다. 

그러나 이러한 차이는 capture한 값의 타입이 reference 타입이라면 발생하지 않는다. <br>
아래의 코드를 살펴보자.

```swift
class SimpleClass {
    var value: Int = 0
}
var x = SimpleClass()
var y = SimpleClass()
let closure = { [x] in
    print(x.value, y.value)
}

x.value = 10
y.value = 10
closure()
// Prints "10 10"
```

만약 closure안에 들어가 있는 표현 값의 타입이 class라면, 우리는 capture list 안에 weak, unowned와 같은 keyword를 사용해서 weak, unowned reference를 하도록 할 수 있다. 

```swift
myFunction { print(self.title) }                    // implicit strong capture
myFunction { [self] in print(self.title) }          // explicit strong capture
myFunction { [weak self] in print(self!.title) }    // weak capture
myFunction { [unowned self] in print(self.title) }  // unowned capture
```

또한 아래의 코드와 같이 임의적인 표현에 name 붙여서 binding 할 수도 있다.<br>
이때 이 expression은 closure가 생성되고 value가 지정된 강도로 capture 될 때 평가된다.  

```swift
// Weak capture of "self.parent" as "parent"
myFunction { [weak parent = self.parent] in print(parent!.title) }
```

### Defining a Capture List
**capture list안에 있는 item**들은 class instance(e.g. self)에 대한 참조나, 일부 값으로 초기화 한 변수(e.g. delegate = self.delegate)는 **weak, unowned keyword를 쌍으로 하여 구성**할 수 있다.

다음과 같이 작성할 수 있다. 

```swift
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate]
    (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```

이때 pairing은 [] 괄호 안에 ,로 구분되게 작성하며 capture list는 **closure의 parameter list 전에 작성하거나 return 타입을 반환한다면 해당 반환타입을 작성하기 전**에 작성해주어야 한다. 

```swift
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate] in
    // closure body goes here
}
```

만약 closure의 parameter list나 return type을 context로 부터 유추할 수 있어서 작성하지 않아도 된다면 **capture list는 closure의 시작 부분에 작성하고 뒤에 in keyword를 붙여준다.**

### Weak and Unowned References
그렇다면 capture list안에서는 어떠한 상황에 capture를 weak 혹은 unowned를 사용해야할까?

`unowned` : closure와 capture된 객체가 항상 서로를 참조하고 항상 동시에 할당 해제되는 경우에 `unowned` 를 사용하면 된다.  <br>
`weak` : **미래에 capture된 참조 값이 nil이 될 수 있다면** `weak` 를 사용하면 된다. `weak` 는 항상 **optional type**이며 참조하는 **객체가 할당 해제되는 시점에서 자동적으로 nil**이 된다. 따라서 closure의 body안에서 참조하는 객체들의 존재를 검사할 수 있게 해준다. 

> capture된 reference가 절대로 nil이 될 가능성이 없다면 
weak 보다는 unowned를 사용해서 capture 해야한다. 

따라서 위에서 예시로 들었던 HTMLElement의 경우에는 unowned reference를 사용하여 retain cycle을 해결하면 된다. 

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }

}
```

closure의 capture list가 생긴 부분을 제외하고는 다른 부분은 다 동일하다. <br>
이때 unowned self라고 되어있는데, 이는 strong reference가 아닌 unowned로 self를 capture하겠다는 의미이다. 

```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```

예전과 동일하게 HTMLElement 객체를 생성하고 사용할 수 있다. 

![](https://images.velog.io/images/minni/post/977455c2-656c-4101-9c7f-796b9d5b8262/image.png) 

그러나 참조의 관계는 위의 그림과 같이 바뀌었다. 
이번에는 closure가 HTMLElement에 대해서 unowned 참조를 하고 있으며 그렇기 때문에 paragraph에 nil을 할당하여 strong reference를 끊으면 이제는 할당 해제되는 것을 확인할 수 있다.

```swift
paragraph = nil
// Prints "p is being deinitialized"
```

따라서 deinitializer에 의해서 할당 해제 문구가 출력되는 것을 확인할 수 있다. 

## 참고
[ARC Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)<br>
[Expressions Swift Programming Language](https://docs.swift.org/swift-book/ReferenceManual/Expressions.html#ID544)