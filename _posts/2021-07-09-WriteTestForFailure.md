---
title:  "[WWDC] Write Tests to Fail"
excerpt: "WWDC20 Write Tests to Fail"

categories:
  - WWDC
tags:
  - [testing]
last_modified_at: 2021.07.09
---

# 들어가며
기존에 `Unit Test` 나 `UI Test` 의 경우에는 초록색 박스, 즉 통과를 하기 위해서 테스트를 진행했다면, 이제 테스트에 실패했을 때 이를 잘 분류하고, 미리 사전에 방지할 수 있는 방법들에 대해서 알아보자. 

실제로 테스트는 한 번만 작성되지만 여러 번 실행되고 결과를 분류하게 된다. 테스트가 프로그램에서 버그를 찾게 되면 테스트는 실패하게 되고, 이것은 바로 테스트가 의도하는 것이다. 이때 지속적인 통합 시스템에서의 테스트에서 **실패를 원인을 파악하는 도구는 바로 test result bundle**이다. 이번 세션에서는 test result bundle을 사용해서 테스트를 보다 쉽게 분류할 수 있는 방법과 테스트를 보다 견고하게 만들어 테스트를 디버깅하지 않고도 프로그램의 실패를 분류하는 방법에 대해서 알아보자. 

## Test template
우리가 기존에 사용하는 테스트 템플릿은 `set up`, `test`, `tear down` 의 테스팅 패턴을 따르게 된다. 그리고 `test` 섹션은 `test: actions` 과 `test: assertions` 로 세세하게 분류할 수 있다. 각각에 대해서 알아보자. 



### 1️⃣ set up

그렇다면 이제 `set up` 부터 알아보자. `set up` 은 테스트를 실행하기 전에 요구하는 가정을 명시적으로 작성하고 앱과 환경 상태를 설정하는 부분이다. 

Xcode 11.4에서 새로운 setup 함수인 `setUpWithError()` 가 소개되었다. `setUpWithError()` 는 throws 키워드를 사용하여 setup 도중에 발생하는 error들을 catch 하거나 pass 할 수 있다. 기존의 `setUp` 함수를 `setUpWithError` 함수로 변경하게 되면, error를 관리하는 측면에서는 이점을 가져갈 수 있다. 또한 이전에 진행한 test가 앱의 상태나 data를 수정할 수 있기 때문에 `setUpWithError` 함수 내부에는 **테스트가 진행되기 전의 초기 상태로의 설정**하는 코드를 넣어줄 수 있다. 

![1](/assets/images/WWDC20WriteTestForFailure/1.png)

1. `continueAfterFailure = false` 로 설정하여 만약 문제가 발견되면 테스트가 즉시 실패하게 된다. 이렇게 하면 나중에 여러 오류들 중에서 발생한 error를 찾는 것보다 **훨씬 빠르게 첫 번째 error를 찾을 수 있게 된다.** 
2. `let app = FrutaApp()` 과 `app.launch()` 를 통해서 `setUpWithError` 내부에서 **각각의 테스트에서 새롭게 앱을 시작**할 수 있도록 한다. 
3. `app.launchArgument.append("")` 를 통해서 **launchArgument와 환경 상태를 설정**하여 앱 내에서 상태를 빠르게 설정할 수 있다. 이를 통해서 모든 상태를 설정할 수는 없지만, 테스트 도중에 이중 인증을 생략하는 것과 같이 필요한 경우가 있다. 

   또한 아래와 같이 코드를 작성해서 기존에 앱을 들어가면 보이는 왼쪽 시뮬레이터에 나와있는 메뉴 탭을 건너뛰고, 바로 오른쪽 시뮬레이터처럼 레시피 탭에서 시작하도록 사용할 수도 있다. 

   ![2](/assets/images/WWDC20WriteTestForFailure/2.png)

   이와 같은 작은 변경 사항은 결국 **불필요한 작업을 방지하여 테스트 실행 속도를 향상**시킬 수 있다는 점이다. 더욱 중요한 것은 **레시피 탭에서의 메뉴에 대한 테스트 결과에 대해서만 신경쓰면 되고, 메뉴 탭에서의 발생하는 error에 대해서는 분류할 필요가 없다는 것**이다. 

#### 요약

`setUpWithError` 를 사용하여 

1. error 처리 능력이 향상
2. 모든 테스트에 대해서 앱 실행과 같은 일반적인 설정을 수행
3. launchArguments를  사용해 앱의 상태를 설정



### 2️⃣ Test: action

각각의 테스트들은 **구체적인 목표**를 가져야하며 이러한 목표는 **test 함수명으로 반영하여 작성해야한다.**

![3](/assets/images/WWDC20WriteTestForFailure/3.png)

위의 예시인 `testIngredientsListAcuuracy()` 테스트 함수의 목표는 **재료 리스트의 정확성을 판단**하는 것이다. 그리고 테스트 내부에서 수행하는 action의 경우에는 나중에 실패 분류를 쉽게 하기 위해서 **최소화하는 것**이 좋고 여기서는 1번이라고 표시된 부분처럼 Berry Blue를 선택하는 것이 예시가 될 수 있다. 

따라서 해당 Berry Blue를 선택하게 되면, 2번 부분에서 레시피인 재료 리스트를 가져오게 되고 이를 action의 결과로서 검증할 수 있다. 

![4](/assets/images/WWDC20WriteTestForFailure/4.png)

이때 `result bundle` 에서는 **테스트명을 구체적으로 작성해주어서 어떠한 테스트가 검증되었는지 쉽게 눈으로 확인**할 수 있다. 



⚠️  `result bundle` 을 보는 방법은 아래의 그림처럼 Xcode 메뉴에서 `View → Navigatiors → Report` 를 누르거나 Navigator에서 가장 오른쪽에 있는 report 버튼을 누르면 By group, By Time 으로 정렬해서 볼 수 있습니다. 

![5](/assets/images/WWDC20WriteTestForFailure/5.png)



#### 💡Tips - enum의 사용 

네이밍에 대해서 이야기 해보자면, UI 요소들의 네이밍은 자주 바뀌게 된다. 따라서 잦은 네이밍의 변경에 대처하기 위해서 **모든 String 값을 enum으로 관리하는 것을 추천**한다. 

![6](/assets/images/WWDC20WriteTestForFailure/6.png)

위와 같이 enum으로 모든 String을 관리한다면, **UI가 변경되었을 때 쉽고 빠르게 대처가 가능**하다. 또한 아래와 같이 알아차리기 어려운 **spelling 실수와 이로 인한 테스트 실패를 횟수를 줄일 수 있다.** 

![7](/assets/images/WWDC20WriteTestForFailure/7.png)



#### 💡Tips - Factor common code

이와 같이 오류를 최소화하는 또 다른 방법은 **공통되는 코드를 helper function에 반영하여, 여러 테스트에서 동일한 코드 경로를 사용할 수 있도록 하는 것**이다. 

![8](/assets/images/WWDC20WriteTestForFailure/8.png)

`let recipe = try app.smoothieList().selectRecipe(smoothie: .berryBlue)` 코드를 보면 실제로 스무디 종류에 따라서 계속 반복해서 `smoothieList()` 에 접근하여 레시피를 선택할 것이다. 따라서 이렇게 **공통적인 테스트 경로를 테스트 할 때 마다 중복해서 작성하는 것보다는** `SmoothieList` 라는 클래스를 만들어 내부 `selectRecipe()` 와 같은 함수를 구현해, **여러 테스트에서 동일한 코드 경로를 사용할 수 있도록 하는 것**이다. 

#### 💡Tips - model domain of App

또 다른 방법은 **앱 도메인을 모델링하고, 해당 도메인에서 테스트 언어를 설계**하는 것이다. 이렇게 하면 테스트는 앱의 언어를 반영하게 된다. 

![9](/assets/images/WWDC20WriteTestForFailure/9.png)

이 Fruta 앱에서 `SmoothieList` 를 요청할 수 있고, `SmoothieList` 내부에서 Recipe UI 요소들을 반환하는 `selectRecipe()`와 같은 작업을 수행할 수 있다. 이러한 Recipe UI 요소들은 앱과 요소들을 lower level에서 추적하기 위해서 생성한 `FrutaUIElement` 클래스를 기반으로 한다. 

위와 같은 방식으로 **object-oriented (객체 지향적)으로 공유 코드를 생성**한다. 테스트는 요소와 쿼리를 기반으로하는 매우 기능적인 것으로 취급되지만, 가독성 측면에서 이를 객체 지향 환경으로 시뮬레이션할 수 있다. 

![10](/assets/images/WWDC20WriteTestForFailure/10.png)

이렇게 하면 테스트를 앱을 마치 자신의 생각을 보여주는 일련의 subview처럼 테스트에서 호출할 수 있게 해준다. 
따라서 이러한 모델링을 하게되면 **아래의 그림과 같이 각 요소의 계층 구조를 줄이고, 해당 요소의 하위 요소에만 쿼리를 할 수 있게 된다.** 

![11](/assets/images/WWDC20WriteTestForFailure/11.png)



#### 💡Tips - framework, Swift package를 사용해 testing code 공유

수년 간 이런식으로 shared testing 코드를 만들다 보니까 규모가 많이 커지게 되었고, 이를 해결하기 위해서 `shared testing code`를 **product code로 취급**하게 되고 **이를 테스트를 위한 shared framework로 생성**한다. 만약 여러 앱들 사이에서 testing code를 사용해야하는 경우라면 **Swift package를 사용해서 testing code를 공유**할 수도 있다. 


#### 요약

1. 구체적인 목표를 가지도록 테스트를 설계하고 이는 테스트 함수명에 반영되어야함
2. enum 사용해서 UI change에 간단하고 빠르게 대처
3. factor common code를 helper function에 포함시켜서 UI change에 간단하고 빠르게 대처
4. 앱의 UI 계층을 반영하기 위해서 modeling을 진행함
5. framework나 Swift package를 사용해서 프로젝트 사이에 test code를 공유



### 3️⃣ Test: Assertion

실제로 테스트에 있어서 **심장과 같이 중요한 작업을 하는 곳**이다. 지금부터는 테스트 실패를 쉽게 분류할 수 있는 **test assertion**과 **error handling**을 하는 방법에 대해서 알아보자. 



#### 💡Tips - Add Assertion Messages

![12](/assets/images/WWDC20WriteTestForFailure/12.png)

다음과 같이 `XCTAssertEqual()` 를 사용할 때, 실패하게 되면 optional message가 등장하게 된다. 지금과 같이 간단하게 몇개만 테스트를 하는 것이라면 여기서 3과 2가 무엇인지 알 수 있지만, 만약 result bundle에서 이를 확인한다면 수많은 테스트에서 3과 2가 무엇을 의미하는지 모를 것이다. 

따라서 **아래와 같이 단서를 제공할 수 있도록 문장을 추가하여 메시지를 작성**하는 것이 좋다. 그러나 종종 **자동 시스템에서 assertion 실패를 읽게 되는데**, 이러한 경우에는 **message가 구체적이지만 너무 구체적이지 않는 것이 좋다.** 그러므로 동일한 이유로 실패한 여러 테스트를 인식하는데 assertion message를 사용할 수 있도록 **날짜나 타임 스탬프 또는 고유한 파일 경로와 같은 항목을 생략**하는 것이 좋다. 



#### 💡Tips - Use relevant `XCTAssert*` function

즉, 테스트에서 달성하려는 것에 대한 **관련된 assertion 함수를 사용하는지** 확인해 볼 필요가 있다. 이를 통해서 **오류가 발생하여 테스트가 실패하여 표시되는 자동 메시지는 더욱 관련성이 높게** 된다. Xcode12에서는 실패를 보고하는 새로운 low-level 방식인 `XCTIssue` 가 추가되었다. 이에 대한 내용은 WWDC Triage Test Failures with XCTIssue 를 참고하면 된다. 



#### 💡Tips - Use waitForExistence for asynchronous event

테스트에 있어서 가장 문제가 되었던 것들 중 하나는 바로 **비동적인 이벤트**들이다. 
예를 들어 아래와 같이 레시피 버튼을 누르게 되면, 내부에서 코드에 따라서 시간이 걸릴 수도 있다. 

![13](/assets/images/WWDC20WriteTestForFailure/13.png)

따라서 recipe element를 바로 리턴하게 되면 존재하지 않을 수도 있다. 예전에는 이러한 것을 `sleep` 을 사용하였고, 이러한 방법은 다소 시간이 소요되었다. 이를 해결하는 방법 중에 하나는 **`waitForExistence` 를 시간초과(timeout)과 함께 사용**하는 것이다. `waitForExistence` 는 만약 timeout보다 기대 시간이 빠르게 충족된다면, 즉 **제한 시간보다 빠르게 결과가 반환된다면 남은 대기하는 시간을 절약할 수 있게 된다.** 

![14](/assets/images/WWDC20WriteTestForFailure/14.png)

또한 자신이 설계한 환경에서 테스트를 통과하거나 통과하지 못할 수도 있고, 이는 위의 그림과 같이 result bundle에서 Ingredients View를 찾기 위해서 5초동안 기다렸다는 것을 확인할 수 있다. 



#### 💡Tips - Unwrapping Optionals

아래와 같이 String 배열에서 favorites의 개수를 반환하려는 함수가 있는데 보다시피 optional을 unwrap 하는데 시간을 사용하지 않고, **바로 강제로 추출**하도록 작성을 하였다. 

![15](/assets/images/WWDC20WriteTestForFailure/15.png)

실제로 이를 Xcode에서 실행시키면 **충돌이 발생하게 되고 error로 인해서 테스트가 중단**되게 된다. 

![16](/assets/images/WWDC20WriteTestForFailure/16.png)

지속적인 통합 환경에서 이러한 문제가 발생하게 되면 result bundle에는 테스트가 중단되었고 신호가 불량이라는 내용을 테스트 실패와 함께 받게된다. 

따라서 이러한 문제를 해결하기 위해서는 **optional을 확실하게 unwrap 해주어야한다.** 

![17](/assets/images/WWDC20WriteTestForFailure/17.png)

Unwarp을 해주는 방법은 위와 같이 4가지 방법이 있다. 

1. `if let` 을 사용해서 optional unwrapping
2. `guard let` 을 사용해서 optional unwrapping과 error throw 
3. `nil coalescing operator` (nil 병합 연산자)를 사용해서 nill인 경우 `??` 뒤의 default 값을 사용 
4. XCTest framework에서 제공하는 `XCTUnwrap` 을 사용 → 테스트 결과가 nil인 경우 error throw를 하는 guard let을 단순화한것

따라서 `XCTUnwrap` 을 사용하면 result bundle에서 자동 생성된 메시지와 함께 작성한 메시지가 보여지게 된다. 

`unwrapping optional` 을 가장 좋은 부분은 **충돌로 인한 종료가 아니라 우아하게 실패하고 자신이 작성한 `tearDown()`을 호출**한다는 것이다. 



#### 💡Tips - Throwing Error from shared code

테스트를 진행할 때 발표자는 **shared code에서는 assertion을 사용하는 것 대신에 error를 throw**한다고 한다. 왜냐하면 shared code는 많은 테스트에서 실행중이고, 이러한 테스트 중에서 일부 테스트에서는 숨겨진 것이 표시되지는 않는지, error dialog는 나타나는지에 대해 확인하기 위해서 **일부로 실패하는 테스트를 작성하는 경우도 있기 때문이다.** 

![18](/assets/images/WWDC20WriteTestForFailure/18.png)

따라서 위와 같이 ingredients를 검증하는 `verify()` 라는 shared method가 있다고 할 때, 이전에 보여줬던 ingredients가 남아있는지에 대해서 테스트를 진행하게 된다. 그리고 만약 남아있다면 error를 던지게 되는데, Error type인 `RecipeError` 를 보면 `CustomStringConvertible` 프로토콜의 요구 사항인 오류 설명에 표시할 값을 전달하는 경우가 많다. 이를 사용하므로서 result bundle에서 **더욱 상황과 관련된 에러 문구가 작성**되게 된다. 

![19](/assets/images/WWDC20WriteTestForFailure/19.png)

만약 local에서 실패를 분류한다고 하면 Xcode 12의 새로운 기능인 코드인 오류의 역추적을 통해서 직접 확인할 수 있으므로 shared code에서의 오류가 실제로 어디에서 발생했는지 더 이상 찾지 않아도 된다. 또한 아래와 같이 Runtime Issues Navigator와 result bundle에서도 이를 확인할 수 있다. 

![20](/assets/images/WWDC20WriteTestForFailure/20.png)



#### 💡Tips - Use `XCTContext.runActivity()` and attachments

또한 result bundle에는 사용자가 읽을 수 있는 disclosure group이 포함되어 있는데, 이러한 그룹은 당시 어떤 테스트를 진행했는지에 대해서 context를 제공한다. 따라서 아래의 result bundle을 보면 Blue berry smothie의 ingredient 에서 Grape를 찾고 있는 것을 알 수 있다. 그리고 이러한 식으로 **context를 제공하기 위해서 `XCTContext` 를 사용**한다. 이는 아래 그림에서 노란색 박스와 같이 해당 블록에서 수행된 작업과 함께 result bundle에 표시되게 된다. 이를 통해서 result bundel에 조직과 컨텍스트를 추가하여 **테스트에 따라 쉽게 읽을 수 있도록 할 수 있다.** 

![21](/assets/images/WWDC20WriteTestForFailure/21.png)

runActivity를 할 때 실행할 수 있는 또 다른 하나는 바로 **`XCTAttachment` 를 사용하는 것**이다 위의 그림에서 주황색 박스로 표시된 코드 부분을 throw 전에 삽입하게되면, result bundle에 주황색박스와 같이 첨부 파일이 추가되는 것을 확인해 볼 수 있다. 이때 파일, 이미지, 데이터와 같은 첨부파일을 `XCTContext` 또는 test case에 추가하면 번들로 표시되게 된다. 이는 **실패한 테스트에 대한 추가적인 log를 수집할 수 있는 좋은 방법**이며 **특히 CI 시스셈에서 테스트를 수행하는 경우 좋은 방법**이다. 앞에서 **assert comment 로 파일 경로를 추가하는 것은 좋지 않다고 했지만, 이 방법을 통해서 파일 경로와 파일 자체를 첨부**할 수 있기 때문이다. 

이렇게 하면 나중에 실패를 보다 쉽게 분류할 수 있다. 제품 실패 분류에 필요한 모든 데이터는 나중에는 사용할 수 없을 수도 있으므로 위와 같이 테스트를 통해서 수집되어야한다. 



#### 💡Tips - XCTSkip

때로는 **테스트는 아예 실행되지 않아도 되는 경우**가 있다. 

이러한 경우 **선택적인 메시지를 추가하여 실행되지 않는 테스트를 문서화**하는데 `XCTSkipUnless()`, `XCTSkipIf()` 를 사용한다. 

![22](/assets/images/WWDC20WriteTestForFailure/22.png)

1. 테스트가 실행 중인 플랫폼과 관련이 없는 테스트라면 이를 건너뛰는 것
2. **테스트 stubb out** 으로 사용하여 어떠한 테스트가 구현되지 않았는지와 어떤 테스트가 복귀되었는지 확인
3. 여러 이유로 현재 해결할 수 없는 테스트가 있을 수 있으며, 이때 이러한 테스트 실패에 대한 오류는 분류하고 싶지 않지만 그래도 테스트는 계속 진행하여 테스트 트랙을 놓지고 싶지 않은 경우

![23](/assets/images/WWDC20WriteTestForFailure/23.png)

또한 `XCTSkip` 을 사용하면 result bundle에서 건너뛴 테스트를 계속해서 볼 수 있으므로, 문제를 해결하였다면 테스트를 작성하거나 수정하면 된다. 



#### 요약

1. Asssertion message의 추가
2. 관련된 `XCTAssert*` 기능을 사용하여 result bundle 오류에 컨텍스트를 추가
3. `waitForExistence` 를 사용하여 sleep 대신에 비동기 이벤트나 시간관련되 이슈를 처리
4. optional을 unwrap하여 테스트가 중간에 실패하지 않도록 한다
5. share code에서는 실패하는 테스트에 대비하여 assertion을 발생시키기 보다는 error를 던져 에러를 잡을 수 있도록함
6. `XCTContext.runActivitiy` 와 `XCTAttachment` 사용하여 result bundle에 컨텍스트 및 컨텐츠를 첨부 
7. 현재 시나리오에서 실행되지 않을 것으로 예상되는 테스트에는 `XCTSkip` 사용



### 4️⃣ tear down

마지막으로 해야할 것은 다음과 같다. 

#### 요약

1. **`teardownWithError`  의 사용** <br>
   새로운 오류 관리에 있어서의 이점을 챙기지 위해서 throw를 하므로 이를 사용하는 것이 좋다. 
2. teardown 함수안에서 **실패에 대한 일부 분석을 포함하여 추가적으로 log를 수집**
3. setup에서 설정했던 내용들 중에서 **변경이 일어났던 부분을 reset**




### 전체 내용 요약

1. `setup` : 테스트에 필요한 환경과 가정들을 설정하는 부분
2. `test action ` : 자신의 앱을 model로 한 shared code를 통해서 테스트하고자하는 필요한 작업을 수행하는 부분
3. `test assertion` : helper method, errors, test assetion을 통해서 위의 작업이 제대로 수행되었는지 검증하는 부분
4. `tear down` : 테스트를 마치고 data를 수집하고 정리하는 부분 



이러한 기술들을 통해서 테스트를 보다 견고하게 수행하고 프로젝트의 문제를 쉽고 빠르게 분류하여 테스트 환경 친화적인 제품을 제공할 수 있도록 하자. 



## 참고자료

1. [Writing tests to fail - WWDC 2020](https://developer.apple.com/videos/play/wwdc2020/10091/?time=996)





