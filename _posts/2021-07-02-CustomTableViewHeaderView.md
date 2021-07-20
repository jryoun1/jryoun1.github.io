---
title:  "[Swift] Custom TableViewHeaderView 생성하기"
excerpt: "Custom TableViewHeaderView 생성하기"

categories:
  - Swift
tags:
  - [TableViewHeaderView]
last_modified_at: 2021.07.02
---

## 들어가며 
이번 포스팅은 프로젝트를 진행하던 도중에 생긴 궁금증도 해결할 겸 작성해 본다. 
프로젝트는 간단한 날씨 앱 프로젝트 인데, `TableView` 의 HeaderView를 생성해야 할 일이 있었다. 

상황은 아래의 그림과 같이 현재 날씨를 받아와서 보여주는 부분과 
5일치의 예보 날씨를 보여주는 부분이 나뉘게 된다. 

![](https://images.velog.io/images/minni/post/477321f9-afc9-4cd6-87ec-930630bb9936/image.png)

여기서 `TableView` 를 사용해서 구현하는 방법에는 2가지 있다고 생각한다. 

1. 2개의 Section으로 구현
현재 날씨를 나타내는 `TableViewCell` 과 예보날씨를 나타내는 `TableViewCell` 각각 생성
2. 1개의 Section으로 구현
예보 날씨를 나타내는 `TableViewCell` 생성과 `TableViewHeader`에 현재 날씨 구현

이 두 가지 방법 중에서 2번째 방법으로 구현을 해보려고 한다. 
2번째 방법을 선택한 이유는 만약 현재 날씨를 나타내는 `cell` 이 여러 개이고 반복 되는 것이라면 1번 방법처럼 이를 하나의 `CustomTableViewCell` 로 구현해 2개의 Section으로 구현했을 것 같다.
그러나 위의 경우에는 현재 날씨 `cell`은 한 번만 나타나기 때문에 `TableViewHeader`에 구현해도 괜찮을 것 같다고 생각했다. 

그렇다면 이를 구현하는 방법에 대해서 알아보자.

## TableViewHeaderView 추가하기

Table View에서 `Section` 은 `Header + rows + Footer` 를 의미한다.
이때 `Header`, `Footer` 는 다른 section과의 구분을 지어주는 영역으로 section의 시작과 끝을 알 수 있는 부분이다. 

처음에는 `TableView` 의 style을 default인 `plain` 으로 하여 `TableViewHeaderView` 를 추가하였다. 

추가하는 방법은 다음과 같다. 

### UITableViewHeaderFooterView 객체 생성 후 구현

`UITableViewHeaderFooterView` 객체 생성하고 Header view를 아래의 코드와 같이 구현해준다. 

```swift
import UIKit

final class WeatherTableViewHeaderView: UITableViewHeaderFooterView {
    static let headerViewID = "WeatherTableViewHeaderView"
    var weatherImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.translatesAutoresizingMaskIntoConstraints = false
        return imageView
    }()
    
    private var addressLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.adjustsFontForContentSizeCategory = true
        label.font = .preferredFont(forTextStyle: .body)
        return label
    }()
    
    private var mininumAndMaximumTemperatureLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.adjustsFontForContentSizeCategory = true
        label.font = .preferredFont(forTextStyle: .body)
        return label
    }()
    
    private var currentTemperatureLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.adjustsFontForContentSizeCategory = true
        label.font = .preferredFont(forTextStyle: .largeTitle)
        return label
    }()
    
    private var verticalStackView: UIStackView = {
        let stackView = UIStackView()
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.axis = .vertical
        stackView.distribution = .fillProportionally
        stackView.spacing = 10
        return stackView
    }()
    
    override init(reuseIdentifier: String?) {
        super.init(reuseIdentifier: reuseIdentifier)
        setupHeaderView()
        configureLayout()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
    
    private func setupHeaderView() {
        verticalStackView.addArrangedSubview(addressLabel)
        verticalStackView.addArrangedSubview(mininumAndMaximumTemperatureLabel)
        verticalStackView.addArrangedSubview(currentTemperatureLabel)
        contentView.addSubview(weatherImageView)
        contentView.addSubview(verticalStackView)
    }
    
    private func configureLayout() {
        NSLayoutConstraint.activate([
            weatherImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 10),
            weatherImageView.widthAnchor.constraint(equalToConstant: 100),
            weatherImageView.heightAnchor.constraint(equalTo: weatherImageView.widthAnchor),
            weatherImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            
            verticalStackView.topAnchor.constraint(greaterThanOrEqualTo: contentView.topAnchor, constant: 5),
            verticalStackView.leadingAnchor.constraint(equalTo: weatherImageView.trailingAnchor, constant: 10),
            verticalStackView.bottomAnchor.constraint(greaterThanOrEqualTo: contentView.bottomAnchor, constant: -5),
            verticalStackView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            verticalStackView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -10)
        ])
    }
}
```

### TableView에 생성한 객체 등록
`UITableViewHeaderFooterView` 객체를 table view에 등록하면 된다. 

```swift
final class MainViewController: UIViewController {
    private let weatherTableView = UITableView()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupWeatherTableView()
    }
        
     private func setupWeatherTableView() {
     weatherTableView.delegate = self
     weatherTableView.dataSource = self
     weatherTableView.register(WeatherTableViewHeaderView.self, forHeaderFooterViewReuseIdentifier: WeatherTableViewHeaderView.headerViewID)
        
     weatherTableView.translatesAutoresizingMaskIntoConstraints = false
     view.addSubview(weatherTableView)
     NSLayoutConstraint.activate([
         weatherTableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
         weatherTableView.topAnchor.constraint(equalTo: view.topAnchor),
         weatherTableView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
         weatherTableView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])
    }
}
```
### TableViewDelegate 함수에서 생성한 뷰 사용

table view delegate 객체에 `tableView(\_:viewForHeaderInSection:)` 함수에 생성한 `UITableViewHeaderFooterView` 객체를 headerview로 불러와서 사용 

```swift
    func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
        guard let weatherTableViewHeaderView = tableView.dequeueReusableHeaderFooterView(withIdentifier: WeatherTableViewHeaderView.headerViewID) as? WeatherTableViewHeaderView else {
            return UIView()
        }
        
        // weatherTableViewHeaderView property value configure
        // ...
        
        return weatherTableViewHeaderView
    }
```


### 결과 화면
위와 같이 코드를 작성하고 실행한 결과 아래와 같이 동작하게 된다. 

![](https://images.velog.io/images/minni/post/a7be8dad-9cfb-41c7-b4c5-6f7cdc8e4265/Simulator%20Screen%20Recording%20-%20iPhone%2011%20-%202021-07-01%20at%2023.58.24.gif) 

그러나 다음과 같이 `TableView` 를 스크롤 할 때 `TableViewHeaderView` 가 내려가는 것이 아니라 Section이 끝날 때까지 고정되어 있게 된다. 따라서 이 문제를 해결하기 위해서는 다음과 같이 수정해주었다. 

## 해결 방법
먼저 해당 문제는 `TableView`의 style이 `plain` 으로 되어 있기 때문이었다. 
따라서 `TableView` 의 style을 `grouped` 로 변경해주었다. 
그렇게 하면 정상적으로 `TableViewHeaderView` 가 고정되지 않고 스크롤 되는 것을 확인해 볼 수 있다. 

```swift
// 기존 코드
import UIKit
final class MainViewController: UIViewController {
    private let weatherTableView = UITableView()
    // ...
}

// 수정 코드
import UIKit
final class MainViewController: UIViewController {
    private let weatherTableView = UITableView(frame: CGRect.zero, style: .grouped)
    //...
}
```

### 결과 화면
위와 같이 코드를 수정하고 실행한 결과 아래와 같이 동작하게 된다. 
![](https://images.velog.io/images/minni/post/b1e2eb3f-a865-4468-8806-710ce4d67c49/Simulator%20Screen%20Recording%20-%20iPhone%2011%20-%202021-07-02%20at%2000.51.30.gif) 이제 정상적으로 `TableView` 를 스크롤 할 때 `TableViewHeaderView` 가 고정되지 않고 스크롤 되는 것을 확인해 볼 수 있다. 

### 💡 TableView Style 
`table view` 는 아래와 같은 3가지 style을 가진다. 

![](https://images.velog.io/images/minni/post/0c53e561-8305-42f3-b468-d757598771a5/image.png)

- `plain`: 기본 table view style <br>
=> section header, footer는 table view가 스크롤 될 때 inline seperator로 구분되며, float하게 표시 
- `grouped`: plain에서 header와 footer의 간격을 늘려주는것 <br>
=> section header, footer가 non-float하게 표현 
- `inset grouped`: rows들의 bounds가 둥근 모형이며, row 양 옆에 inset이 들어감


### ⚠️ 추가로 발생할 수 있는 문제점 
`TableView` 의 style을 plain → grouped로 변경하고 사용하게되면 문제점이 발생할 수 있다. 
만약 **Footer가 필요없고 Header만 필요한 경우**에도 자동적으로 Footer까지 생성되게 된다. 
(반대로 **Header가 필요없고 Footer가 필요한 경우**도 발생 가능)

이러한 경우에는 다음과 같이 해결할 수 있다. 
```swift
let tableView = UITableView()

// footer가 필요없는 경우
tableView.sectionFooterHeight = 0

// header가 필요없는 경우
tableView.sectionHeaderHeight = 0
```



## 참고문헌 
1. [Apple Document - Adding Headers and Footers to Table Sections](https://developer.apple.com/documentation/uikit/views_and_controls/table_views/adding_headers_and_footers_to_table_sections)
2. [Apple Document - UITableView.Style](https://developer.apple.com/documentation/uikit/uitableview/style)