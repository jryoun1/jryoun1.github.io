---
title:  "[Swift] ScrollView"
excerpt: "Swift에서 ScrollView를 사용하는 방법에 대해서 알아보도록 하자"

categories:
  - Swift
tags:
  - [ScrollView]
toc: true
toc_sticky: true
last_modified_at: 2020.12.27
---
## 들어가며
`UIScrollView`는 iOS에서 다양하고 유용한 컨트롤 중 하나이다. <br> `UITableView` , `UICollectionView` 의 기본이라고 할 수 있고, 기기의 정해진 화면보다 큰 콘텐츠를 제공할 수 있는 방법이다. 

## ScrollView
먼저 Apple의 Auto Layout Guide 문서에 나와있는 Working with ScrollView라는 파트를 같이 보자.

![](https://images.velog.io/images/minni/post/ce820342-f7b9-4719-a2eb-9eb4d8646cd0/image.png) 

공식문서에 나와있는 내용에 따르면 ScrollView를 사용할 때에 우리는 superview내에서 ScrollView frame의 size와 location 그리고 ScrollView의 내용 영역인 contents area의 크기를 모두 정의해주어야 한다고 나와있다. 

역시 문서만 봐서는 무슨 내용인지 와닫지 않으므로 한 번 직접 스토리보드에 scrollview를 그려보자.

![](https://images.velog.io/images/minni/post/0712291c-1c18-4e93-b74a-0028fced9baa/image.png) 

ViewController 위에다가 ScrollView를 올려놓고 constraints를 safeArea에서 0씩 주었는데 위의 그림과 같이 빨간색 줄과 함께 ScrollView의 content height와 width가 모호하다는 에러가 발생하게 된다. 

분명 다른 Objects들과는 다르게 왜 제약을 설정했는데도 모호하다는 것일까?
<br>
이에 대한 해답은 위에 나와있다 <br>
공식문서에서 보았듯이 ScrollView는 **ScrollView의 frame size, location** 뿐만 아니라 ScrollView에 들어갈 내용, **즉 contents area의 크기까지 정의해주어야 한다는 것이다.**

위에서 ScrollView의 frame size와 location은 constraint를 통해서 정해진 것 같지만 **실제로 ScrollView 안에 들어갈 contents의 size가 지정되지 않아서 ScrollView는 자신의 size와 location을 알지 못해서 빨간줄이 생겼던 것이다.** 

## Interface Builder로 ScrollView 만들기
Apple에서는 이러한 ScrollView안에 포함되는 content에 대해서 dummy view나 layout group에 포함시키면 쉽게 ScrollView를 사용할 수 있다고 한다. 

아래의 순서를 따라서 인터페이스 빌더에서 ScrollView를 그려보자. 

**1. Add the ScrollView to the scene** <br>
먼저 View Controller에 ScrollView를 추가한다. 

**2. Draw constraints to define the ScrollView's size and position, as normal.**<br>
화면에 추가된 ScrollView 자체의 사이즈와 위치를 constraints를 통해서 정해준다. 
1번과 2번까지는 위에서 화면에 ScrollView 넣고 제약을 0으로 준 것이다.

**3. Add a view to the ScrollView. Set the view's Xcode specific label to Content View.** <br>
![](https://images.velog.io/images/minni/post/b7efcde9-e335-4fe1-804a-b13c59aeccba/image.png) 

ScrollView안에 view를 넣어주고 Content View라고 이름을 변경해준다. 
이때 view의 label은 꼭 변경해야하는 것은 아니지만 헷갈림을 방지하기 위해서 바꿔준다고 생각하면 된다. 

**4. Pin the content view's top, bottom, leading and trailing edges to the ScrollView's corresponding edges. The content view now defines the ScrollView's content area.**<br>
![](https://images.velog.io/images/minni/post/73c3e716-09aa-4719-9b82-c40511652e65/image.png) 

Content View를 Superview인 ScrollView의 top, bottom, leading, trailing에 맞춰 constraint를 설정한다.

**5. (Optional) To disable horizontal scrolling, set the content view's width equal to the ScrollView's width. The content view now fills the ScrollView horizontally.**<br>
만약 **세로로만 스크롤을 적용**하고 싶은 경우에는 **ScrollView의 width를 content view의 width랑 동일하게 설정**해주면 된다. 그럼 이제 해당 ScrollView는 세로로만 스크롤이 가능하게 된다. 

![](https://images.velog.io/images/minni/post/52db0155-1db8-4570-83ef-9fef723ec331/image.png) 

이번 예시에서는 세로로만 스크롤이 가능하도록 할 예정이니 위의 그림과 같이 Content View의 width를 ScrollView의 width와 동일하게 설정해준다. 

이제 width는 크기가 고정되어 파란색으로 표시되는 것을 확인할 수 있다. 
아직 ScrollView와 Content View의 세로의 크기가 정해지지 않아 빨간색인 것을 볼 수 있다.

**6. (Optional) To disable vertical scrolling, set the content view's height equal to the ScrollView's height. The content view now fills the ScrollView vertically.**<br>
만약 **가로로만 스크롤을 적용**하고 싶은 경우에는 **ScrollView의 height를 content view의 height와 동일하게 설정**해주면 된다. 그럼 이제 해당 ScrollView는 가로로만 스크롤이 가능하게 된다. 

**7. Lay out the ScrollView’s content inside the content view. Use constraints to position the content inside the content view as normal.**<br>
이제 해당 ScrollView의 content인 content view안에 원하는 내용들을 lay out 해주고 제약을 걸어주면 ScrollView를 사용할 수 있다. 

![](https://images.velog.io/images/minni/post/122d4311-be82-4f80-afbf-5f548739a636/image.png) 

Label을 하나 넣고 top, bottom, leading의 제약을 준 결과 Label이 content의 전부인 ScrollView가 완성이 된 것이다. 

![](https://images.velog.io/images/minni/post/7217c333-5f44-46ee-8d4e-de5ea60e12a5/image.png)

## contentLayoutGuide
`contentLayoutGuide`에 대해서 알아보자. <br>
먼저 공식 문서를 보면 다음과 같다. 
- ScrollView의 content 영역과 관련된 오토 레이아웃 제약 조건을 만드려면 이 레이아웃을 사용하라고 나와있다. 
- 즉, ScrollView의 **변환되지 않는 content 사각형을 기반으로 하는 레이아웃 가이드**이다. 

![](https://images.velog.io/images/minni/post/b425a1b5-a532-40fd-a05d-f8f6378a5314/image.png)


## frameLayoutGuide
`frameLayoutGuide` 에 대해서 알아보자. <br>
먼저 공식 문서를 보면 다음과 같다. 
- content 사각형과 반대로 ScrollView 자체의 프레임 사각형을 포함하는 오토 레이아웃 제약을 만드려면 이 레이아웃 가이드를 사용하라고 나와있다. 
- 즉, ScrollView의 **변환되지 않는 frame 사각형을 기반으로 하는 레이아웃 가이드**이다. 

![](https://images.velog.io/images/minni/post/8998c726-2504-45d4-8099-395c529bf3f3/image.png)

예를 들면 아까 위에서 만들었던 화면에서 스크롤이 되는 content 부분과는 별개로 화면에서 계속 보이는, 즉 맨 위로가기 라는 버튼을 만들고 싶다고한다. 이 버튼의 경우에는 scroll 되는 화면과는 별개로 화면에 계속 남아 있어야하므로 제약을 줄 때 frame Layout Guide와 제약을 주면 된다. 

![](https://images.velog.io/images/minni/post/7814e6ce-c1e3-428a-9749-44928bf55036/image.png) 

위의 그림과 같이 frame Layout Guide와 제약을 주면 화면에서 스크롤의 어느 위치에 있던 button은 항상 저 자리에 있는 것을 확인해 볼 수 있다. 


### frame Layout Guide 이용한 팁
ScrollView가 올라가 있는 상태에서 Content View안에 아무것도 없을 때, 아래의 그림과 같이 빨간색 줄이 보기 싫은 경우에는 다음과 같이 설정을 해주면 빨간색 줄을 보지 않고 _마음의 안정을 찾을 수 있다._ 

![](https://images.velog.io/images/minni/post/7fcc903f-52c3-4e96-bc85-df39cf85da74/image.png) 

이 과정은 필수는 아니지만 Auto Layout에서 빨간줄이 보기싫다면 이를 통해 심신의 안정을 얻을 수 있다. 

**1.Content View의 height를 ScrollView의 height와 Equal로 설정해보자.** <br>
![](https://images.velog.io/images/minni/post/c4d1c53d-3b9b-4522-b05a-939eaf0d23b1/image.png)

다음과 같이 빨간줄이 사라진 것을 확인할 수 있다. 

**2. 여기에 label을 올리고 아까와 같이 top, bottom, leading에 제약을 10씩 줘보자.** <br>
![](https://images.velog.io/images/minni/post/8c085787-fdbc-4f2a-9437-bc48bc2fa1da/image.png) 

기존에는 Content View의 height이 label과의 제약에 맞춰서 줄어 들었다면 이번에는 아까와는 다르게 label이 늘어난 것을 확인할 수 있다.  이는 1번에서 설정해준 Content View의 height이 ScrollView의 height과 동일하게 설정 되어 있기 때문에 label이 늘어나 버린 것이다. 

**3. 1번의 제약의 priority를 낮춰보자.** <br>
![](https://images.velog.io/images/minni/post/db8830c3-7677-4382-814c-28cffa0156dc/image.png) 

priority를 250으로 낮춰주었더니 처음과 동일한 결과를 나타나게 된다. 

## 참고
[Apple document: Auto Layout Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/WorkingwithScrollViews.html#//apple_ref/doc/uid/TP40010853-CH24-SW1) <br>
[Apple document: frame Layout Guide](https://developer.apple.com/documentation/uikit/uiscrollview/2865772-framelayoutguide)<br>
[Apple document: content Layout Guide](https://developer.apple.com/documentation/uikit/uiscrollview/2865870-contentlayoutguide)<br>
[야곰닷넷 Auto Layout ScrollView](https://yagom.net/courses/autolayout/lessons/working-with-scroll-views/topic/the-basis/)