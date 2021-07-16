---
title:  "Engineering for Testability"
excerpt: "WWDC17 Engineering for Testability"

categories:
  - WWDC
tags:
  - [testing]
last_modified_at: 2021.07.14
---

# 들어가며

WWDC 2017 Engineering for Testability 에서는 2가지 큰 주제에 대해서 다룰 예정이다. 

1. **Testable app code**
    - code가 testable 하다는 것은 어떤 의미인가
    - 기존의 코드를 testability하게 개선하는 방법들

2. **Scalable test code**
    - Unit Test와 UI Test 간의 균형
    - UI Test를 확장 가능하게 작성하는 방법
    - Test 코드 품질의 중요성

<br>

# 1️⃣ Testable app code

테스트 코드를 작성하면 app code가 정상적으로 작동하는지와 이에 대한 확신, 그리고 시간이 지나 app이 성장하고 변경되어도 자신이 작성한 코드가 퇴행되는 것을 해결할 수 있고, 또 자신의 코드를 문서로도 작성할 수 있다는 장점들에 대해서는 많이 들어봤을 것이다. 그러나 막상 테스트 코드를 작성하려고 하면 어떻게 작성해야하는지 막막한 경우가 많다. 

## Unit Test의 구조

따라서 먼저 아래의 예시를 보자. Swift Standard Library의 Sorted 함수의 실행을 테스트하는 코드이다. 

![1](/assets/images/WWDC17EngineeringForTestability/1.png)

테스트는 아래와 같은 순서로 이뤄지게 된다. 

1. 정렬되지 않은 배열을 `input` 으로 가짐 
2. input 배열에 sorted 함수를 사용
3. 기대되어지는 값과 실제 sorted 함수가 적용된 input 배열을 비교하여 검증 

이러한 과정을 일반화하면 아래와 같이 생각할 수 있다. 

![2](/assets/images/WWDC17EngineeringForTestability/2.png)

1. 필요한 상태나 값으로 input을 준비
2. 실제로 테스트 할 코드를 테스트
3. 결과로 반환되는 값이나 상태를 검증

그리고 이러한 과정은 `Arrange Act Assert Pattern` 이라고 부르기도 한다. 

## Testable code의 특징

하지만 실제로 우리가 작성하고 테스트를 하려는 코드에는 위의 정렬 알고리즘 같은 코드가 없는 경우가 많다. 즉, 앱의 대부분의 코드는 정렬 알고리즘과는 상당히 다를 것이다. 그럼에도 불구하고 위의 정렬 알고리즘에 예시에서 우리는 app code를 더욱 `testable` 하게 만드는 몇 가지 특징들이 있다. 

`testable code` 는 클라이언트가 해당 테스트가 동작하는 **모든 input을 제어할 수 있는 방법**과 **모든 output을 검사할 수 있는 방법**을 제공한다는 점과 나중에 **코드의 동작에 영향을 줄 수 있는 내부 상태에 의존하지 않는다**는 점이다.  

그렇다면 이제 app code가 위와 같은 특징들을 가지고 testability 를 개선하는데 사용할 수 있는 기술들에 대해서 알아보자. <br>

## Testablility 기술 🔍

### 🔍 Protocols and parameterization

#### App 소개

첫 번째 기술은 protocol과 parameterization을 코드에 사용하는 방법이고, 이번 기술에서 예시로 사용하는 앱은 다음과 같다. Document browser 앱으로 다양한 종류의 문서를 왼쪽 화면처럼 preview에 보여주고, 그리고 수정하거나 더 자세하기 보기 위해서 segment control을 설정하고 open 버튼을 누르면 오른쪽 화면처럼 다른 앱으로 전환되는 기능을 가지고 있다. 

![3](/assets/images/WWDC17EngineeringForTestability/3.png)

#### 기존 작성 코드와 테스트 코드

이제 작성된 코드를 살펴보자

![4](/assets/images/WWDC17EngineeringForTestability/4.png)

1. ViewController의 event handler는 open 버튼이 클릭 되었을 때 작동하게 된다. 
2. segmentControl에 따라서 URL을 생성하기 위한 business logic 부분 
3. UIKit가 제공하는 UIApplication의 shared 객체를 사용해서 생성한 url을 열 수 있는지 확인하고 가능하면 해당 url로 open 수행
4. 생성한 url이 열 수 없다면 Error 처리 호출



이에 open 버튼이 정상적으로 작동하는지 테스트해보자. 이때 이를 테스트하는 방법에는 다양한 방법이 존재할 수 있다. 그 중 하나는 `UITest` 를 작성하는 방법이다. 

>  앱 실행 → 해당 화면으로 이동 → open버튼을 누르기 전에 설정 → open 버튼 탭 → ✅ 다른 앱으로 전환되는지 확인

위와 같이 `UITest` 를 진행하면 작동은 확인할 수 있으나, 이렇게 테스트를 작성하게 되면 단점이 발생하게 된다. 만약 여러 종류의 문서에 대해서 open 기능을 테스트하려면 **시간이 오래 걸릴 것**이다. 더 큰 문제는 `UITest` 는 **앱을 다른 앱으로 전환시키는데 사용되는 생성된 URL을 검사할 수 있는 방법이 없다**는 점이다. 

따라서 이러한 경우에는 `Unit Test` 가 필요로 되어지고 이를 위해서 아래와 같이 테스트 코드를 작성해보자. 

![5](/assets/images/WWDC17EngineeringForTestability/5.png)

1. 작업을 할 ViewController의 객체가 필요하므로 controller 변수에 선언
2. View를 load하여 control 하는 속성들과 view data들을 채워줌
3. open 버튼을 누르기 전에 설정이 필요한 요소(segmentControl) 설정
4. document 제공
5. open 버튼을 탭

그러나 이렇게 Unit Test로 작성을 하게되어도 실제로 마지막에 무엇을 검증해야 하는지에 대해서 의문이 생기게 된다. 

따라서 다시 원래 코드로 돌아가서 무엇이 이 테스트를 어렵게 만드는지 확인해보자. 

![6](/assets/images/WWDC17EngineeringForTestability/6.png)

1. View Controller에 있는 것만으로 테스트 방법이 복잡해지게 된다. 왜냐하면 View Controller 객체와 함께 작동하려면 몇가지 검토할 것들이 있기 때문이다. 
2. `input` 의 상태(segmentedControl의 상태)를 View에서 직접 가져온다. 이러한 방법을 테스트의 일부 하위 뷰에 속성을 설정하여 `input` 을 간접적으로 제공하도록 해야함
3. `UIApplication.shared` 객체를 사용한 것이 가장 큰 문제이다. 
4. `canOpenURL()` 의 반환 값은 다른 method의 input 이고, 이는 전체 시스템의 상태에 따라서 달라지므로 해당 query에 결과를 제어할 수 있는 프로그래밍 방법이 존재하지 않는다. 
5. `open()` 을 사용하여 URL을 열때 부작용을 관찰한 Unit Test 작성 방법이 존재하지 않고, 게다가 이 함수를 사용하게 되면 app이 백그라운드로 전송되며 이후에 다시 앱을 foreground 상태로 가져오는 방법은 존재하지 않는다. <br>

#### Unit Test가 가능하도록 수정

위의 코드의 testability를 높이기 위해서 다음과 같이 수정해보자. 

![7](/assets/images/WWDC17EngineeringForTestability/7.png)

먼저 View Controller로부터 꺼내는 것부터 이를 시작할 수 있는데, 이러한 logic과 동작을 캡슐화하기 위해서 새로운`DocumentOpener` 클래스를 생성한다. 그리고 open mode와 document 에 대한 input은 함수의 매개변수를 통해서 받을 수 있게 한다. 
<br> ⚠️ 그러나 여전히 `UIApplication.shared` 를 사용한다는 문제를 해결해야만 한다. 

![8](/assets/images/WWDC17EngineeringForTestability/8.png)

따라서 이를 해결하기 위해서는 해당 메소드로의 직접 사용을 중지하고, 클래스에 `init()` 에 새롭게 생성한 application 객체를 매개변수로 전달할 수 있도록 수정한다. 이때 박스로 표시된 `= UIApplication.shared` 를 입력하므로서, **기존의 ViewController에서는 변경하지 않아도 작동**할 수 있게한다. 그리고 `open()` 함수에서 기존의 `UIApplication.shared` 를 새로운 객체로 변환한다. 

이제 다시 테스트 코드로 가서 작성을 해보자. 

![9](/assets/images/WWDC17EngineeringForTestability/9.png)

테스트 코드를 작성하려고 해봐도 여전히 문제는 해결되지 않는다... 왜냐하면 우리는 통제할 수 있는 UIApplication 객체를 생성해서 매개변수로 전달해야하기 때문이다. 이를 위해서는 UIApplication을 상속하여 `canOpenURL()` , `open()` 함수를 오버라이딩 하는 방법이 있겠지만, **UIApplication은 singleton이 엄격하게 적용되어 다른 객체를 생성하려고 하면 error를 던지게 된다.** <br>

##### 프로토콜을 사용한 해결 방법

따라서 상속이 아닌 **프로토콜**을 사용해서 이를 해결해보자. 

![10](/assets/images/WWDC17EngineeringForTestability/10.png)

`URLOpening` 이라는 프로토콜을 생성하고, 기존의 UIApplication과 동일하게 `canOpenURL()` 과 `open()` 함수를 가지도록 한다. 그리고 UIApplication이 `URLOpening` 프로토콜의 기본 구현이도록 하기 위해서 UIApplication을 extension하여 해당 프로토콜을 채택하도록 한다. 여기서 기존의 UIApplication.shared의 함수와 동일하게 함수를 선언했기 때문에 extension에 코드를 작성할 필요는 없다. 

이제 실제 클래스로 가서 새롭게 구현한 프로토콜을 따르도록 수정해주자.

![11](/assets/images/WWDC17EngineeringForTestability/11.png)

기존에 UIApplication 타입이었던 것들을 새롭게 생성한 `URLOpening` 타입으로 변경해주고 변수명도 수정해준다. 이때 역시나  ` init()` 함수에 기본값으로 `= UIApplication.shared` 작성하여 ViewController에서 사용할 때에는 변경할 필요 없게 해준다. <br>

##### 테스트 코드 작성

이제 다시 테스트 코드로 돌아가서 작성해보자. 이때 UIApplication은 우리가 테스트할 때 필요한 제어 기능을 제공하지 않기 때문에, 이를 대신해서 사용할 프로토콜의 2번째 mock을 구현해야 한다. 아래와 같이 `URLOpening` 프로토콜에서 선언한 2개의 함수의 실행에 대해서 구현해야한다. 

![12](/assets/images/WWDC17EngineeringForTestability/12.png)

`canOpenURL()` 함수는 documentOpener의 input 역할을 해주고, 그 결과 이제 테스트에서 input을 제어할 수 있게 된다. 그리고 `canOpen` 이라는 변수를 반환 값으로 넘겨주면 된다. 

`open()` 함수는 documentOpener의 output 역할을 해주고, 이때 함수 내부에서는 전달된 url에 접근하는 것을 `openedURL` 이라는 URL? 타입의 변수에 저장하여 나중에 확인할 수 있도록 한다. 

이제 마지막으로 테스트 코드를 작성하면 다음과 같다. 

![13](/assets/images/WWDC17EngineeringForTestability/13.png)

1. 새롭게 생성한 `MockURLOpener` 클래스의 객체를 생성하고 `canOpen` 속성을 지정하여 input을 구성
2. 새롭게 생성한 `DocumentOpener` 클래스에게 mockURLOpener를 매개변수로 넘겨주며 생성
3. 테스트를 위한 설정이 끝났으니 `open()` 에 Document와 mode를 매개변수로 넘겨주며 호출하여 테스트
4. mockURLOpener의 `openedURL` 속성과 기대되어지는 URL 값을 비교하여 검증 

이를 통해서 `open()` 가 호출되었고, 전달된 URL이 올바른 데이터임을 검증할 수 있다. 여기서 `canOpen` 의 속성을 false로 변경하거나, 다른 입력 데이터를 주어서 테스트를 계속 작성할 수 있다. 

#### 요약

물론 아직 모든 것이 완벽하게 해결되지는 않았지만 일부 문제들은 해결되었으므로 일단 넘어가면서 요약해보자. 우리는 기존 코드에서 Unit Test가 가능하도록 refactoring을 수행하였는데, 다음과 같은 것들을 진행했다. 

1. Singleton 객체인 UIApplication.shared 에 대한 **외부 참조를 추출해내고 이를 매개변수화된 입력으로 제공하여 대체** <br>
   이러한 방법을 **`의존성 주입(Dependency Injection)`** 이라고 하고, init() 매개변수를 사용하고, 테스트 되는 함수의 property setter나 매개변수를 사용한다.  
2. **기존에 의존하고 있던 불변하는 클래스로부터 코드를 분리시키기 위해서 새로운 프로토콜을 생성**
3. 테스트에서 기존의 것을 대체하고 사용할 것인 mock 구현을 작성

✅ 이를 통해서 `input` 에 대한 제어권을 얻을 수 있었고, 가시성 있는 `output` 을 제공할 수 있게 되었다. 

### 🔍 Separating logic and effects

이번에는 논리(logic)을 효과와 분리하는 것이 testability를 높이는데 어떻게 사용되는지 알아보자.

#### 예시 소개

이번 기술에서 사용하는 예시는 다음과 같다. 앱에서 이전에 서버에서 다운로드한 data들을 빠르게 검색하기 위해서 사용할 수 있는 `OnDiskCache` 클래스이다. 

![14](/assets/images/WWDC17EngineeringForTestability/14.png)

이 캐시는 구조체 Item으로 저장하는 항목을 표현하고, 내부에는 파일 시스템 내부의 `path` , 저장된 기간을 나타내는 `age` , 그리고 해당 Item의 크기인 `size` 를 가지고 있다. 또한 `currentItems` 를 통해 캐시 내부에 저장된 모든 Item 집합을 가져올 수 있다. 

또한 캐시를 정리해주는 `cleanCache()` 함수를 가지고 있고, 이 함수는 주기적으로 호출되어서 파일 시스템 내부의 너무 많은 양을 차지하지 않도록 해야한다. 함수에서는 다음과 같은 순서로 구현되어 있다. 

1. 현재 모든 Item들을 최신 Item부터 가장 오래된 Item 순으로 정렬한다.
2. 정렬된 모든 Item 항목에 대해서 item 크기의 누적해서 더해나간다. 
3. 만약 누적된 합인 `cumulativeSize` 가 캐시의 최대 크기보다 커진다면 해당 Item을 캐시에서 삭제해준다.

#### 테스트에서의 문제점

그렇다면 이러한 `cleanCache()` 함수를 테스트 한다고 할때 `input` 과 `output` 은 어떤 것이 될 지 알아보자. 

![15](/assets/images/WWDC17EngineeringForTestability/15.png)

`input` 중 하나는 캐시 크기를 지정할 매개변수인 `maxSize` 이며, 이미 `cleanCache()` 함수의 매개변수로서 테스트에서 제어할 수 있다. 또 다른 `input` 은 현재 캐시에 저장된 Item 목록인 `currentItems` 인데, 이때 문제가 되는 부분은 이를 사용하기 위해서는 **파일 관리자를 사용해 디스크에서 파일 목록을 검색한다는 점**이다. 이는 입력이 실제로 파일 시스템에서 파생되어진 것이며, 테스트에서 해결해야할 종속성 문제이다. 

`output` 의 경우에는 `cleanCache()` 함수는 반환 값이 없다. 따라서 `output` 은 데이터가 아니라, 디스크로부터 특정 Item들이 사라지는 부작용(side effect)이 될 것이다.  

파일 시스템에 대한 종속성 때문에 `cleanCache()` 함수에 대한 테스트는 파일 시스템 관리자를 사용해야한다. 테스트 setup을 위해서는 임시적인 디렉토리를 생성하여 특정 크기의 파일로 채우고, 파일들에게 각각 특정한 timestamp를 제공해주어야한다. 그리고 `output` 을 검증하기 위해서 파일 시스템으로 돌아가서 어떠한 파일들이 남아있는지를 확인해야한다. 



##### Protocols and parameterization을 사용한 해결과 문제점

![16](/assets/images/WWDC17EngineeringForTestability/16.png)

위와 같은 방법에 접근하는 법은 그 전에 다뤘던 기술인 **Protocols and parameterization을 사용하는 것**이다.  File manager 프로토콜을 새롭게 생성하여 파일 목록을 가져오는 함수와, 제거하는 함수를 가지게 한다. 그리고 반환되는 파일 목록과 제거된 파일 쿼리를 지정할 수 있는 것을 test 코드에서 구현해주면 될 것이다. 

그러나 이렇게 해도 **파일 관리자가 아직도 테스트하려는 코드와 간접적으로 상호작용**을 하고 있게 된다. 



##### 로직을 분리하는 방법으로 해결

`cleanCache()` 함수에서 제거되는 파일을 결정하는 책임 로직을 배제하고 이를 `CleanupPolicy` 로 생성하여 직접적으로 상호작용할 수 있게 하자. 

![17](/assets/images/WWDC17EngineeringForTestability/17.png)

먼저 `CleanupPolicy` 를 프로토콜로 생성하자. 이 프로토콜에는 파일을 삭제하는 `itemsToRemove()`  하나의 함수만 있으면 된다. 

![18](/assets/images/WWDC17EngineeringForTestability/18.png)

이때 기존의 `OnDiskCache` 클래스의 `cleanCache()` 와 새롭게 만든 프로토콜의 `itemsToRemove()` 함수를 비교해보면 다음과 같다. `itemsToRemove()` 함수의  `input` 은 캐시에 저장되어있는 파일 집합이고, `output` 은 삭제된 파일 집합을 반환하게 된다는 것을 확인할 수 있다. 

이제 새롭게 생성한 `CleanupPolicy` 프로토콜을 채택하여 기존의 `cleanCache()` 의 알고리즘을 구현해보자. 

![19](/assets/images/WWDC17EngineeringForTestability/19.png)

먼저 `MaxSizeCleanupPolicy` 구조체를 생성하고 `CleanupPolicy` 프로토콜을 채택한다. 이후 그림에서 노란색으로 표시된 `maxSize` 를 프로퍼티로 가지고 `itemsToRemove()` 함수를 가지도록 구현한다. 그리고 `itemsToRemove()` 함수 내부에는 주황색으로 표시한 `output` 으로 제공하기 위해서 파일 집합을 생성하고, 제거되는 파일들을 집합에 추가하여 반환하도록 한다. 이때 파일들을 정렬하고, 누적 사이즈의 크기를 계산하는 과정은 기존의 `cleanCache()` 에서의 알고리즘과 동일하다. 

![20](/assets/images/WWDC17EngineeringForTestability/20.png)

위의 코드를 통해서 기존의 `OnDiskCache` 클래스에서 나타나는 부작용은 제거되었지만, **기존 알고리즘을 그대로 따르며 `input` 으로 데이터를 넘기고 `output` 으로 데이터를 반환하도록 수정**된 것을 확인해 볼 수 있다. 이를 통해서 위와 그림과 같이 데이터 흐름을 시각화할 수 있다. 여기서 코드가 **데이터의 입력, 데이터의 출력이라는 매우 기능적인 스타일을 취했다는 점**을 유의해야한다. 



##### 테스트 코드

위에서 새롭게 작성한 코드를 기반으로 테스트 코드를 작성하면 다음과 같이 작성할 수 있다. 

![21](/assets/images/WWDC17EngineeringForTestability/21.png)

1. `input` 으로 사용할 몇개의 캐시 처리된 아이템들의 집합을 정의
2. 새롭게 구현한 `MaxSizeCleanupPolicy` 구조체의 인스터스를 `maxSize` 매개변수와 함께 초기화하고, `itemsToRemove()` 함수를 호출하면서 매개변수로 미리 만들어놓은 `input` 을 넘겨주어 반환 값을 `ouputItems` 에 저장
3. 실제 결과와 예상되어지는 결과를 비교하여 검증

위와 같은 형식의 코드를 사용하여 **`input` 을 쉽게 제어하고, `output` 에 대한 가시성도 확보하게되며, 숨겨진 상태도 없게 된다.** 

따라서 **파일 시스템과 같은 느린 리소스에 의존하지 않기 때문에 빠르게 실행**되며, 테스트 되어야하는 **중요한 부분에 있어서 주의가 분산되지 않아 테스트를 읽기 쉽게** 만든다. 그리고 **사용되는 모든 데이터를 테스트 내부에 스스로 가지고 있게** 된다. 

이제 기존의 `OnDiskCache` 클래스를 보면 추출되고 나서 남은 것이 거의 없게 된다. 

![22](/assets/images/WWDC17EngineeringForTestability/22.png)

`CleanupPolicy` 인스터스인 policy의 `itemsToRemove()` 함수를 사용하여 제거할 파일 목록을 받게 되고, 해당 목록을 반복하면서 파일 시스템에서 제거해준다. 

남은 테스트를 위해 FileManager 프로토콜과 테스트 구현코드를 작성하여 매우 독립적인 Unit Test를 작성하거나, Integration Test(통합테스트)를 통해 코드가 옳바르게 동작하고 있다는 것을 확인할 수 있다. 



#### 요약

이번 예시를 통해서 다음과 같은 것들을 알아보았다. 

1. **비지니스 로직과 알고리즘을 별도의 타입으로 추출**하는 방법
2. 이러한 기술을 사용할 때에 **알고리즘**은 `input` 과 `output` 을 설명하기 위해서 **값 타입(value type)을 사용하는 다소 기능적인 스타일을 취함** <br>
   → 이를 통해서 필요한 만큼 상세하게 알고리즘을 수행하는 Unit Test가 가능
3. 컴퓨터 데이터에 기반으로 side effect를 실행시키는 소량의 코드만 남김 <br> 
(e.g. 추출되고 남은 `cleanCache()` 함수)


## Testable app code 요약

Testable app code 파트에서는 app 코드를 구조화 할 수 있는 두 가지 기술에 대해서 살펴보았다. 

그리고 이를 통해서 테스트 코드를 작성할 때 `input` 에 대한 제어권과 `output` 에 대한 가시성을 가지게 되고, 그 결과 효과적인 Unit Test 코드를 작성할 수 있게 된다. 

# 2️⃣ Scalable test code

이제 테스트 코드의 확장성(scalable)을 높이는 방법과 테스트를 더 빠르고, 읽기 쉬우며, 모듈화 할 수 있는 방법들에 대해서 알아보자. 

## UI Test와 Unit Test간의 균형

먼저 `UI Test` 와 `Unit Test` 간의 균형을 잘 잡아야한다. 아래의 Test Pyramid를 보자. 

![23](/assets/images/WWDC17EngineeringForTestability/23.png)

중간에 `Integration Test` 도 있지만 `UI Test` 와 `Unit Test` 에만 더 집중해서 보자. 

`UI Test` 

- 테스트 피라미드의 맨 위에 위치
- 테스트의 양이 적다
- 테스트하는데 시간이 오래 걸림
- 유지 보수 비용이 높은편
- 실패 시 무엇이 잘못되었는지 즉시 알기 어려움 

`Unit Test` 

- 테스트 피라미드의 가장 아래에 위치
- UI Test에 비해서 훨씬 양이 많음
- UI Test에 비해서 훨씬 빠르게 실행됨
- 유지 보수 비용이 낮은편
- 실패 시 무엇이 잘못되었는지 즉시 알 수 있음

이와 같이 Test pyramid는 테스트의 분포를 표현하기에 좋은 방법이지만, **모든 상황을 나타내는 것이 아니다.** 따라서 일부 UI Test는 unit test와 유사할 수 도 있고, Unit Test도 독립적으로 분리된 것이 아니라 다른 코드 모듈과 상호작용하는 경우도 있을 수 있다. 

즉, **Test pyramid는 좋은 예시이지만 상황에 따라서 유연하게 테스트를 작성**할 필요가 있다. 따라서 `UI Test` 와 `Unit Test` 를 작성할 때 **각각의 강점을 고려해서 작성**하면 된다. 

`Unit Test` 의 강점은 **모든 app의 소스 코드에 접근하지 않고는 도달하기 어려운, 크기가 작은 코드들을 테스트하는데 유용**하다는 점이다. 

반면 `UI Test` 의 강점은 **많은 코드들을 테스트 해야할 때 유용**하다는 점이다. 물론 `Unit Test` 는 app의 모든 소스 코드에 접근 가능한 반면 `UI Test` 는 그렇지 않다는 점을 알아두어야 한다. 

## UI Test를 확장 가능하게 작성하는 방법

이제 `UI Test` 에서 테스트 코드의 품질을 올릴 수 있는 방법에 대해서 알아보자. 제시하는 세 가지 방법을 통해서 애플리케이션 코드에 따라서 확장 가능한 테스트 코드를 쉽게 생성할 수 있다. 

### 🔍 UI element 쿼리의 추상화

#### 예시 소개

다음과 같이 ViewController에 여러 개의 버튼이 있는 App이 있다고 가정해보자. 

![24](/assets/images/WWDC17EngineeringForTestability/24.png)

이때 각각의 버튼은 동일 View hierarchy에 존재하고 있고, 유일한 **차이점은 바로 각 버튼의 이름**이다. 따라서 이러한 쿼리를 7번 작성하는 것보다는 아래와 같이 함수를 사용해서 이를 개선해보자. 



#### 쿼리를 추상화 하기

![25](/assets/images/WWDC17EngineeringForTestability/25.png)

`tapButton()` 함수를 생성하여 이를 호출하는 식으로 개선할 수 있다. 그러나 이 조차도 버튼의 이름을 제외하고 똑같은 코드를 작성하기 때문에 이를 더욱 개선할 수 있다. 

모든 버튼 이름을 배열에 넣고 이를 반복하는 방식으로 개선할 수 있다. 이를 통해서 추후에 버튼이 추가되더라도, `tapButton("newButtonName")` 처럼 새로운 **코드 작성이 아닌 배열에 버튼 이름만 추가**해주면 된다. 그 결과 **유지보수에서의 이점**을 확인해 볼 수 있다. 

`UI Test` 의 특성상 이러한 쿼리를 많이 실행하게 된다. 따라서 **동일한 쿼리를 여러 번 사용하는 경우에는 비록 쿼리의 일부분이라도, 이를 변수로 저장**하는 것이 좋다. 또한 **유사한 쿼리가 있는 경우에는 helper method로 생성하여 코드를 읽기 쉽고 깔끔하게 만들 수 있다.** 

이처럼 **테스트 코드 줄 수의 단축과 helper method의 사용은 UI Test에서 새로운 테스트를 더 빠르고 쉽게 구현**할 수 있게 된다. 



### 🔍 객체와 유틸리티 함수의 생성

#### 예시 소개

이번에는 아래과 같이 테스트에서 게임의 설정(난이도, 소리)을 변경하는 코드를 작성한다고 하자. 

![26](/assets/images/WWDC17EngineeringForTestability/26.png)

위와 같은 코드는 **확장성 있게 작성된 코드가 아니다**. 물론 이 코드를 작성한 사람은 자신이 작성하였기 때문에 모든 것이 어떻게 구성되어있고 작동하는지 알고 있을 수 있다. 그러나 추후에 이 코드를 본다거나, 작성자가 아닌 다른 누군가가 이러한 코드를 본다고 할 때에는 이해하기 어려운 코드임에 틀림없다. 

위의 코드를 이해하기 위해서 아래와 같은 상황을 인지하고 있어야한다. 

1. 들어갔다 나갔다 할 수 있는 `Setting Page` 가 있다는 것을 알아야함
2. `난이도`를 설정하는 Page가 있다는 것을 알아야하고 설정 후에 다시 뒤로 가야함
3. `사운드`를 설정하는 Page가 있다는 것을 알아야하고 설정 후에 다시 뒤로 가야함

게다가 만약 실제 UI를 변경해야하는 일이 발생하게 된다면 위의 코드로 테스트를 진행할 수 없게 될 것이다. 



#### 유틸리티 함수로 추상화하기

따라서 이러한 **로직을 helper method(유틸리티 함수)안으로 넣어서 추상화 시키도록 수정**하자. 

![27](/assets/images/WWDC17EngineeringForTestability/27.png)

먼저 노란색으로 표시한 부분처럼 `setDifficulty()` 함수와 `setSound()` 라는 함수로 기존의 각각의 설정창에 들어가서 설정하고 뒤로 가는 로직을 추상화하여 개선할 수 있다. 

또한 여기서 추가로 초록색으로 표시한 부분처럼 두 함수의 **매개변수의 타입인 String을 Enum 타입으로 변경**하여, **컴파일 시 넘겨주는 매개 변수가 타당한지도 확인**할 수 있도록 추가 개선할 수 있다. 

![28](/assets/images/WWDC17EngineeringForTestability/28.png)

이렇게 구현한 helper method로 기존 테스트 코드를 대체하게 되면 위의 코드와 같이 훨씬 줄어든 것을 확인해 볼 수 있다. 그러나 여기서도 설정 페이지에 들어가거나 나오는 부분에 대해서 개선할 수 있는 방법이 있다. 



#### GameApp 객체 생성

![29](/assets/images/WWDC17EngineeringForTestability/29.png)

위의 코드 처럼 `GameApp` 클래스를 만들고 앞서 구현한 helper method와 enum 타입을 포함하도록 작성한다. 그리고 `configureSettings()` 라는 함수를 새롭게 생성하여, 난이도와 소리 설정을 매개변수로 받게 하고 이전에 테스트 코드에서 작성했던 설정 logic을 그대로 이전해오면 된다. 

![30](/assets/images/WWDC17EngineeringForTestability/30.png)

이제 테스트 코드에서는 기존에 작성했던 코드 대신, `GameApp` 클래스를 인스턴스화 하는 동시에 `configureSettings()` 함수를 난이도와 사운드 설정을 매개변수로 넘겨주며 호출하여 초기 설정을 하도록 수정할 수 있다. 따라서 테스트에서 **초기 설정을 해줘야하는 경우에 `configureSettings()` 함수만 호출**하면 되고, 만약 **설정하는 부분을 수정해야 한다면 해당 함수만 수정**하면 된다. 

이 예시를 통해서 테스트에서 확장 가능한 코드를 작성할 때 가장 중요한 작업 중 하나는 바로 **나중에 라이브러리처럼 사용할 수 있도록 추상화 시키는 것**이다. 이를 통해서 하나 이상의 테스트에서 적용할 수 있는 **공통된 작업들을 캡슐화** 할 수 있게 되고, 여러 플랫폼에서 테스트 코드를 공유할 수 있게 된다. 

그 결과 **유지보수성도 개선**되며, 추상화된 작업들에 관련된 변경 사항이 있다면 **코드를 한 곳에서만 수정**하면 된다. 



#### XCTContent의 사용

또 하나의 개선 사항은 `configureSettings()` 함수를 작성할 때, `XCTContext` 를 사용하는 방법이다. 아래와 같이 기존 로직을 `XCTContext.runActivity` 블록 내부에서 실행하도록 하는 것이다. 

![31](/assets/images/WWDC17EngineeringForTestability/31.png)

그 결과 테스트 이후에 노란색 박스와 같은 result bundle에서 log를 얻을 수 있게 되고, 기존보다 훨씬 편리하게 log를 정리할 수 있다. 



### 🔍 UI Test를 위한 macOS 키보드 단축키 활용

#### 예시 소개

아래의 그림에서 왼쪽의 표준 MacOS 색상 선택기를 사용해 텍스트의 색상을 선택할 수 있는 앱이 있다고 할 때, 색상이 제대로 설정되었는지 확인하기 위한 테스트 코드를 작성한다고 하자. 

![32](/assets/images/WWDC17EngineeringForTestability/32.png)

이때 App에서 색상 선택기를 불러오는 방법은 `Format 메뉴 → Font 메뉴 → Show Colors 선택` 하는 것이다. 그리고 이를 `UI Test` 에서 작성하려면 오른쪽과 같이 코드로 작성할 수 있다. 

![33](/assets/images/WWDC17EngineeringForTestability/33.png)

이때 Show Colors 메뉴에 연결된 **바로 가기 키를 활용**하여 여러 줄의 코드를 작성하는 대신 위와 같이 한 줄의 코드로 수정할 수 있다. 그리고 이렇게 만든 코드를 추상화하여 `showColors()` 라는 함수로 만들어 가독성을 높여준다. 

![34](/assets/images/WWDC17EngineeringForTestability/34.png)

이제 위의 그림처럼 테스트에서 **유지해야할 코드가 줄어들 뿐만 아니라, 실제로 테스트 하려는 것과 직접적으로 관계가 없는 실행되어야할 코드도 줄어들게 된다.** 또한 테스트 속도도 빨라지고, 가독성도 좋아지게 된다. 

위의 예시와 같이 **MacOS 애플리케이션의 `UI Test` 를 작성하는 경우에는 메뉴바를 거치지 않고 단축키를 사용**할 수 있다. 물론 한 번 정도는 메뉴바를 통해서 색상 선택기를 불러오는 것이 제대로 작동하는지 테스트해 볼 수는 있지만, 이러한 행위를 반복해서 할 필요는 없고, 위와 같이 수정하여 **코드의 양을 줄이고 가독성을 높일 수 있다.** 



## 테스트 코드 품질의 중요성

마지막으로 **좋은 테스트 코드를 작성하는 것이 곧 좋은 코드를 작성하는 것이라는 것**을 알아야한다. 

대부분의 우리는 App 코드를 작성하는데 있어서는 좋은 디자인 원칙을 지켜가면서 코드를 작성하여, 가능한 한 앱을 최고로 만들기 위해서 시간과 노력을 많이 투자한다. 

그러나 테스트 코드에 대해서는 단지 추가적인 요구사항이라고 생각하고 마치 마지막 체크 박스를 채우기 위해서 급하게 작성하곤 한다. 하지만 App 코드를 작성하는 것과 같이 **세부적인 것에 주의를 기울이지 않으면, 테스트 코드 또한 확장할 수 없게된다.** 

![35](/assets/images/WWDC17EngineeringForTestability/35.png)

따라서 다음과 같은 생각을 가지고 테스트 코드를 작성하는 것이 좋다. 

- 비록 실제로 배포되지는 않지만 테스트 코드 또한 중요하다.
- 테스트 suite는 App의 진화를 지원해야하는 것이며, 변경에 따라 방해가 되면 안된다. <br>
  즉, 낮은 품질의 테스트 코드에서는 App을 변경할 때 마다, 테스트 코드도 업데이트를 해야하는 부담이 생기게 된다. 그러나 품질을 생각하며 테스트 코드를 설계하면, 확정 능력에 있어서 제한되지 않게 된다. 
- App 코드를 작성하는데 적용되는 원칙들은 테스트 코드에도 적용되어야 한다. <br>
  즉, 테스트 코드와 App 코드는 동일하게 보아야 한다. 


![36](/assets/images/WWDC17EngineeringForTestability/36.png)

이러한 방법을 실천하는 방법으로는, 테스트 코드로 code review를 하는 것이 아닌 **테스트 코드에 대한 code review를 하는 것**이다. 이렇게 하면 다른 사람이 내가 작성한 테스트나 테스트 커버를 확인할 수 있고, 이를 통해서 테스트 코드를 향상시킬 수 있게 된다. 

![37](/assets/images/WWDC17EngineeringForTestability/37.png)

실제로 전체의 반은 App 코드이고, 나머지 절반은 이를 검증하는 테스트 코드라고 할 수 있다. 그리고 App 코드를 업데이트 하게되면, 테스트 코드도 업데이트를 해야한다. 

하지만 앞으로는 이를 별개로 생각하지 말고, **App 코드와 테스트 코드는 하나라고 생각**해야한다. 그렇게 이번 세션에서 초반 부분에 언급한 것처럼 **코드를 더욱 테스트 가능하게 작성**하고, **테스트 코드와 App 코드를 동일하게 취급**함으로써 **전체 App의 품질을 향상**시킬 수 있을 것이다. 


# 참고 문서

[WWDC2017 Engineering for Testability](https://developer.apple.com/videos/play/wwdc2017/414/?time=2178)

