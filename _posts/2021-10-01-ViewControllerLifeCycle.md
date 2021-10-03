---
title:  "[iOS] ViewController life cycle"
excerpt: "ViewController의 생명주기에 대해서 알아보자"

categories:
  - iOS
tags:
  - []
last_modified_at: 2021.10.01
--- 

### 들어가며
![1](/assets/images/ViewControllerLifecycle/1.png)

ViewController의 생명주기는 위의 그림과 같다. <br>
대부분의 iOS 앱들은 다수의 ViewController들로 이루어져있다. <br>
물론 하나인 앱도 있긴하다. 이때 ViewController들의 생명주기에 대해서 알아보자. 

### 1️⃣ init

#### nib, xib
먼저 init에 대해서 알기전에 아래 용어들에 대해서 알아보자. 

`nib` 은 **NeXT Interface Builder**의 약자이다. 그리고 binary로 되어있다. 

`xib` 는 **XML Interface Builder**의 약자이다. text 기반으로 되어있다. 

`.xib` 파일은 빌드 시점에 `.nib` 파일의 형태로 바뀐다. 즉, **xib가 컴파일 되면 nib가 되는 것이다.** 인터페이스 빌더로 작업한 UI는 **`.xib` 파일의 형태로 저장되고 빌드 시점에 앱 번들로 복사되고 런타임에 로드**된다. 

`.xib` 파일은 텍스트 기반의 파일로 `.nib` 파일보다 소스 컨트롤에 용이하며 읽기에 용이하다.

#### nib Name, nib File
`nib 파일` 은 **인터페이스 빌더에서 생성한 객체들을 직렬화하여 저장하는 파일**로, UI를 구성하는 객체들(편의상 인터페이스 객체라 부르겠음)을 저장하게 된다. 이 파일에는 인터페이스 빌더를 통해 추가한 인터페이스 객체들(창, 뷰, 버튼 컨트롤 등)과 이러한 객체들의 세부 설정(스타일, 색상, 폰트 등), 그리고 객체들 간의 연결(connection)정보가 모두 포함된다. 

`nib Name` 은 (interface builder에서 지정했다면)지정 되어 있는 **view controller의 nib file이름**이다. 

#### init(nibName:bundle:)

> "Creates a view controller with the nib file in the specified bundle."
> bundle에 있는 nib file로 **viewcontroller를 생성하여 반환**하는 함수
> 즉, storyboard가 아닌 .nib 파일로 관리되는 경우 init(coder:) 대신에 위의 메소드를 초기화 용도로 사용 가능

UIViewController의 **지정 이니셜라이저**(반드시 구현이 되야하는)이다. 

⚠️ 실제로 init(nibName:bundle:) 생성자는 **코드로 nib 파일을 만든 뷰**를 초기화할 때 사용하는 생성자이다. 따라서 storyboard, .xib로 생성한 경우에는 해당 메소드가 아닌 `init?(coder:)` 를 통해서 초기화가 된다. 

#### init?(coder:)

> **스토리보드나 xib를 통해** View Controller, View를 만드는 경우 View controller, View의 객체가 생성될 때 마다 초기화 작업을 하는 메서드

`xib` 파일로 만든 뷰는 실제로 저장될 때 **아카이빙** 되어 저장된다. 따라서 해당 뷰를 불러올 때는 **언아카이빙을** 통해 불러와야한다.

스트리보드도 내부적으로는 일종의 xib 파일들로 이루어져 있기 때문에 `init?(coder:)` 가 호출된다. 그러나 이 시점에서는 `@IBOutlet` `@IBAction` 은 준비되어 있지 않다.


#### NSCoding 프로토콜

서브 클래스에서 init을 작성하게 되면 무조건 `init?(coder:)`를 구현하라고 컴파일 에러가 뜨게 되는데 그 이유는 바로 **NSCoding 프로토콜 때문**이다. 

NSCoding 프로토콜은 이를 채택하는 클래스로 하여금 실패가능한 이니셜라이져를 작성하도록 한다. 

![2](/assets/images/ViewControllerLifecycle/2.png)

프로토콜에 명세된 이니셜라이져를 구현하면 **required** 키워드가 붙게된다. required 키워드가 붙는 이니셜라이져는 **상속받는 자식 클래스에서도 반드시 구현**을 해주어야한다. 

따라서 UIView, UIViewController는 **NSCoding 프로토콜을 채택**하고 있으므로, 이를 상속받는 클래스는에서는 `required init?(coder:)` 를 구현해주어야하는 것이다. 

⚠️ 그렇다면 여기서 왜 컴파일러는 `required init?(coder)` 는 **새로운 지정 이니셜라이저를 작성할 때만 구현하라고 하는 것인가**?

Swift에서는 자식 클래스에서 지정 이니셜라이저를 따로 작성하지 않는 경우, 부모의 이니셜라이저를 자동으로 상속한다. 따라서 `required init?(coder)`또한 상속되어 오류가 나오지 않는다.

반면에 자식 클래스에서 이니셜라이저를 따로 작성하게 되면, **부모의 이니셜라이저들이 자동으로 상속되지 않아서** `required init?(coder)` 을 구현하라는 문구가 나오는 것이다. 

#### awakeFromNib()

> "Prepares the receiver for service after it has been loaded from an Interface Builder archive, or nib file."

Interface builder archive 또는 nib 파일이 **생성된 후 초기화 작업을 준비**하는 곳이다. Instance가 만들어진 후에 호출되며 IBOutlet, IBAction 스토리보드의 View가 모두 바인딩 된다. 따라서 awakeFromNib가 호출되는 시점에는 IBAction과 IBOutlet이 연결되어 있으므로 이들은 nil이 아님을 보장한다.

이 메소드는 `init?(coder:)`를 통해 **뷰가 모두 언아카이빙된 후 호출된다.** `@IBOulet`과 `@IBAction`이 모두 자리가 잡힌 후 호출되는 것이다. `init?(coder:)`가 언아카이빙의 시작점이라면 `awakeFromNib()`은 끝나는 시점이라고 할 수 있다.

⭐️ **xib 혹은 storyboard를 통해서 view 객체를 생성하는 경우**에는 관련 init 메서드가 호출이 되지 않는다. 대신에 `awakeFromNib()` 가 호출되므로 이 안에서 상황에 맞게 초기화를 진행하면 된다. 

반면에 **코드로만 view를 작성하는 경우**, 즉 storyboard나 nib, xib로 만든것이 아닌 경우에는 **awakeFromNib()은 호출되지 않는다.** 대신에 `init(frame:)` , `init(style: UITableViewCell.CellStyle, reuseIdentifier: String?)` 와 같은 생성자 메서드를 사용해서 내부에서 초기화를 진행하면 된다. 그리고 init()을 구현하게 되면 NSCoding을 채택했기 때문에 required init?()도 필수적으로 구현해 주어야한다. 

호출 순서
- nib을 사용하는 경우
init → initWithCoder → awakeFromNib
- nib을 사용하지 않는 경우
init → initWithFrame

##### 의문점
⚠️ 그렇다면 view controller에서는 왜 awakeFromNib()이이 호출되지 않을까?
위에서 분명 storyboard도 일종의 xib로 이뤄져있기 때문에 `init?(coder:)` 를 통해서 초기화된다고 하였다. 
```swift
class ViewController: UIViewController {
    @IBOutlet weak var button: UIButton!

    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        print("1st Viewcontroller init()")
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        print("1st Viewcontroller required init()")
    }
    
    override class func awakeFromNib() {
        super.awakeFromNib()
        print("1st Viewcontroller awakeFromNib")
    }
    
    override func loadView() {
        super.loadView()
        print("1st ViewController loadView()")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        print("1st ViewController viewDidLoad()")
    }
}
```
그렇다면 위에서 `init?()` 이 호출된 이후에 awakeFromNib이 호출되어야할 것 같은데 breakpoint를 걸어놔도 awakeFromNib은 호출이 되지 않는다. 

그 이유는 내가 바보여서다... 당연히 불려야하고 불리는 것이 맞다.
```swift
    override class func awakeFromNib() {
        super.awakeFromNib()
        print("1st Viewcontroller awakeFromNib")
    }

    override func awakeFromNib() {
        super.awakeFromNib()
        print("1st Viewcontroller awakeFromNib")
    }
```
`awakeFromNib`을 치면 자동완성에 이렇게 2가지 함수가 나타나게 된다. 그러나 NSObject의 extension으로 구현되어있는 `awakeFromNib()` 의 경우에는 class 키워드가 붙지 않았으므로, class 키워드가 없는 `awakeFromNib()` 를 호출해야지만 제대로 override가 되는 것이다.

왜 class func awakeFromNib()을 추천해주는거냐.. 엑코야... 하... 다음과 같이 실험을 해봤는데 원래대로라면 컴파일러 에러가 떠야하는데 안뜨는 것을 보면 무엇인가 내부에 구현이 되어있는 건가 싶은데 무엇인지는 모르겠다. 

```swift
class A {
    open func hi() {
        print("Ahi")
    }
}

class B: A {
    override func hi() {
        print("Bhi")
    }
    
    // 컴파일 에러 발생 ⚠️
    // Method does not override any method from its superclass
    override class func hi() {
        print("class Bhi") 
    }
}
```


##### 의문점을 찾다가 알게 된 사실
awakeFromNib이 호출되는 시점에서는 IBOutlet 요소들이 연결된다고 나와있지만, 그렇지 않은 경우가 있었고 이는 **viewcontroller와 view hierarchy는 서로 다른 nib 파일에서 런타임에 로드되기 때문**이라고 한다. 자세한 내용은 이 [stack overflow](https://stackoverflow.com/questions/17400547/awakefromnib-outlets-and-storyboards-is-the-documentation-wrong)를 확인해보면 된다. 


### 2️⃣ loadView

> 화면에 띄어질 View를 만드는 메소드

storyboard나 .nib 파일로 만들어지는 경우가 아니라 모두 직접적으로 코딩하여 만드는 경우를 제외하고서는 override하지 않는 것이 좋다.


### 3️⃣ viewDidLoad

>  "Called after the controller's view is loaded into memory" <br>
>  뷰의 컨트롤러가 **메모리에 로드되고 난 후에 호출**된다. 

즉, 뷰의 로딩이 완료되었을 때 **시스템에 의해서 자동으로 호출**된다. 일반적으로 리소스를 초기화하거나, 초기화면을 구성하는 용도로 사용한다.

⭐️ 화면이 **처음 만들어질 때 한 번만 실행**되므로, 처음 한 번만 실행해야하는 초기화 코드가 있는 경우 해당 메소드 내부에서 작성


#### viewDidLoad는 한 번만 호출될까?

⚠️ 과연 viewDidLoad는 한 번만 호출이 되는 것이 맞는 것일까?

> 이에 대한 정답은 생에서는 오로지 한 번만 호출이 되는 것이 맞다. 

⚠️ 그러나 이와 같은 상황에서, 즉 **navigationController의 root view controller**가 아니라면 `viewDidLoad` 가 여러 번 호출될 수 있다. 

왜냐하면 navigationController는 stack처럼 view가 push, pop되는데 

첫 번째인 root viewcontroller가 아닌 viewcontroller라면 다음과 같은 상황이 벌어진다.

```swift
// push하는 경우
Root viewcontroller → 2 번째 viewcontroller

// pop하는 경우 
// 2번째 viewcontroller는 deinit되어버린다.
Root viewcontroller 
```
즉, **스택에서 pop이 된 데이터는 메모리에서 사라지게 된다.** 즉, 해당 생이 마무리 되는 것이다.  

따라서 pop 했다가 다시 push를 하게 되면, 2번째 viewcontroller는 다시 새롭게 메모리에 로드가 되고 `viewDidLoad` 가 호출되게 되는 것이다. 아예 메모리에서 해제되었다가 다시 새로운 생을 시작하므로 `viewDidLoad` 가 다시 불리는 것이다. 

#### IBOutlet의 초기화 시점

⚠️ @IBOutlet 프로퍼티들은 **viewDidLoad 전까지는 전부다 nil**이기 때문에, **viewDidLoad에서부터 유효**하다!


### 4️⃣ viewWillAppear

> "Notifies the view controller that its view is about to be added to a view hierarchy." <br>
> 뷰가 **화면(뷰) 계층에 추가되기 직전**에 호출 된다

즉, 뷰가 나타나기 직전에 호출이 된다. **다른 뷰로 갔다가 다시 돌아오는 상황에서 해주고 싶은 처리**가 있는 경우 viewWillAppear에서 구현해주면 된다.

⭐️ `viewDidLoad` 는 한 번만 호출이 되지만, `viewWillAppear` 은 **뷰가 나타날 때마다 호출이 된다.**

### 5️⃣ viewDidAppear

> "Notifies the view controller that its view was added to a view hierarchy." <br>
> **뷰가 뷰계층에 올라갔다는 것**(= 뷰가 나타났다는 것)을 컨트롤러에게 알려주는 역할 

즉, **뷰가 화면에 나타난 직후에 실행**

⭐️ 화면에 **적용될 애니메이션을 그려준다.**

### 6️⃣ viewWillDisappear

> "Notifies the view controller that its view is about to be removed from a view hierarchy." <br>
> 뷰 계층에서 뷰가 막 사라지기 직전에 호출되는 함수

뷰가 삭제 되려고 하고 있는 것을 뷰 컨트롤러에 통지

### 7️⃣ viewDidDisappear

> "Notifies the view controller that its view was removed from a view hierarchy." <br>
> 뷰 계층에서 뷰가 제거되었음을 뷰 컨트롤러에게 알려줌


### ~~8️⃣ viewDidUnload~~

> "Called when the controller's view is released from memory. " <br>
> 컨트롤러의 뷰가 메모리로부터 해제되는 시점에 호출된다.

⚠️ Deprecated 되었다. 

이 메소드가 호출되는 시점에 이미 view는 nil이다.


### 8️⃣ Deinit

> View controller가 메모리에서 사라지기 전 이 메소드가 호출된다

이 메소드를 할당 받은 자원 중 ARC에 의해 해지가 불가능한 자원들을 해제하기 위해 `override` 할 수 있다. 또한 백그라운드에서 돌리기 위해 이전의 메소드에서 멈추지 못하였던 행위들을 이 메소드 내에서 멈출 수 있다.

⚠️ 그러나 View Controller가 **화면에서 사라지는 것이 메모리에서 해지된다는 것을 의미하지 않는다**는 것을 명심해야한다.

즉 화면에서 사라진다고 메모리에서 해지되는 것은 아니다.

많은 Container View Controller들이 그들의 View Controller들을 메모리에서 유지하고 있기 때문이다.

e.g. Navigation Controller의 경우에는 stack 구조로 push 되어 있던 viewcontroller들은 여전히 메모리에 남아있고, back하는 경우에만 메모리에서 해지됨

이러한 이유로 여러분은 화면에서 사라진 View Controller들이 정상적으로 작동하고 여전히 notification 을 받을 수 있다는 것을 명심해야한다.


### 호출 순서

2개의 뷰가 있을 때 호출되는 순서

```swift
1st ViewController required init
1st ViewController loadView
1st ViewController viewDidLoad
1st ViewController viewWillAppear
1st ViewController viewDidAppear

(2번째 viewController로의 이동하는 버튼클릭)

-----------------------------------------------------

2nd ViewController required init
2nd ViewController loadView
2nd ViewController viewDidLoad
⚠️ 주의해서 봐야하는 구간
1st ViewController viewWillDisappear
2nd ViewController viewWillAppear
1st ViewController viewDidDisappear
2nd Viewcontroller viewDidAppear
```
위에서 실행 결과를 보듯이 1st 뷰컨트롤러의 viewWillDisappear이후에 viewDidDisapper가 바로 호출되는 것이 아니라,
2nd 뷰컨트롤러의 viewWillAppear이 호출된 이후에 1st viewcontroller viewDidDisappear, 2nd viewcontroller viewDidAppear가 호출된다.

### 마치며
ViewController의 생명주기에 대해서 공부해보았다.

특히 init 부분에서 잘 모르고 그냥 사용하던 부분들이 있었는데 이번 기회에 알게 되어서 기쁘다 ㅎㅎ

### 참고 
[apple Document loadView](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview) <br>
[apple Document viewDidLoad](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621495-viewdidload) <br>
[apple Document viewWillAppear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621510-viewwillappear) <br>
[apple Document viewDidAppear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621423-viewdidappear) <br>
[apple Document viewWillDisappear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621485-viewwilldisappear) <br>
[apple Document viewDidDisappear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621477-viewdiddisappear) <br>
[awakeFromNib의 용도](http://codershigh.dscloud.biz:30004/t/awakefromnib/107) <br>
[loadView와 viewDidLoad의 차이](https://yagom.net/forums/topic/loadview와-viewdidload-차이에-대한-질문입니다/)<br>
