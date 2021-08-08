---
title:  "[Swift] HTTP multipart/form-data 사용하기"
excerpt: "서버로 데이터를 POST할 때 multipart/form-data 타입을 사용해보자"

categories:
  - Swift
tags:
  - [multipart/form-data, HTTP]
last_modified_at: 2021.08.08
---

## 들어가며
App을 개발하는 도중에 서버와 네트워크 통신을 하는데 데이터를 보낼 때 서버측에서 요청하는 데이터의 타입에는 여러 가지가 있을 수 있다. 그 중에서 `multipart`는 무엇이며, 왜 사용하며, 어떻게 사용해야 하는지에 대해서 알아보자.

## HTTP 
> 인터넷 상에서 클라이언트와 서버가 자원을 주고 받을 때 사용하는 통신 규약

HTTP Method에는 많은 종류가 있지만 그 중에서 서버로 파일이나, 데이터를 업로드 할 때 사용하는 `POST`를 기준으로 하여 포스팅을 작성할 것이다. 

### 클라이언트 → 서버 (파일 업로드) 

- 클라이언트가 웹 브라우저라면 **form**을 통해서 파일을 등록해 전송하게 되고, 웹 브라우저가 보내는 HTTP 메시지는 `Content-Type`이 multipart/form-data로 지정된다. 이때 서버는 multipart 메시지에 대해서 각 파트별로 분리하여 개별 파일의 정보를 얻게 된다.
- 이미지 파일의 경우에도 문자로 이루어져 있기 때문에 이미지 파일을 스펙에 맞게 문자로 생성하여 HTTP request body에 담아서 서버로 전송한다.

![1](/assets/images/Multipartformdata/1.png){: width="200" height="200"}

`HTTP(request, response)` 는 위의 그림과 같이 4개의 파트로 나눌 수 있으며 Message Body에 들어가는 타입을 HTTP Header의 Content-Type 필드에 명시해 줄 수 있다. 해당 필드에 들어갈 수 있는 타입 중 하나가 바로 **multipart**이다. 

### form
> 입력 양식 전체를 감싸는 태그를 의미

- `name` : **form의 이름**으로 서버로 보내질 때 이름의 값으로 데이터 전송
- `action` : form이 전송되는 서버 url 또는 html 링크
- `method` : 전송 방법 <br> `GET`: Default / `POST`: 데이터를 url에 공개하지 않고 숨겨서 전송
- `autocomplete` : `.on` 으로 설정하면 form 전체에 자동 완성 허용
- `enctype` : 폼 데이터가 서버로 제출될 때 해당 데이터가 인코딩 되는 방법 
  - `application/x-www-form-urlencoded` : default로 값으로 **모든 문자들을 서버로 보내기 전에 인코딩**됨을 명시
  - `text/plain` : **공백문자는 "+" 기호로 변환**하지만 **나머지 문자는 모두 인코딩되지 않음**을 명시
  - `multipart/form-data` : **모든 문자를 인코딩하지 않음**을 명시, 주로 **파일이나 이미지**를 서버로 전송할 때 사용

## multipart
> HTTP Header에 Message Body에 들어갈 데이터 타입을 정의하는 Content-type의 필드 중에서 MIME(Multipurpose Internet Mail Extensions) 타입 중의 하나이다.

### multipart 사용 이유
파일을 업로드 할 때 사진 설명과 사진을 위한 `input` 2개 있다고 할 때, 사진 설명 `input` 의 content-type은 `application/x-www-form-urlencoded` 이 될 것이고, 사진 `input` 의 content-type은 `image/jpeg` 가 될 것이다. 이때 HTTP Request Body의 content-type으로는 하나의 타입이 들어가야하기 때문에 이러한 경우, 즉 **한 개의 body에 2종류 이상의 데이터나 여러 개의 메시지를 하나의 메세지로 구분해서 만들어주는 것**이 바로 `multipart` 타입이다. 

### multipart/form-data 특징
- multipart 타입을 통해 MIME은 **트리 구조의 메세지 형식을 정의**할 수 있다
- multipart 메시지는 `"Content-type:"` 헤더에 **boundary 파라미터를 포함**
- boundary는 **메시지 파트를 구분하는 역할**을 하며, **메시지의 시작과 끝 부분도 나타냄**
- 첫 번째 boundary 전에 나오는 내용은 MIME을 지원하지 않는 클라이언트를 위해 제공
- boundary를 선택하는 것은 클라이언트의 몫 <br>(주로 무작위 문자(UUID)를 선택해서 메시지 본문과의 충돌을 피함)

### multipart/form-data 사용할 때 HTTP 규약

![2](/assets/images/Multipartformdata/2.png)

서버에 multipart/form-data로 데이터를 보낼때의 request header와 body는 위 이미지와 같이 구성 되어있다. [(이미지 출처 탁구치는 개발자)](https://lng1982.tistory.com/209)

이미지에서 여러 규약들을 확인해 볼 수 있다. 

- `Content-Type` : multipart/form-data로 지정되어야함
- 전송되는 **파일 데이터의 구분자**로 **boundary에 지정되어 있는 문자열**을 이용
- boundary의 문자열 중 마지막 `------WebKitFormBoundary(UUID)--` 는 마지막에 `--` 가 추가로 붙는데, 이는 **body의 끝을 알리는 의미**

## HTTP Request, Response 형식
아래의 실제 HTTP Request와 Response의 데이터를 살펴보면서 어떠한 형식으로 되어있는지 확인해보자.
```swift
// HTTP Request Data
POST /file/upload HTTP/1.1[\r][\n]
Content-Length: 344[\r][\n]
Content-Type: multipart/form-data; boundary=Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]
Host: localhost:8080[\r][\n]
Connection: Keep-Alive[\r][\n]
User-Agent: Apache-HttpClient/4.3.4 (java 1.5)[\r][\n]
Accept-Encoding: gzip,deflate[\r][\n]
[\r][\n]
--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]
Content-Disposition: form-data; name=files; filename=test.txt[\r][\n]
Content-Type: application/octet-stream[\r][\n]
[\r][\n]
aaaa
[\r][\n]
--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]
Content-Disposition: form-data; name=files; filename=test1.txt[\r][\n]
Content-Type: application/octet-stream[\r][\n]
[\r][\n]
1111
[\r][\n]
--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu--[\r][\n]

// HTTP Response Data
HTTPHTTP/1.1 200 OK[\r][\n]
Server: Apache-Coyote/1.1[\r][\n]
Accept-Charset: big5, big5-hkscs, euc-jp, euc-kr...[\r][\n]
Content-Type: text/html;charset=UTF-8[\r][\n]
Content-Length: 7[\r][\n]
Date: Mon, 30 Jun 2014 01:28:19 GMT[\r][\n]
[\r][\n]
SUCCESS
```

 - `header` 와 `header` 를 구분하는 것은 **개행 문자 1개**
 - `header` 와 `body` 를  구분하는 것은 **개행 문자 2개** 
 - `body` 에 포함되어 있는 `filedata` 를 구분하는 것은 **boundary**

 ⚠️ `\r`은 **carrige return**으로 **입력 커서를 라인 맨 앞으로 이동**시키는 특수 문자이다. <br>
그리고 개행 문자(\n)는 다음 라인으로 이동시키는 특수 문자이다. <br> 개행 문자도 Line Feed 라고 불러서 `\r\n`을 CRLF라고 많이 한다고 한다. 
<br>그러나 리눅스에서는 `\r`이 안 붙는다. 왜냐하면 리눅스에서는 `\n` 하나로 윈도우에서 `\r\n`이 하는 일을 처리하기 떄문이다.

## 실제 프로젝트에서 사용하기 

### Content-Type, httpMethod 설정
실제로 프로젝트에서 이를 사용하려면 다음과 같이 `Content-Type`과 `httpMethod`를 설정해주어야한다. 
```swift
let boundary = "Boundary-\(UUID().uuidString)"

var request = URLRequest(url: URL(string: "https://[서버URL]")!)
request.httpMethod = "POST"
request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
```
### httpBody 구현
그리고 request에 httpBody를 multipart/form-data 형태로 생성해주고 이를 추가해야한다. 이때 Data Type을 httpBody에 추가하는 것과 나머지 타입들을 추가하는 방법은 아래와 같이 다르다. 

```swift
// 나머지 타입을 추가하는 경우
func convertFormField(named name: String, value: String, using boundary: String) -> String {
  var fieldString = "--\(boundary)\r\n"
  fieldString += "Content-Disposition: form-data; name=\"\(name)\"\r\n"
  fieldString += "\r\n"
  fieldString += "\(value)\r\n"

  return fieldString
}

// Data 타입을 추가하는 경우
func convertFileData(fieldName: String, fileName: String, mimeType: String, fileData: Data, using boundary: String) -> Data {
  let data = NSMutableData()

  data.appendString("--\(boundary)\r\n")
  data.appendString("Content-Disposition: form-data; name=\"\(fieldName)\"; filename=\"\(fileName)\"\r\n")
  data.appendString("Content-Type: \(mimeType)\r\n\r\n")
  data.append(fileData)
  data.appendString("\r\n")

  return data as Data
}

extension NSMutableData {
  func appendString(_ string: String) {
    if let data = string.data(using: .utf8) {
      self.append(data)
    }
  }
}
```
위와 같은 변환해주는 함수들을 생성하고 이제 실제로 httpBody에 데이터 모델의 프로퍼티들을 변환해서 넣어주면 된다.
```swift
let httpBody = NSMutableData()

for (key, value) in formFields {
  httpBody.appendString(convertFormField(named: key, value: value, using: boundary))
}

httpBody.append(convertFileData(fieldName: "image_field",
                                fileName: "imagename.png",
                                mimeType: "image/png",
                                fileData: imageData,
                                using: boundary))

httpBody.appendString("--\(boundary)--")

request.httpBody = httpBody as Data
```

### 네트워크 요청
그리고 생성한 request로 네트워크 작업을 진행하면된다. <br>
```swift
URLSession.shared.dataTask(with: request) { data, response, error in
  // handle the response here
}.resume()
```
그럼 multipart/form-data 형태로 데이터가 POST되고 서버에서는 이를 받아서 multipart 메시지에 대해서 각 파트별로 분리하여 개별 파일의 정보를 얻게 된다.

## 마치며
이번 포스팅을 통해서 multipart타입이 무엇이며, 왜 사용하는지 그리고 직접 multipart/form-data으로 httpBody를 작성해보면서 구현하기 급급했었던 지난 번과는 다르게 확실하게 이해 할 수 있었다. 물론 나중에는 Alamofire나 다른 서드파티 라이브러리를 사용할 수도 있겠지만, 직접 URLSession을 통해서 네트워크 통신을 구현해야하는 경우도 있을 수 있으니까 이번 경험과 기록이 훗날 좋은 자료가 되었으면 한다. 🤓 

## 참고
[레나참나 HTTP multipart/form-data 이해하기](https://lena-chamna.netlify.app/post/http_multipart_form-data/) <br>
[탁구치는 개발자 HTTP multipart/form-data raw 데이터는 어떤 형태일까?](https://lng1982.tistory.com/209) <br>
[JungHyun Baek Multipart/form-data란?](https://junghyun100.github.io/Multipart_form-data/) <br>
[Donnywals Uploading images and forms to a server using URLSession](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/)