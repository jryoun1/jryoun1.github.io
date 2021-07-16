---
title:  "[WWDC]Advances in UICollectionView"
excerpt: "WWDC20에서 iOS 14에서 UICollectionView에서의 변화에 대해서 알아보자."

categories:
  - WWDC
tags:
  - [UICollectionView, Compositional Layout, Diffable DataSource, Modern Cells]
last_modified_at: 2021.01.17
---
# 들어가며
오늘은 WWDC 2020에서 기존의 UICollectionView가 iOS 14에서 어떻게 변화했는지에 대해서 포스팅 해 볼 예정이다. 

# UICollectionView Advance
![](https://images.velog.io/images/minni/post/8ef7e9e8-a12c-4ece-b01b-8268cdcbc8c3/image.png) 

위의 그림과 같은 프로젝트 예시를 가지고 이번 블로그 포스팅을 진행할 것이다. 

이 emoji project는 3가지 부분으로 구현이 되어 있다고 할 수 있다. 
먼저 가장 위쪽에는 emoji들이 **가로로 스크롤 할 수 있게** 되어있다.
가운데 부분은 iOS 14에서 새로 생긴 **expandable-collapsible outline**이다. <br>
(Outline에 해당하는 버튼을 눌렀을 때 뷰가 펼쳐지고 다시 접히게 되어서 위와 같은 이름이 붙은 것 같다) 
그리고 마지막 부분은 UITableView 처럼 생긴 **List 형태**를 가지고 있는 것을 볼 수 있다. 

## UICollectionView APIs
### iOS 6
UICollectionView를 구성하는 API는 크게 3가지 카테고리로 나눌 수 있다. 

![](https://images.velog.io/images/minni/post/ec2354b7-4623-4123-96f2-4d968134d1a1/image.png) 

위의 그림처럼 **`Data`, `Layout`, `Presentation`** 이렇게 3가지 이다. 

UICollectionView 는 **layout에 있는 data**, 즉 무엇이(what), 
**어디(where)에 있는 content에 rendering 되는가**에 대한 개념을 가지고 만들어졌다. 이러한 구분이 UICollectionView를 **매우 유연하게 만들어 준 핵심**이라고 할 수 있다. 

UICollectionView가 iOS 6 에서 처음 release 되었을 때 `data`는
indexPath 기반 프로토콜인 **UICollectionViewDataSource** 를 통해서 관리했었다. <br>

그리고 `layout` 은 UICollectionViewLayout 추상 클래스를 상속하는 **UICollectionViewFlowLayout**을 사용했다.
`presenatation` 은 **UICollectionVeiwCell** 과 **UICollectionReusableView** 두 개의 view 타입을 사용했었다.

### iOS 13
![](https://images.velog.io/images/minni/post/ceae0677-37a6-41d4-abac-7626a1dcb200/image.png) 

그러나 iOS 13에서 Data와 Layout은 **Diffable Data Source**와 **Compositional Layout** 이라는 2개의 새로운 구성요소가 소개되었고, 이 API들은 오늘날UICollectionView 를 만드는데 사용된다. 

### iOS 14
그리고 대망의 iOS 14 에서는 위의 API를 기반으로 새로운 기능들을 만들어내었다.

![](https://images.velog.io/images/minni/post/0c6d6187-fc45-4262-b168-de7ef4c3577c/image.png) 

## Diffable Data Source
### iOS 13
먼저 Diffable Data Source에서 향상된 기능에 대해서 말해보자면,
iOS 13에 도입된 Diffable Data Source는 새로운 **`snapshots`를 추가하여 UI 상태 관리를 크게 간소화**한다. <br>
Snapshot은 **고유한 섹션 및 항목 식별자를 사용해 전체 UI 상태를 캡슐화**합니다.
UICollectionView를 업데이트할 때 **먼저 새 snapshot을 생성**하여 **현재 UI 상태로 채운 다음 data source에 적용**한다.<br>
그러면 Diffable Data Source는 개발자의 **추가 작업 없이 차이점을 계산하고 자동으로 애니메이션을 생성**한다.
이에 대한 자세한 내용은 [WWDC 2019 Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220)를 확인해보면 된다. 

### iOS 14
iOS 14에서는 `section snapshots` 이라는 새로운 snapshot 타입이 생기게 된다. 
이름이 암시하듯이 section snapshot은 **UICollectionView의 하나의 section의 데이터를 캡슐화**해준다. <br>
이러한 향상에는 크게 2가지 이유가 있는데,
첫 번째로 **data source로 하여금 section-sized 단위로 data를 구성하기 쉽게 하기 위해서이다.** 두 번째는 **outline 스타일의 UI rendering을 지원하는 데 필요한 계층적 데이터의 모델링을 허용하기 위해서** 인데, iOS 14에서 찾을 수 있는 흔한 시각적인 디자인이 바로 outline style UI이다. 

![](https://images.velog.io/images/minni/post/b7efc87f-b832-45ea-8463-356587fd7b57/image.png) 

따라서 맨 처음의 emoji project에서 보았던 부분들을 다시 보면 위의 그림과 같이 변한것을 볼 수 있다. <br>

`horizontally scrolling section` 은 `single section snapshot` 으로 구현되어 해당 부분에서 사용되는 content를 modeling하는 것을 확인해 볼 수 있다. <br>

`expandable-collapsible outline style` 인 두 번째 section은 두 번째 `single section snapshot` 을 사용하여 hierarchy data를 modeling 한다. <br>

`list section` 도 세 번째 `single section snapshot`을 해당 섹션을 다시 modeling 하게 된다. 

즉, Emoji project의 경우에는 **Diffable Data Source를 세 개의 개별 섹션 스냅샷으로 구성**했으며, **각 섹션은 단일 섹션의 컨텐츠를 나타냅니다.** <br>
이러한 부분이 iOS 14에서 새롭게 추가된 부분이고, reordering API들도 추가되었는데 이부분에 대해서 자세히 알아보려면 [Advances in Diffable Data Source](https://developer.apple.com/videos/play/wwdc2020/10045)를 확인해보면 된다.


## Compositional Layout
### iOS 13
Composition Layout은 iOS 13에서 처음 소개되었으며, 
작은 레이아웃들을 함께 구성하여 복잡하고 풍부한 레이아웃을 만들 수 있도록 해주었다.

Compositional Layout은 layout의 작동 방식이 아닌, **layout의 모양을 묘사하는 것이므로 훨씬 더 빠르다.** 또한 **section-specific 레이아웃들을 사용해서 정교한 UI**들을 만들 수 있게 해준다. 게다가 **orthogonal(직교) scrolling section 도 지원**을 해준다. <br>
이러한 부분에 대해서 더욱 자세하게 알고 싶다면 Advances in CollectionView Layout 영상을 보면 된다. 

### iOS 14
iOS 14에서는 **Compositional Layout을 기반으로 만든** 새로운 기능을 가지고 있는 `Lists`가 소개된다. `Lists`는 모든 UICollectionView에 **UITableView와 같은 유사한 section들을 포함할 수 있게 해준다.** 
이러한 `Lists`는 **UITableView에서 기대할 수 있는 다양한 기능들** (e.g. swipe, common cell layouts)이 **포함**되어있다.
`Lists` 는 Compositional Layout을 기반으로 작성되므로 **섹션별로 다른 레이아웃과 Lists를 쉽게 혼합하고 매칭**할 수 있다.

Compositional Layout을 사용해서 list-style layout을 생성하는 것을 코드로 확인해보자.
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionLayout.list(using: configuration)
```
위의 코드는 CollectionView의 모든 section이 insectGrouped 모양의 list인 UICollectionView layout을 생성한다. 

![](https://images.velog.io/images/minni/post/8abd6e2e-550a-4c44-883e-e8c89a1dce5e/image.png) 

그 결과 위와 같은 layout이 생성되게 된다. <br>
Compositional Layout List에는 위와 같은 기능 말고도 더 많은 기능들이 있으며 이에 대한 기능들은 UICollectionViewListCell, header & footer 그리고 iPadOS 에서 새로운 Sidebar 모양들이 있으며
[WWDC Lists in UICollectionView](https://developer.apple.com/videos/play/wwdc2020/10026)를 확인해보면 된다. 

## UICollectionView & Modern Cells
### cell registrations
iOS 14에서는 cell registrations이라는 새로운 API를 사용해서 새로운 방식으로 cell들을 구성할 수 있다. 
`Cell registrations` 은 view model로 부터 cell을 설정하는 간단하고, 재사용이 가능한 방법이다.
`Cell registrations` 은 resue identifier를 사용해서 cell class나 nib을 등록하는 추가 과정을 생략해버린다. 

대신에 view model로부터 새로운 cell을 설정하기 위해서 configuration closure를 사용하는 일반적인 registration type을 사용하게 된다. 실제로 API를 살펴보자
```swift
let reg = UICollectionView.CellRegisteration<MyCell, ViewModel> { cell, indexPath, model in 
    // configure cell content
}

let dataSource = UICollectionViewDiffableDataSource<S, I>(collectionView: collectionView) {
                     collectionView, indexPath, item -> UICollectionViewCell in 
      return collectionView.dequeueConfiguredReusableCell(using: reg, for: indexPath,
                             item: item.viewModel)
}
```
먼저 **cell registration을 생성**해준다. <br>
이때 **cell class들을 상속하는 generic 타입**인데, 여기서는 MyCell과 ViewModel class이다. registration은 **closure를 가지는데, 새로운 cell이 생성되면 해당 closure가 호출**된다. 따라서 바로 **해당 closure 부분이 ViewModel로 부터 cell의 content를 구성해주는 부분**이다. 

cell registration을 사용하면 더 이상 기존에 register를 등록하는 추가단계가 필요하지 않다. registration을 사용해서 새로운 API dequeueConfiguredReusalbeCell과 함께 Diffable DataSource cell provider로 부터 cell registration을 사용하면 된다. 

### cell content configurations
cell content configuration은 **UITableView 표준 cell 타입에 표시된 것과 비슷한 cell에 대한 표준화된 레이아웃을 제공**한다. 이러한 configuration은 **어떠한 cell에나 사용될 수 있으며, 일반적인 UIView에도 사용이 가능**하다. 
cell content configuration은 매우 유연하며 실제로 예시들을 확인해보자. 

![](https://images.velog.io/images/minni/post/e66d8c3f-3c73-44f6-8c49-8396648ad7e7/image.png) 

위의 그림과 같이 일반적인 cell layout에 간단한 이미지와 text가 들어가 있다고 하자. 
이때 아래 코드처럼 **`UIListContentConfiguration.cell()`로 구성**을 하게 되면 위와 같이 생성이 된다. 
```swift
var contentConfiguration = UIListContentConfiguration.cell()
contentConfiguration.image = UIImage(systemNamed: "hammer")
contentConfiguration.text = "Ready. Set. Code."
cell.contentConfiguration = contentConfiguration
```

![](https://images.velog.io/images/minni/post/a9f0d40c-37b2-4293-980d-1d930aa57edf/image.png) 

반면에 아래 코드처럼 **`UIListContentConfiguration.valueCell()`로 구성**을 하게 되면 위와 같이 생성이 된다. 

```swift
var contentConfiguration = UIListContentConfiguration.valueCell()
contentConfiguration.image = UIImage(systemNamed: "hammer")
contentConfiguration.text = "Ready. Set. Code."
cell.contentConfiguration = contentConfiguration
```

![](https://images.velog.io/images/minni/post/73583a9d-f056-45ce-8c6c-facf2797c564/image.png)

반면에 아래 코드처럼 **`UIListContentConfiguration.subtitleCell()`로 구성**을 하게 되면 위와 같이 생성이 된다. 
```swift
var contentConfiguration = UIListContentConfiguration.valueCell()
contentConfiguration.image = UIImage(systemNamed: "hammer")
contentConfiguration.text = "Ready. Set. Code."
cell.contentConfiguration = contentConfiguration
```
이러한 **content Configuaration은 매우 가벼우며, content의 모양을 매우 간단하게 설명**해준다. framework가 모든 레이아웃 고려 사항들을 처리하고 성능을 자동으로 최적화하게 된다. 

### background configurations
content configuration과 매우 유사하지만 **모든 cell들의 background에 색상, 테두리 스타일 등과 같은 속성을 조절할 수 있는 기능을 가지고 있다.** 따라서 modern cell configuration의 surface를 거의 건들이지 않게 된다. 
<br> 이에 대해서 자세하게 알고 싶다면 [WWDC Modern Cell Configuration](https://developer.apple.com/videos/play/wwdc2020/10027)을 참고하면 된다. 

# 참고
1. [WWDC 2020 Advances in UICollectionView](https://developer.apple.com/videos/play/wwdc2020/10097/?time=506)