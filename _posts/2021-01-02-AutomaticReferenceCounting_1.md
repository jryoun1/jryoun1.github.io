---
title:  "[Swift] ARC(Automatic Reference Counting) - 1"
excerpt: "Swift에서 ARC가 어떻게 작동하는지 알아보자."

categories:
  - Swift
tags:
  - [ARC, retain cycle]
toc: true
toc_sticky: true
last_modified_at: 2021.01.02
---

## 들어가며 
Swift에서 **ARC(Automatic Reference Counting)** 는 앱의 메모리 사용을 추적하고 관리한다. 
즉, class객체에 의해서 사용되는 memory들을 더 이상 class객체가 사용되지 않을때 자동으로 수거한다.
따라서 C언어처럼 우리가 직접 메모리를 할당하고, 수거하지 않아도 된다.
그러나 몇몇의 case들은 직접 신경을 써주어야한다. 

Reference counting은 [struct와 class를 비교했던 글]({{site.url}}{{site.baseurl}}/wwdc/UnderstandingSwiftPerformance_2)을 보면 class의 instance에게만 적용된다는 것을 알 수 있다. 
단어 그대로 reference counting은 참조 타입(reference type)에게만 해당하므로 구조체(struct)와 열거형(enum)과 같은 값 타입(value type)들은 참조 값을 저장하거나 전달하지 않으므로 이에 해당되지 않는다.


## ARC는 어떻게 작동할까?
Swift 공식문서에 따르면 ARC는 다음과 같이 작동한다고 한다. 

클래스의 새로운 객체를 생성할 때 마다, ARC는 해당 객체에 대한 정보를 저장하기 위해서 메모리를 할당하게 된다. 이 메모리에는 해당 객체의 type 정보와 객체의 값을 가지게 된다. <br>
만약 객체가 더 이상 쓸모 없어지면, 즉 참조를 당하지 않는다면 ARC는 해당 메모리를 해제하여 다른 곳에 사용될 수 있도록 한다. class 객체가 사용되지 않는다면 메모리의 일부를 차지하지 않는다는 것을 보장해주므로 메모리의 관리에 있어서 효율적이다. <br>

**하지만** <br>
ARC가 사용되고 있는 객체를 메모리에서 할당 해제하게 된다면, 더 이상 해당 객체의 프로퍼티나 메서드로의 접근은 불가능해지며 접근을 하려고 한다면 앱에 충돌이 발생하여 중단되게 된다. 

따라서 위와 같은 일이 발생하지 않도록 ARC는 **class 객체를 참조하는 속성, 상수 및 변수들의 수를 추적하고 관리한다**. 그리고 하나라도 객체를 참조하고 있다면 ARC는 해당 객체는 사용 중이라고 판단하여 메모리에서 할당 해제를 시키지 않는다. 

위와 같이 ARC가 계속해서 추적하고 관리하게 하기 위해서는 class 객체의 속성, 상수 및 변수들은 해당 객체와 `strong reference` 의 관계가 되어야한다. 이러한 참조는 `strong reference` 라고 하는데 그 이유는 **`strong reference` 가 남아있는한 해당 인스턴스에 대한 메모리에서의 할당 해제는 일어나지 않기 때문**이다.  

### ARC 작동 예시
아래의 코드를 통해서 ARC가 어떻게 작동하는지 확인해보자. 
Person이라는 class가 있고, 이 class는 name이라는 stored property를 가지고 있다.
initializer는 name에 값을 설정해주고 print를 하고, deinitializer도 print를 한다. 

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}          
```

따라서 Person class의 객체가 생성되면 init이 호출되고, 할당 해제가 되면 deinit이 호출되게 된다.

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?
```

Person 타입을 가질 수도 있고, 그렇지 않을 수도 있는 3개의 변수를 정의했다.

```swift
reference1 = Person(name: "John Appleseed")
// (ARC 참조 카운트 +1)
// "John Appleseed is being initialized"
```

그리고 reference1에는 Person class 객체를 생성해 할당해주었다.
이때 initializer에 의해서 해당 문장이 출력되는 것을 확인해 볼 수 있다. <br>
이 말은 즉, Person 객체가 reference1 변수에 할당되어 있고 `strong reference` 가 하나라도 있기 때문에 ARC은 memory에서 Person을 할당해제 하지 않게 된다. 

만약 여기서 똑같은 Person 객체를 남은 두 개의 변수에 할당시켜준다면 어떻게 될까?

```swift
reference2 = reference1
// (ARC 참조 카운트 +1)
reference3 = reference1
// (ARC 참조 카운트 +1) 
```

이제 그럼 두 개의 `strong reference` 가 추가로 생기게된다. <br>
따라서 총 3개의 `strong reference` 가 Person 객체 존재하게 된다. 

여기서 2개의 변수에 nil을 할당해주어서 `strong reference` 를 끊어주게 되어도, 하나의 `strong reference`가 남아있기 때문에 ARC는 Person 객체를 memory에서 할당 해제 하지 않는다. 

```swift
reference1 = nil
// (ARC 참조 카운트 -1)
reference2 = nil
// (ARC 참조 카운트 -1)
```

### 그렇다면 언제 ARC는 Person 객체를 memory에서 할당 해제할까?

```swift
reference3 = nil
// (ARC 참조 카운트 -1)
// "John Appleseed is being deinitialized"
```

바로 reference3에 nil을 할당해서 `strong reference` 이 깨지게 되는 시점, 즉 더 이상 Person 객체를 사용하지 않는다는 것이 확실한 ARC 참조 카운트가 0이 되면 그때 deinit이 발생하게 된다. 
따라서 deinit시에 출력되도록 한 문구가 출력되는 것을 확인할 수 있다. 

## 순환 참조 (Strong referecence Cycle)
### 순환 참조란 무엇일까? 
위의 예시에서는 Person class 객체가 생성될 때 마다 참조되는 회수를 카운트 할 수 있었고, 더 이상 사용하지 않아서 메모리에서 할당 해제 될 때도 이를 추적하고 관리할 수 있었다. 

**그러나** <br>
코드를 잘못 작성하게되면 class 객체의 **참조 카운트가 절대로 0이 될 수 없는 상황**이 될 수가 있다. 이 문제는 두 개의 class 객체가 서로 `strong reference` 를 유지하여 각 객체가 다른 객체를 참조하는, 즉 활성 상태로 유지하는 경우에 발생할 수 있다. 이를 **순환 참조(`strong reference cycle`)** 이라고 한다. 

### 순환 참조 예시

아래의 예시 코드를 확인해보자.

```swift
class Person {
    let name: String
    init(name: String) {
        self. name = name
    }
    var apartment: Apartment?
    deinit {
        print("\(name) is being deinitialized")
    }

class Apartment {
    let unit: String
    init(unit: String) {
        self. unit = unit
    }
    var tenant: Person?
    deinit {
        print("Apartment \(unit) is being deinitialized")
    }
```

Person 클래스와 Apartment 클래스가 있다. <br>
Person 클래스는 name, apartment라는 프로퍼티가 있는데 
아파트가 없는 사람도 있을 수 있기 때문에 apartment는 `optional`이다.

Apartment 클래스는 unit과 tenant라는 프로퍼티가 있는데 
비어있는 아파트도 있을 수 있기 때문에 tenant는 `optional`이다. 

두 클래스 모두 deinitializer가 있고 메모리에서 할당 해제되면 해당 문구를 출력하게 된다.

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A: = Apartment(unit: "4A")
```

이제 Person 인스턴스 변수인 john과 Apartment 인스턴스 변수인 unit4A를 생성하고 값을 할당해준다. 

![](https://images.velog.io/images/minni/post/0d1c7dd8-963d-4dfb-8ef5-c95e9ba99093/image.png) 

이때 john과 unit4A 변수는 Person 인스턴스와 Apartment 인스턴스에 위의 그림과 같이 `strong reference` 를 가지게 된다. 

```swift
john!.apartment = unit4A
unit4A!.tenant = john
```

그리고 john은 unit4A를 소유하게 되고 unit4A는 세입자로 john이 오게 됩니다. 

위의 코드처럼 두 인스턴스를 연결하게 되면 인스턴스 간에 `순환 참조(strong reference cycle)` 가 생성되게 된다. 

![](https://images.velog.io/images/minni/post/2c6d38ce-d678-4020-8a48-a94aeccdb881/image.png)

위의 그림 처럼 이제 Person 인스턴스와 Apartment 인스턴스 사이에도 `strong reference` 가 생성되는 것을 확인할 수 있다. 즉, Person 인스턴스는 `strong reference` 로 Apartment 인스턴스를 가지며, Apartment 인스턴스도 `strong reference` 로 Person 인스턴스를 가지게 됩니다. 

```swift
john = nil
unit4A = nil
```

따라서 john과 unit4A 변수가 가지는 `strong reference` 를 nil을 할당함으로써 해제시켜도 참조 카운트의 값은 0으로 떨어지지 않게 되고 따라서 ARC에 의해서 메모리에서 할당 해제되지 않고, 할당 해제가 발생하지 않았기 때문에 deinitializer는 당연히 호출되지 않게 된다.  

![](https://images.velog.io/images/minni/post/22fb3f34-f6fd-4abd-8c10-fbbbe6612242/image.png) 

위의 그림과 같이 Person 인스턴스와 Apartment 인스턴스 사이에는 여전히 `strong reference` 가 남아있으며, 이들 인스턴스에 접근할 수 있는 방법은 없고 따라서 두 인스턴스 사이에 `strong reference` 는 깨지지 못해 **메모리의 누수(memory leak)** 가 발생하게 되는 것이다.

## 순환 참조 해결 방법
### 순환 참조는 꼭 해결해야만 할까?

순환 참조가 많고 해결하지 않는다면 memory 상에는 결국 사용되지 않지만 자리를 차지하고 있는 인스턴스들이 많아지게 되고 이는 나중에 진짜 memory에 할당이 필요한 인스턴스들이 자리가 없어서 할당되지 못하고 메모리 부족과 같은 문제를 야기할 수 있다. 

### 순환 참조를 어떻게 하면 해결할 수 있을까?
Swift에서는 순환 참조를 해결할 수 있는 방법으로 두 가지 방법을 제시한다. 

1. **weak reference**
2. **unowned reference**

위의 두 가지 참조 방법은 `strong reference`를 하지 않고 서로를 참조 할 수 있는 방법이라고 한다.

#### 두 가지 방법의 차이점은?
`weak reference`는 optional 타입이며, `unowned reference`는 optional 타입이 아니다. 

#### 언제 어떠한 것을 사용해야할까?
`weak reference` 의 경우에는 다른 인스턴스의 수명이 짧을 때 사용하며 
`unowned reference` 의 경우에는 다른 인스턴스의 수명이 같거나 긴 경우에 사용한다. 


### 1️⃣ 약한 참조(weak reference)

`weak reference` 는 참조하는 인스턴스를 강하게 참조하지 않는다.
따라서 `weak reference` 가 **참조하는 동안에는 해당 인스턴스는 메모리로부터 할당 해제가 가능**하다.<br>

ARC는 인스턴스가 할당 해제되면 `weak reference` 를 자동으로 nil로 설정하게 되고, `weak reference` 의 경우에는 runtime에 nil로 변경할 수 있어야하므로 **항상 상수(let)이 아닌 변수(var)로 선언되어야만** 한다. 

```swift
class Person {
    let name: String
    init(name: String) {
        self. name = name
    }
    var apartment: Apartment?
    deinit {
        print("\(name) is being deinitialized")
    }

class Apartment {
    let unit: String
    init(unit: String) {
        self. unit = unit
    }
    weak var tenant: Person?
    deinit {
        print("Apartment \(unit) is being deinitialized")
    }
```

다시 위에서 보았던 예시를 통해 알아보자. <br>
이때 달라진 점은 단 하나 Apartment 클래스의 **tenant가 `weak reference` 타입으로 선언**되어 있다는 점이다.

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```

위의 코드는 예전과 동일하게 Person 인스턴스 변수인 john과 Apartment 인스턴스 변수인 unit4A를 생성하고 값을 할당해주고 unit4A는 john의 apartment로, john을 unit4A의 세입자로 설정해주었다.

![](https://images.velog.io/images/minni/post/77326d51-f4ac-4566-876e-eafb16fdd996/image.png) 

그러나 이전과 다른 점은 Person 인스턴스는 여전히 Apartment 인스턴스에게 `strong reference` 하는 반면에 **Apartment 인스턴스는 Person 인스턴스에게 `weak reference` 를 하게 된다.** 
이때 john이 하고 있는 `strong reference` 를 끊어주면 Person 인스턴스에 대한 강한 참조는 더 이상 존재하지 않게 된다. 

```swift
john = nil
// "John Appleseed is being deinitialized"
```

john에 nil을 할당해주므로 Person 인스턴스가 메모리에서 할당 해제되고, deinitializer에 의해서 구문이 출력되게 된다. 

![](https://images.velog.io/images/minni/post/176d1d68-ae44-447c-832d-aa52180c9338/image.png)

위의 그림에서 처럼 Person 인스턴스는 더 이상 `strong reference` 가 없기 때문에 할당 해제 되고, Person 인스턴스를 참조했던 **tenant 프로퍼티는 ARC에 의해서 nil로 설정되게** 된다. <br>
이제 남은 `strong reference`는 unit4A 변수의 Apartment 인스턴스에 대한 강한 참조 뿐이다. 

```swift
unit4A = nil
// "Apartment 4A is being deinitialized"
```

unit4A에 nil을 할당해주어서 `strong reference`를 끊어주면 더 이상 Apartment 인스턴스에 남은 강한 참조는 없으므로 메모리에서 할당 해제된다. 

![](https://images.velog.io/images/minni/post/36560a14-598a-4c85-8a24-0f9c3aab30a9/image.png) 

`weak reference`를 사용해서 strong reference cycle 문제를 해결하는 것을 확인할 수 있었다. 

Swift 공식 문서에 의하면 `weak reference` 의 경우에는 **다른 인스턴스의 수명이 짧을 때 사용**하며, **반드시 변수로 선언**된다고 나와있는데, 이번 예시를 통해서 Person 인스턴스(다른 인스턴스)가 할당 해제되면(수명이 짧음) ARC에 의해서 Apartment 프로퍼티인 tenant가 nil이 되는 것을 확인할 수 있었다. (옵셔널이면서 변수로 선언되는 이유)


### 2️⃣ 미소유 참조(unowned reference)
다음은 strong reference cycle을 해결할 수 있는 또 하나의 방법인 `unowned reference` 이다. 

`unowned reference` 는 `weak reference` 와 같이 참조하는 인스턴스를 강하게 참조하지 않는다. <br>
그러나 weak reference와 다르게 다른 인스턴스의 수명이 같거나 긴 경우에 사용하게 된다. 

`unowned reference` 의 경우에는 항상 값이 있다고 생각한다.<br>
따라서 선언을 할 때 **optional 타입이 아니고**, ARC는 `unowned reference` 를 **nil로 설정하지 않는다**. <br>

즉, `unowned reference` 는 **절대로 할당 해제가 되지 않을 인스턴스를 참조하는 경우에만 사용**해야하며, 만약 `unowned reference` 로 선언된 **할당이 해제된 인스턴스를 참조하려고 한다면 runtime error가 발생**하게 될 것이다. 

예시를 통해서 알아보자.

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { 
        print("\(name) is being deinitialized") 
    }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { 
        print("Card #\(number) is being deinitialized") 
    }
}
```

이번 예시는 Apartment와 Person 예제와는 다른 관계를 가지고 있는 Customer와 CreditCard예시이다. 

고객은 credit card를 가지고 있거나 그렇지 않을 수는 있지만 credit card는 항상 고객과 연결이 되어있어야만 한다. 그래서 Customer 클래스의 card 프로퍼티는 optional 타입으로 선언되어 있다. <br>

반면에 CreditCard 클래스의 customer 프로퍼티는 optional 타입이 아닌 `unowned reference` 타입으로 선언이 되어있다. 게다가 CreditCard 인스턴스는 number와, customer의 정보를 init할 때 넘겨주어야만 생성이 될 수 있고, 이는 CreditCard 인스턴스가 생성될 때 항상 customer 인스턴스를 참조하게 되고 순환참조를 피하기 위해서 `unowned reference` 참조를 사용한다. 

```swift
var john: Customer?
john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
```

이제 john 변수에 Customer 인스턴스를 만들어 인스턴스를 초기화시켜준다.
그리구 john의 card로 CreditCard 인스턴스를 생성하면서 초기화시켜서 할당시켜준다. 

![](https://images.velog.io/images/minni/post/6ffcf37c-3822-4ee7-9ae5-5214b407673e/image.png) 

해당 관계는 위의 그림과 같이 표현된다. <br>
Customer 인스턴스는 CreditCard 인스턴스를 `strong reference` 하게 되고, CreditCard 인스턴스는 Customer 인스턴스를 `unowned reference` 하게 된다. 

![](https://images.velog.io/images/minni/post/8a7d760f-cb0b-41c6-91da-c6b97155cd4f/image.png)

여기서 john 변수의 Customer 인스턴스로의 `strong reference`를 끊어주게 되면, 더 이상 Customer 인스턴스는 `strong reference`가 없게 된다. 

```swift
john = nil
// "John Appleseed is being deinitialized"
// "Card #1234567890123456 is being deinitialized"
```

Customer 인스턴스는 메모리에서 할당 해제되게 되고, 이후에 CreditCard 인스턴스도 strong reference가 없기 때문에 메모리에서 할당 해제된다. 따라서 deinitializer에 의해서 구문이 출력되는 것을 확인할 수 있다. 

`unowned reference` 는 다른 인스턴스의 수명이 같거나 수명이 긴 경우 사용한다고 위에서 언급했었는데 Customer의 card 프로퍼티의 경우에는 다른 인스턴스(Customer)의 수명과 같았기 때문에 `unowned reference` 로 선언이 되었다.  

### 3️⃣ Unowned Optional References
기존에 unowned reference와 weak reference의 차이 중 하나는 optional 타입이 될 수 있고 없음에 따라서
`weak reference` 는 nil이 될 수 있고 `unowned reference` 는 nil이 될 수 없었다. 

그러나 Swift5 이상부터는 **`unowned reference` 의 경우에도 optional 타입이 될 수 있다.** 
즉, `weak reference` 와 `unowned reference` 는 이제 같은 맥락에서 사용이 가능해졌다. 

#### unowned와 unowned optional 차이점은?
`weak reference`의 경우에는 이전과 동일하지만, `unowned optional reference`는 **이제 항상 값이 있는 유효한 객체를 참조하거나, nil로 설정이 되어있는 경우에만 사용**을 하면 될 것이다. 

글로만 보면 잘 모르겠으니까 예시를 통해서 알아보자!

```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}
```

이번 예시에서는 학교에서 Department(학과)와 Course(강좌)의 관계이다. 

Department(학과)는 각각의 Department(학과)에서 제시하는 course와 `strong reference` 를 유지한다. 
Course(강좌)의 경우에는 2개의 `unowned reference` 를 가지는데 하나는 department(학과)이며, 다른 하나는 학생이 다음으로 수강해야 할 nextCourse(다음강좌)로서 어떠한 객체도 소유하지 않는다. 

모든 course(강좌)는 department(학과)와 연결이 되어야하므로 **department 프로퍼티는 non-optional 타입**이지만 **어떠한 강좌(course)들은 다음으로 수강해야할 강좌가 없을 수도 있기 때문에 optional 타입**을 가지게 된다. 

```swift
let department = Department(name: "Computer Science Engineering")

let intro = Course(name: "Welcome to C language", in: department)
let intermediate = Course(name: "Welcome to C++ language", in: department)
let advanced = Course(name: "Welcome to Swift", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]
```

이제 위의 Department와 Course를 사용한 예시를 보자.<br>

department라는 상수는 Department 인스턴스로 컴퓨터공학과라는 이름으로 생성되며 초기화되어 할당된다. <br>
intro라는 상수는 Course 인스턴스로 C언어라는 이름으로 학과는 department(컴퓨터공학)으로 초기화되며 할당된다. 
intermediate라는 상수도 Course 인스턴스로 C++언어라는 이름으로 학과는 위와 동일하게 초기화되며 할당된다. <br>
advanced 상수도 Course 인스턴스로 Swift언어라는 이름으로 학과는 위와 동일하게 초기화되며 할당된다. <br>

이때 intro(C언어)의 다음 강의로 intermediate(C++) 설정해주고
intermediate(C++)의 다음 강의로는 advanced(Swift)를 설정해준다. 
마지막으로 department(컴퓨터공학과)의 강좌에는 intro(C), intermediate(C++), advanced(Swift)가 있다고 설정해준다. 

![](https://images.velog.io/images/minni/post/4f510fe4-ebd8-4404-b841-be2fa2688562/image.png) 

해당 코드들은 위의 그림과 같은 관계를 가지게 된다. <br>
intro와 intermediate의 경우에는 다음 강의가 있다고 설정을 해주었기에 
설정해 준 course와 `unowned optional reference`를 가지게 된다. 

`unowned optional reference` 는 nil이 될 수 있다는 점을 제외하고는 
ARC의 측면에서는 `unowned reference` 와 동일하게 동작하게 된다. 

따라서 `unowned reference`와 마찬가지로 `unowned optional reference` 는 nextCourse가 **항상 메모리에서 할당 해제된 것을 참조하지 않도록 관리**를 해야하며 위의 예시에서는 만약 department(컴퓨터공학과)에서 course(강좌)를 삭제하는 경우에 **삭제되는 강좌를 참조하는 관계를 모두 제거해 주어야한다.**

### 4️⃣ Unowned References and Implicitly Unwrapped Optional Properties
`weak reference`와 `unowed reference`를 통해서 대부분의 strong reference cycle을 해결할 수 있다. 

1. Person과 Apartment 예시에서는 **두 가지 프로퍼티 모두 옵셔널 타입으로 nil이 될 수 있는 경우**에 strong reference cycle이 발생할 수 있었고, 이를 `weak reference`를 사용함으로서 해결할 수 있었다.

2. Customer과 CreditCard 예시에서는 **하나의 프로퍼티는 옵셔널 타입으로 nil이 될 수 있고, 다른 하나는 non-opitonal 타입으로 nil이 될 수 없는 경우**에 strong reference cycle을 `unowned reference`를 통해서 해결할 수 있었다. 

3. **두 개의 프로퍼티가 항상 값을 가지고, 한 번 initialize된 이후에는 nil이 되지 않는 경우가 있다.** 
이 시나리오에서는 하나의 클래스에서 unowned 프로퍼티를 가지고, unowned를 가지지 않는 다른 클래스에서 암시적으로 벗겨진(implicitily unwrapped) optional 프로퍼티를 사용하여 해결할 수 있다. 

예시를 통해서 알아보자.

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

두 개의 클래스의 상호의존성을 보면, City의 initializer는 Country 인스턴스를 country 프로퍼티에 저장한다. 

City의 initializer는 Country의 initializer 내부에서 호출이 되게 되는데, 이때 Country의 initializer는 **새로운 Country 인스턴스가 완전히 초기화되기 전에 City initializer로 self를 매개변수로 넘겨주지 못하게 된다**. 

따라서 이러한 요구사항을 만족시켜주기 위해서는 Country 클래스 안의 capitalCity 프로퍼티를 `implicitly unwrapped optional` 타입으로 선언해야한다. 이는 capitalCity 프로퍼티는 default 값으로 nil을 가지게 되지만 다른 옵셔널들과는 다르게 **값에 접근할 때 언랩핑을 해줄 필요없이 접근이 가능**하다. 

여기서 capitalCity 프로퍼티가 **기본 값으로 nil을 가지기 때문에** 새로운 Country 인스턴스는 **name 프로퍼티가 설정되자마자 완전히 초기화 될 수 있고**, 이것은 name 프로퍼티가 설정되자 마자  Country initializer는 reference를 시작하고 암시적 self 프로퍼티를 전달할 수 있게 된다. 따라서 Country initializer는 self를 City initializer의 매개변수로 보낼 수 있다. 

즉, strong reference cycle 없이 Country와 City 인스턴스를 하나의 문장으로 생성할 수 있고 capitcalCity 프로퍼티는 옵셔널 값을 언랩핑하지 않고 바로 접근해서 사용이 가능하다.

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// "Canada's capital city is called Ottawa"
```

위의 코드를 보면 country 변수에 Country를 생성하면서 초기화해주는데 이 과정안에는 City도 생성되면서 초기화가 되게된다. 그리고 country의 capticalCity에 접근을 언랩핑 없이 하는 것을 확인할 수 있다. 

위의 예시에서 `implicitly unwrapped optional` 을 사용하는 것은 two-phase class initializer의 요구사항을 만족시켜주기 위해서 사용했으며, capitalCity 프로퍼티는 한 번 initialze가 끝나면 strong reference cycle을 만들지 않고 non-optional 값처럼 접근하고 사용할 수 있게된다. 

## 결론
class의 객체끼리에서 strong reference cycle을 해결하기 위한 방법으로 총 4가지가 있었다.

1. weak reference
2. unowned reference
3. unowned optional reference
4. unimplicitly unwrapped optional

여기서 `weak` 와 `unowned` 의 차이에 대해서 마지막으로 언급해 보려고 한다. 

Swift5 이전에는 차이가 optional 타입이냐 아니냐에 따른 nil값이 있고 없음 이었지만 unowned optional reference가 생기면서 이제 unowned도 nil 값을 가질 수 있게 되었다.

#### unowned와 weak의 차이점은?

먼저 `unowned` 는 `weak` 와 다르게 let으로 선언될 수 있다.
`weak` 의 경우에는 runtime동안에 ARC에 의해서 nil값으로 변경이 되기 때문에 변수로만 선언이 가능했다면 `unowned` 는 옵셔널이 가능한 타입과 그렇지 않은 타입으로 선언이 가능하다. 옵셔널이 불가능한 타입의 경우에는 let으로 선언이 가능하며, 옵셔널이 가능한 타입은 `weak` 타입과 마찬가지로 변수로만 선언이 가능하다. 

`unowned` 와 `weak` 는 사용되는 시점이 다르다. <br>
`unowend` 의 경우에는 다른 인스턴스의 생명주기가 같거나 더 긴 경우에 사용된다.<br>
`weak` 의 경우에는 다른 인스턴스의 생명주기가 짧은 경우에 사용된다. <br>
그렇다면 `unowned optional` 이 나오기 전에는 인스턴스의 생명주기가 짧은 경우에는 weak를 사용하였는데 이제는 `unowned optional` 을 사용하는 것이 유리하다. 

왜냐하면 `weak` 의 경우에는 ARC가 계속해서 인스턴스를 추적하다가 객체가 사라질 때 nil로 값을 변경하게되는데 이 과정은 오버헤드라고 할 수 있기 때문에 `unowned optional` 을 사용하면 이러한 오버헤드를 줄일 수 있다. 

다음 포스팅에서는 class 객체 사이에 발생하는 retain cycle 이외에도 closure와 class 객체 사이에 발생할 수 있는 retain cycle과 해결방법에 대해서 알아볼 예정이다.

[👉🏻 다음 포스팅으로 가려면 여기]({{site.url}}{{site.baseurl}}/swift/AutomaticReferenceCounting_2)

## 참조 

[apple document ARC](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)






