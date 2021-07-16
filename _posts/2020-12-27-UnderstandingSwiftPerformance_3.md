---
title:  "[WWDC]Understanding Swift Performance (Struct VS Class) - 3"
excerpt: "WWDC16에 나온 Swift에서 성능을 향상시키는 방법 중에서 구조체와 클래스에 대해서 알아보자."

categories:
  - WWDC
tags:
  - [Struct VS Class, Understanding Swift Performance, Struct, Class, dynamic dispatch, static dispatch]
last_modified_at: 2020.12.27
---

## 들어가며
지난 포스팅은 WWDC 2016 Understanding Swift Performance 영상에서 reference counting에 대해서 알아보았다. <br>

👉🏻 [지난포스팅이 궁금하신 분들은 여기를 눌러주세요!!]({{site.url}}{{site.baseurl}}/wwdc/UnderstandingSwiftPerformance_2)

이번 포스팅에서는 이제 performance의 마지막 측면인 method dispatch에 대해서 알아 볼 것이다. 

## method dispatch
Swift에서 runtime에 메소드를 호출하게 되면, Swift는 정확한 구현을 실행해야한다. 이러한 method dispatch에는 `static dispatch` 와 `dynamic dispatch` 가 있다. 

### static dispatch
![](https://images.velog.io/images/minni/post/cff6ef91-f0a3-48e0-bb5c-690c87720503/image.png) 

compile time에 어떤 구현이 실행하도록 결정이 된다면 이는 static dispatch라고 한다. 
그리고 실제로 runtime에는 구현으로 바로 이동할 수 있게 된다. 이것은 complier가 실제로 어떠한 구현이 실행될 것인지 알 수 있는 **가시성(visibility)**을 가지게 되기 때문에, inline과 같은 것을 포함하여 코드의 최적화를 가능하게 한다.

### dynamic dispatch
![](https://images.velog.io/images/minni/post/b3b507f3-898f-4af9-800b-9c8c91a14682/image.png) 

반대로 dynamic dispatch는 static dispatch와는 대조적이다. 
dynamic dispatch는 compile time에 어떤 구현을 실행할 것인지 결정을 할 수 없게 되며, 그 결과 runtime에 어떠한 구현을 실행할 것인지 **찾고(look up)**, 그리고 나서 **이동(jump to it)**하게 된다. 

물론 이러한 비용은 앞선 포스팅에서 다뤘던 heap allocation과 reference counting에서 발생하는 thread synchronization 비용보다는 차이가 크지 않다.

그러나 dynamic dispatch는 **compiler의 가시성(visibility)을 차단**하기 때문에, static dispatch에서는 compiler가 수행할 수 있었던 **최적화를 막게된다**. 


## inlining이란?
자, 그럼 위에서 언급한 최적화에서 Inlining이라는 것은 무엇일까? 
전 포스팅에서 다뤘던 Point 구조체 예시를 다시 한번 보자.
```swift
// Method Dispatch
// Struct (inlining)

struct Point {
  var x, y: Double
  func draw() { 
     // Point.draw implementation  
  }
}
func drawAPoint(_ param: Point) {
    param.draw()
}

let point = Point(x: 0, y: 0)
drawAPoint(point)
```
Point Struct는 x, y를 가지고 있고 draw()라는 메소드를 가지고 있다. 
drawAPoint() 메서드는 매개변수로 Point를 가지고 draw()함수를 호출해준다. 
point은 x,y를 0,0이라는 값을 가지게 생성(construct)되고, drawAPoint()함수를 호출하게 된다. 

이때 drawAPoint()와 draw()함수는 둘 다 `static dispatch` 이다. 
이게 무슨 의미인가 하면 compiler는 정확히 어떠한 구현이 실행될지 알고 있고 따라서 drawAPoint를 실제 drawAPoint가 수행하는 draw()함수로 바꿀 수 있게 된다. 

![](https://images.velog.io/images/minni/post/037e4efd-007c-477d-9fb8-a1e0faa05aba/image.png) 

즉, 위의 그림처럼 최적화를 수행할 수 있게 된다. 결국에 drawAPoint(point)함수는 point.draw()를 호출할 것을 compiler는 알고 있기 때문에 drawAPoint()를 draw()로 대체할 수 있게 된다. 
이와 같은 논리를 따르면 결국 point.draw()도 아래와 같이 대체될 수 있다. 

![](https://images.velog.io/images/minni/post/6366b6c7-528e-441c-9e68-ab9a1c9d4c22/image.png) 

결국에는 draw() 함수를 호출할 필요도 없이 바로 draw()함수의 실제 구현부를 실행하게 되는 것이다. 

따라서 실제로 run time에서는 point가 생성되고 나서 바로 run implementation을 진행하게 된다. 즉, 위에서 대체했던 2번의 과정에 대한 overhead가 발생하지 않게 된다.
이러한 이유로 static dispatch가 dynamic dispatch 보다 빠른 것이다. 

### dynamic dispatch와 static dispatch가 차이가 나는 이유
그러나 사실 single dynamic dispatch와 single static dispatch를 비교하면 그리 큰 차이는 존재하지 않다. 하지만 static dispatch의 체인은 compiler가 전체 체인을 통해서 가시성(visibility)를 가지게 되는 반면에 dynamic dispatch 체인은 모든 단계마다 추론을 하지 않고 상위 level에서 막히게 된다. 

결론은 위에서 본 Point Struct 예시처럼 static dispatch의 경우에는 compiler가 static method의 dispatch 체인을 마치 하나의 구현처럼 축소시킬 수 있고 결국 call stack overhead는 발생하지 않는다는 말이다. 

## dynamic dispatch는 그럼 왜 사용할까?
그렇다면 왜 overhead가 발생하는 dynamic dispatch를 사용하는 것일까? 
가장 큰 이유 중에 하나는 바로 **다형성(polymorhism)** 때문이다. 

![](https://images.velog.io/images/minni/post/9954f3d6-a444-43ae-aceb-f380dfc07448/image.png) 

위의 그림과 같은 전통적인 OOP(Object Oriented Program)을 보자. 
Drawable이라는 추상적인 class가 있고, 이를 상속받은 Point, Line이라는 클래스가 있다.
이들은 모두 draw()라는 함수를 자신들만의 특별한 실행으로 override 해서 구현해놓았다.

![](https://images.velog.io/images/minni/post/a4f04456-85d3-4c97-80d3-fbc61861df9a/image.png) 

그리고 나서 이제는 drawables라는 `Drawable` 타입을 가지는 배열을 선언해놓았다. 
이 배열은 Drawable을 상속한 lines, points를 가질 수 있고, 이때 각각의 원소들은 draw()라는 메소드를 호출할 수 있다. 

이것이 가능한 이유는 바로 Drawable, Line, Point는 모두 다 class이고 따라서 **이들은 모두 다 같은 크기로 참조형태로 배열에 저장**이 되기 때문이다.  

![](https://images.velog.io/images/minni/post/ea2a0b01-5ef8-45ac-9101-2ae6e948630f/image.png) 

그리고 실제로 배열에 저장되어 있는 참조는 Heap에 있는 내용들을 가리킬 것이고, Heap에서는 지난 포스팅에서 말했듯이 자체적으로 refCount를 가지고 있을 것이다. 

![](https://images.velog.io/images/minni/post/439dd625-da82-4d27-b547-0f5bcd9433aa/image.png) 

이제 코드에서 for문을 drawables 개수만큼 반복하면서 인스턴스에 대해서 draw()함수를 호출하게 된다. 
이때 compiler는 정확히 어떠한 구현을 실행해야 하는지, 즉 Line의 draw()를 실행해야하는지, Point의 draw()를 실행해야 하는지 compile time에 알 수 없게 된다. 

### compiler는 어떠한 메서드를 호출할지 어떻게 결정할까?
그렇다면 compiler는 결국 어떤 draw를 호출해야하는지 어떻게 결정하게 될까? 

![](https://images.velog.io/images/minni/post/5af0de5e-d5f7-42fa-8729-b7df546c05f3/image.png) 

compiler는 **class의 type 정보에 대한 포인터를 정적(static)메모리에 저장**하게 된다.
그래서 draw가 호출되었을 때, compiler가 실제로 생성하는 것은 바로 **type 및 정적(static) 메모리의 VTable(Virtual Method Table)라는 것을 조회**하게 된다. <br>
VTable 안에는 어떠한 구현을 실행해야 하는지에 대한 pointer를 저장하고 있기 때문에 **실행하기에 적합한 draw 메서드를 찾고나서 실제 인스턴를 파라미터로 전달해주게 된다**. 

### class를 static dispatch하는 방법
결과적으로 Swift에서 class는 **기본적으로 dynamic dispatch**를 하게 된다. 물론 이러한 dynamic dispatch만으로 큰 차이를 만들지는 않지만, 메소드 체인이나 inline과 같은 최적화는 위에서 본 예시처럼 막히게 된다. 
하지만 모든 dynamic dispatch를 할 필요는 없다. <br>
만약 **상속될 가능성이 없는 class의 경우에는 final** 키워드를 클래스 명 앞에 붙여주면, compiler는 이를 알아차리고 static dispatch를 하게 될 것이다. 
또한 compiler가 application에서 class가 상속이 이뤄지지 않는다는 것을 입증하고 추론할 수 있다면 이러한 dynamic dispatch를 대신하여 static dispatch를 하게 된다. <br>
(방법은 private, fileprivate를 명시해주면 된다)

## 결론
Swift의 성능을 3가지 측면에서 비교해 본 결과는 다음과 같다. 

![](https://images.velog.io/images/minni/post/b8da7751-e380-4fb2-ba46-f2af683c5daf/image.png) 

![](https://images.velog.io/images/minni/post/10c1b676-8c67-42ae-b962-f25db80d8f81/image.png) 

![](https://images.velog.io/images/minni/post/4859f100-27bd-49e0-87ac-c2022b66f8d0/image.png) 

Class, Final Class, Struct일 때 위의 그림과 같은 성능이 나온다고 한다. 

결론은 우리가 Swift에서 fast code를 작성하고 싶다면, 3가지 측면에 대해서 고민해 볼 필요가 있다.
- **allocation** : 객체가 memory에 allocation될 때, heap에 저장이 될 것인가 아니면 stack에 저장이 될 것인가?
- **Reference Counting** : 객체를 주고 받을 때, 얼마나 많은 reference counting overhead가 발생하는가?
- **Method dispatch** : 객체 내 method를 호출할 때 static dispatch인가 dynamic dispatch 인가?

그리고 만약 우리가 이유가 없이 dynamism이나 run time에 비용을 지불하고 있다면 이는 우리가 작성하는 fast code가 아닌 성능 측면에서 비효율적인 코드라는 것이다!!!

## 참고
[WWDC 2016 Understanding Swift Performance](https://developer.apple.com/videos/play/wwdc2016/416/?time=1474) <br>
[ZeddiOS 블로그](https://zeddios.tistory.com/596)