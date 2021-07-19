---
title:  "[Git] git commit log 작성 방법"
excerpt: "git commit log 작성 방법"

categories:
  - Git
tags:
  - [commit log]
last_modified_at: 2021.05.21
---

## 들어가며
프로그래머에게 있어서 어려운 일 중에 하나는 바로 변수명, 함수명 짓기라고 생각합니다. 심지어 Swift에서는 이러한 변수명, 함수명을 정하는 규칙인 [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)도 제공합니다.

그렇다면 변수명, 함수명 짓기가 왜 어려우면서 까다로울까요? 🧐

아래의 예시 코드를 볼까요? 예시 코드는 1부터 10까지의 합을 구하는 함수를 가진 코드입니다.

```swift
var jake = 0
func a() {
    for i in 1 ... 10 {
        jake += i
    }
    print(jake)
}
a() // 55
```

과연 이 코드에서 jake라는 변수명이 무엇을 의미하는지 혹은 a함수 그리고 i가 무엇을 의미하는지 바로 알 수 있을까요? 다시 한 번 똑같은 기능을 하지만 다른 변수명, 함수명을 가진 아래의 함수를 보죠.

```swift
var sumResult = 0
func addOneToTen() {
    for value in 1 ... 10 {
        sumResult += value
    }
    print(sumResult)
}
addOneToTen() // 55
```

위의 코드에 비해서는 굳이 코드 전부를 읽지 않아도 변수명 그리고 함수명만 보고도 어떠한 역할을 하는지 바로 알 수 있는 것 같습니다.

다시 아까 위의 질문인 변수명, 함수명 짓기가 어려우면서 까다로운 원인에 대해서 대답을 해보자면 물론 프로그래밍에서 이에 대한 정확한 정답은 없겠지만 제가 생각한 답은 바로 프로그래밍을 할 때 중요한 것 중 하나인 **가독성** 때문입니다.  프로그래밍은 혼자서도 할 수 있지만 협업을 하는 경우가 많이 있습니다. 이런 경우에 나만 읽기 쉽고 이해가 되는 변수명, 함수명을 사용한다면 해당 코드를 읽어야하는 다른 사람으로서는 정말 고통스러울수도 있을 것 같아요. 😂

따라서 협업을 위해서나 훗날 오랜 시간이 지나고 나서 내가 작성한 코드를 다시 코드를 읽는다고 했을 때에도 이해가 쉽기 위해서 가독성 있는 변수명, 함수명을 사용 하는 것은 프로그래밍에 있어서 중요한 일 중에 하나라고 생각합니다.

## **Git commit message를 잘 써야하는 이유가 무엇일까요?**

이와 비슷하게 프로그래밍에서 어렵고 중요한 일 중 하나는 바로 **commit message** 작성이라고 생각합니다.

그렇다면 git commit message는 왜 작성을 잘해야 할까요? 🤔

이유는 사람마다, 기업 마다 다를 수 있지만 잘 작성된 commit message가 그렇지 않은 commit message에 비해 훨씬 더 유익하다는 사실에 대부분의 프로그래머들이 동의할 것이라고 생각됩니다.

그 중에서 대표적인 3가지 원인은 다음과 같다고 합니다.

1. 더 좋은 commit log 가독성
2. 더 나은 협업과 리뷰 프로세스
3. 더 쉬운 코드 유지보수

그렇다면 여기서 **잘 작성한 commit message**라는 것은 과연 어떠한 것일까요?

프로그래밍에는 항상 정답이 없듯이 잘 작성한 commit message라는 것 역시도 사람마다, 기업마다 기준이 다를 텐데 그나마 더 많은 사람들이 공감하고 실천하고 있는 잘 작성한 commit message 라는 것이 있을까요?

이에 대해서는 여러 commit message style guide가 있겠지만 본문에서는 카르마 commit message style guide를 제안합니다. 또한 이러한 commit message style guide와 함께 잘 작성하기 위한 7가지 약속에 대해서 알아보도록 하죠!! 🔥

## **Commit message style guide**

**Karma Style**

사실 Karma는 연결되어 있는 브러우저에 대한 테스트 코드에 대응하는 소스 코드를 실행하는 웹 서버 생성 도구 입니다. 이에 [카르마 스타일](http://karma-runner.github.io/6.3/dev/git-commit-msg.html)은 Karma에서 Git을 작성할 때 제공하는 convention(규칙)입니다.

카르마에서는 이러한 convention을 제공하는 이유로 2가지 이유를 언급합니다.

- 자동으로 changelog를 생성하기 위해서
- git history를 간단하게 탐색하기 위해서 (예를 들면 type이 style인 경우에 이를 무시하고 볼 수 있다)

### **Commit message 형식**

```bash
<type> (<scope>): <subject>

<body>

<footer>

# 형식에 맞춘 예시
fix(middleware): ensure Range headers adhere more closely to RFC 2616

Add one new dependency, use `range-parser` (Express dependency) to compute
range. It is more well-tested in the wild.

Fixes
```

카르마 스타일의 commit message 형식은 위와 같습니다. 그리고 해당 형식에 맞춰 예시로 작성되어 있는 예시도 같이 확인해 볼 수 있습니다.

그럼 이제 type, scope에는 어떠한 것들이 있으며 subject, body, footer는 어떠한 형식으로 쓰는지 알아보겠습니다.

### **Message subject (첫 번째 줄)**

카르마 스타일에서 git commit message의 첫 번째 줄은 70 자를 넘지 않는 것을 권장하며 두 번째 줄은 한 칸 띄워주고 세 번째 줄부터 각 줄은 80자를 넘지 않는 것을 권장하고 있습니다.  또한 type과 scope는 아래의 종류들을 사용할 수 있으며, **모두 항상 소문자로 사용하는 것**을 권장합니다.

### **가능한 type 종류**

- `feat` = 주로 사용자에게 새로운 기능이 추가되는 경우
- `fix` = 사용자가 사용하는 부분에서 bug가 수정되는 경우
- `docs` = 문서에 변경 사항이 있는 경우
- `style` = 세미콜론을 까먹어서 추가하는 것 같이 형식적인 부분을 다루는 경우 (코드의 변화가 생산적인 것이 아닌 경우)
- `refactor` = production code를 수정하는 경우 (변수의 네이밍을 수정하는 경우)
- `test` = 테스트 코드를 수정하거나, 추가하는 경우 (코드의 변화가 생산적인 것이 아닌 경우)
- `chore` = 별로 중요하지 않은 일을 수정하는 경우 (코드의 변화가 생산적인 것이 아닌 경우)

이때 type은 실제 회사나 팀마다 다를 수도 있으며, 위의 예시는 카르마 스타일 내에서 가능한 type입니다. 그리고 type을 선택하여 commit message를 작성하는 것 역시 프로그래머의 선택이므로 자신의 프로그램에서 commit을 하려는 부분이 어떠한 종류인가 고민해보고, **자신만의 기준을 세워서 작성해보는 것**도 좋은 방법일 것 같습니다.

### **scope 예시**

- init, runner, watcher, config, web-server, proxy, etc

scope의 경우에는 생략이 가능하며, 주로 카르마 도구 내에서 자주 사용하므로 우리는 일단 생략한다고 알아두면 됩니다.

### **subject 작성**

subject(제목)를 작성할 때 명령형 어조를 사용하며, 현재 시제를 사용해야 합니다.

```bash
refactor: renamed the iVars and removed the common prefix
```

그러나 이는 카르마 스타일에서 권장하는 스타일이 아니고 아래와 같이 작성하는 것을 더 권장합니다.

```bash
refactor: rename the iVars to remove the common prefix
```

### **Message body**

body(본문)를 작성할 때 명령형 어조를 사용하며, 현재 시제를 사용하는 것을 권장합니다.

또한 이전 행동과 대조하여 변경을 한 동기를 포함하는 것을 권장합니다.

### **Message footer**

만약 git의 issue 기능을 사용하여 issue를 참조하는 commit message의 경우에는 footer에 closes 를 사용하여 issue를 닫을 수 있다.

```bash
# issue가 한 개인 경우
Closes #234

# issue가 여러개인 경우
Closes #123, 245, 992
```

지금까지 카르마 스타일에 대해서 알아보았는데, 사실 이는 카르마 스타일에서 권장하는 부분이지 또 다른 commit message style인 [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/) 에서는 subject의 시작이 대문자이어야하거나, subject는 50자를 넘지 않는 것을 권장하기도 합니다. 즉, git commit message style은 회사나 팀마다 다 다를 수 있고, 이에 팀원과 어떠한 스타일을 사용해서 작성할 지 의논하여 작성하면 될 것 같습니다.

또한 카르마 스타일 가이드를 보면서 한 가지 의문이 생기는 사람들도 있을 수 있습니다.  지금까지 type과 subject는 잘 작성했지만, body, footer를 꼭 작성해야만 할까🧐에 대한 의문이 생길 수도 있을 것 같은데, 이에 대해서는 굳이 모든 commit message에 body, footer를 작성할 필요없이, 필요로 되어지는 경우에만 작성하면 됩니다.

## **Commit message를 작성을 위한 7가지 약속**

> 아래의 7가지 약속은 commit message를 영어로 작성하는 경우에 최적화되어 있습니다. 따라서 한글로 commit message를 작성하는 경우에는 더 유연하게 적용하여 작성해도 될 것 같습니다.

7가지 약속은 다음과 같습니다.

1. subject와 body 사이는 한 줄 띄워 구분하기
2. subject line의 글자수는 50자 이내로 제한하기
3. subject line의 첫 글자는 대문자 사용하기
4. subject line의 마지막에 마침표(.) 사용하지 않기
5. subject line에는 명령형 어조를 사용하기
6. body는 72자마다 줄 바꾸기
7. body는 `어떻게` 보다 `무엇을`, `왜` 에 맞춰 작성하기

이에 각각의 약속을 아래에서 설명하겠습니다.

### **1. subject와 body 사이는 한 줄 띄워 구분하기**

`git commit` 의 [man page](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion) 에 다음과 같은 글이 나와있습니다.

> Though not required, it’s a good idea to begin the commit message with a single short (less than 50 character) line summarizing the change, followed by a blank line and then a more thorough description. The text up to the first blank line in a commit message is treated as the commit title, and that title is used throughout Git. For example, git-format-patch(1) turns a commit into email, and it uses the title on the Subject line and the rest of the commit in the body.

영어로 길게 되어있는데 이걸 간략하게 요약하면 **subject는 50자 미만으로 작성**하고 body를 작성하기 전에 **한 줄을 띄워쓰고** **body를 작성**하는게 좋다는 이야기 입니다. 물론 위에서도 언급했듯이 모든 commit이 다음과 같은 약속을 따라야하는 것은 아닙니다.

예를 들면 간단하게 세미콜론을 빼먹은 경우에는 굳이 body까지 작성하지 않고 subject만 작성해도 됩니다.  그러나 충분히 commit message에 작성할 내용이 있다면 git의 제안을 따라서 작성하는 것을 권장합니다. 그렇다면 왜 위와 같이 작성 하면 좋다고 할까요? 그 이유는 아래와 같습니다!!

아래에는 위의 형식에 맞춰 subject, 공백 한 줄, body로 구성되어 있습니다.

```bash
Derezz the master control program

MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

이때 `git log` 라는 명령어를 입력하면 `commit log` 들, 즉 commit 한 기록들을 볼 수 있는데 아래와 같이 보여지게 됩니다.

```bash
$ git log
commit 42e769bdf4894310333942ffc5a15151222a87be
Author: Kevin Flynn <kevin@flynnsarcade.com>
Date:   Fri Jan 01 00:00:00 1982 -0200

 Derezz the master control program

 MCP turned out to be evil and had become intent on world domination.
 This commit throws disc of Tron into MCP (causing its deresolution)
 and turns it back into a chess game.
```

여기서 `git log --oneline` 라는 명령어를 입력해서 한 줄 만 보여줘 라는 옵션을 추가해 보면 다음과 같이 나오게 됩니다.

```bash
$ git log --oneline
42e769 Derezz the master control program
```

위와 같이 제목만 보여지게 됩니다. 그러나 만약 subject와 body 사이에 한 줄의 공백 주지 않았다면 어떻게 나올까요?

```bash
$ git log --oneline
42e769 Derezz the master control program MCP turned out to be evil and had become intent on world domination. This commit throws Tron's disc into MCP (causing its deresolution) and turns it back into a chess game.
```

바로 위와 같이 body의 내용도 함께 보이게 됩니다.

또 다른 명령어인 `git shortlog` 는 subject와 body 사이에 공백이 있는 commit 들은 아래와 같이 보여지게 됩니다.

```bash
$ git shortlog
Kevin Flynn (1):
    Derezz the master control program

Alan Bradley (1):
    Introduce security program "Tron"

Ed Dillinger (3):
    Rename chess program to "MCP"
    Modify chess program
    Upgrade chess program

Walter Gibbs (1):
    Introduce protoype chess program
```

즉, 이렇듯 subject와 body 사이에 공백을 한 줄 넣어준다면 프로젝트 중간에 commit message의 subject만 나열해서 보고 싶은 경우에 좀 더 깔끔하게 git log들을 확인할 수 있습니다.

### **2. subject line의 글자수는 50자 이내로 제한하기**

subject의 내용은 50자 이내라는 약속은 그렇게 어려운 제약사항이 아닙니다. 오히려 이를 통해 commit message 작성자는 더욱 핵심적이고 읽기 좋은 commit subject를 작성할 수 있고, 가장 간결히 요약된 subject를 작성할 수 있고 그렇게 작성하도록 고민하는 습관을 들이게 해줍니다.

처음에 50자가 어렵다면 69자로 도전해보는 것도 좋습니다.

### **3. subject line의 첫 글자는 대문자 사용하기**

약속을 소개할 때 말했듯이 해당 7가지 약속들은 영어로 작성하는 경우에 최적화되어 있기 때문에, commit message를 영문으로 작성하는 경우에는 subject의 첫 글자를 대문자로 작성하는 것을 의미합니다. 영문법에서 첫글자는 대문자로 작성해야하는 것은 다들 알고 계시죠? 의외로 영어를 모국어로 사용하는 사람들이 이 문법에 민감하다고 합니다. 이는 마치 한국어로 했을 때 `않해!` 를 보는 수준이라고 합니다.

따라서 아래와 같이 작성하지 마시고

```bash
accelerate to 88 miles per hour
```

이렇게 작성하면 될 것 같습니다.

```bash
Accelerate to 88 miles per hour
```

물론 이는 영어로 commit message를 작성하는 경우에 해당하지 한국어로 subject를 작성하는 경우에는 신경쓰지 않아도 됩니다.

### **4. subject line의 마지막에 마침표(.) 사용하지 않기**

4번 약속도 3번과 동일하게 영문법에 관련되어있습니다. 영어에서는 subject, 제목에 보통 마침표를 찍지 않는다고 합니다.

따라서 아래와 같이 작성하지 마시고

```bash
Open the pod bay doors.
```

이렇게 작성하시면 될 것 같습니다.

```bash
Open the pod bay doors
```

### **5. subject line에는 `명령형` 어조를 사용하기**

subject를 작성하거나 한 줄 메시지로 commit을 할 때, commit message의 가장 첫 문장의 영문법은 `명령조` 로 해야합니다. 즉, 첫 단어는 `동사원형` 으로 작성하라는 의미입니다. 한글로 commit message를 작성하는 경우에는 `명령문` 보다는 `설명문` 으로 작성하는 것이 더 편안하게 받아들여집니다. 예를 들면 이런 메세지가 있다고 가정해 보겠습니다.

```bash
인증 메소드 고쳐라

* CABVerification.java: 15번째 줄 인증 메소드의 인자를 현 정책에 맞게 고쳤다.
* JSONFormat.java: 이 커밋은 이 파일에 적절한 로깅 메소드를 추가한다.
* MainView.java: 약간의 리팩토링.
```

이 commit message의 subject를 다음과 같이 고치고 싶을 것입니다.

```bash
인증 메소드 고쳤다
```

그리고 우리말로 subject를 요약하는 경우라면 때로는 위와 같은 문장보다는 아래와 같은 구문을 더 선호하는 경우도 있습니다.

```bash
인증 메소드 수정
```

그렇다면 왜 영문으로 commit message를 작성하는 경우에는 subject를 `명령문`으로 써야할까요?

위에서 예시로 들은 것처럼 평소에는 `명령문` 이 사람이 읽기에 다소 무례하거나 log message 라는 상황에 어울리지 않는 것처럼 느껴질 수 있습니다. 외국인 역시도 우리와 똑같이 생각하고 있습니다. 그러나 **git 스스로가 자동 commit을 작성할 때 `명령문` 을 사용하고 있습니다.** 예를 들어 `git merge` 명령어를 실행시켰을 때 commit message의 기본값은 다음과 같습니다.

```bash
Merge branch 'myfeature'
```

그리고 `git revert` 명령어를 실행하면 다음과 같은 메시지가 기본 값으로 들어갑니다.

```bash
Revert "Add the thing with the stuff"

This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```

마지막으로 Github에서 Pull-request의 "Merge" 버튼을 클릭하면 자동으로 채워지는 다음과 같습니다.

```bash
Merge pull request #123 from someuser/somebranch
```

이처럼 commit message를 `명령문` 으로 작성한다는 것은, **git built-in convention**을 그대로 따르는 것을 의미합니다. 따라서 commit message를 `명령문` 으로 작성하면, git 스스로 자동으로 채우는 메시지 commit 사이에 자연스레 녹아들게 됩니다. 또한 앞선 항목에서 언급한 `git shortlog` 와 같은 타 명령어에도 연계되어 잘 어울립니다.

위의 `명령조` 는 **subject나 한 줄 commit message인 경우**에 이야기이고, commit message의 **body**를 `명령조` 로 작성할 필요는 없습니다. body를 써야하는 경우에는 자연스럽게 과거형이나 현재형 시제를 사용하여 변경점 평서문으로 작성하면 됩니다.

### **6. body는 72자마다 줄 바꾸기**

git은 자동으로 commit message를 줄바꿈 하지 않습니다. 물론 `git log` 에 스타일을 지정해서 자동으로 줄바꿈이 적용된 log 출력을 만들 수 있습니다. 그렇지 않고 단순히 `git log` 명령어로 보기 좋은 메시지를 만들려고 한다면 적당한 위치에서 Enter키를 눌려주세요. 적당한 위치로 72자를 추천드립니다.

> Tips: git log 에 스타일을 지정하는 방법은 아래에 명령어를 입력해서 사용해보세요. 커스텀 스타일 옵션이 적용된 git log 를 실행시켜주는 "alias" 별칭을 만들어주는 설정입니다. 한 번 설정한 이후에는 log 확인은 git log 대신 git lg 로 간단하게 입력해도 실행됩니다. 명령의 뒷 부분은 자동 줄바꿈과 git log를 보기 좋게 만드는 설정의 예시입니다. 이 예시는 Uno's blog에서 참고하였습니다. 아래의 명령어를 ~/.gitconfig 파일에 작성해놓으면 매번 입력할 필요없이 다음 git 실행부터 항상 적용됩니다.

```bash
$ git config --global alias.lg "log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold red)%h%C(reset) : %C(bold green)(%ar)%C(reset) - %C(cyan)<%an>%C(reset)%C(bold yellow)%d%C(reset)%n%n%w(90,1,2)%C(white)%B%C(reset)%n'"
```

- **`git lg` 실행 결과**

![https://djkeh.github.io/images/20170316_git_commit_message/2.png](https://djkeh.github.io/images/20170316_git_commit_message/2.png)

- **`git lg --one line` 실행 결과**

![https://djkeh.github.io/images/20170316_git_commit_message/3.png](https://djkeh.github.io/images/20170316_git_commit_message/3.png)


### **7. body는 `어떻게` 보다 `무엇을`, `왜` 에 맞춰 작성하기**

subject가 commit message의 내용을 충분히 전달하는 것이 좋습니다. 그리고 body는 어떻게보다는 무엇을, 왜에 맞춰서 작성하는 것이 좋습니다. 이에 대한 예시로 Bitcoin Core 프로젝트에 실제로 사용된 commit message는 아래와 같습니다.

```bash
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.

   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).

   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().

   fail(), clear(n) and exceptions() are just never called. Delete
   them.
```

지금까지 7가지 약속에 대해서 알아보았습니다.

7가지 약속 역시도 제안을 하는 것 뿐이지 git commit message style에는 정답이 없습니다. 따라서 개인 혹은 모둠끼리 스타일 혹은 약속을 제안하여서 작성해도 됩니다. 그러나 약속을 했다면 해당 약속에 따라서 git commit message를 작성하는 것은 중요하겠죠?!🤔

이렇듯 약속해둔 스타일을 지켜 작성하는 습관을 들이는 것은 중요하므로 다들 자신만의 스타일 혹은 제시한 스타일들을 참고해서 git commit message를 작성해나가는 습관을 들여간다면 어떠할까요?!

## **Commit 단위**

야곰🐻 이 commit 단위에 대해서 어떻게 작성하는지 물어봐서 해당 부분을 내용을 추가합니다!!

commit 단위 역시도 팀이나, 회사마다 다르다고 합니다. 저의 경우에는 최대한 한 가지 기능을 하는 것에 대해서 commit을 하려고 합니다. 즉, 함수 단위 혹은 기능 단위로 commit을 하려고 합니다. 물론 commit 단위를 너무 작게 가져가도 자주 commit을 해야해서 좋지 않을 수도 있고, 너무 크게 가져가면 나중에 어떠한 작업이 이뤄졌는지 파악하기가 어려울 수도 있습니다.

따라서 이러한 commit 단위에 대해서 스스로 고민해보거나, 팀원과 상의해서 결정하고 해당 기준에 맞춰서 commit 해나간다면 나중에 프로젝트를 다시 한 번 살펴볼 때 commit log를 보면서 어떠한 작업들을 했는지 확인하는데 큰 도움이 될 수 있겠죠?! 😎 또한 commit을 적절한 단위로 해나간다면 나중에 특정 시점으로 되돌려야하는 경우에도 큰 도움이 될 수 있어요!!

## **선배들은 어떻게 작성하였을까?**

소개해 줄 선배는 바로 우리 캠프의 리뷰어 중에 한 명인 붱이🦉 입니다.

commit log 소개에 기꺼이 자신의 commit log 공유를 해줘서 고마워요 붱이 🙇‍♂️ ☺️

붱이는 앞으로 프로젝트를 진행하면서 여러분들이 프로젝트에 대해서 코드를 작성하면 이에 대해서 리뷰를 해 줄 멋지고 닉값을 제대로 하여 새벽에 주로 활동하는 선배입니다~~ 👏👏

![gitcommitlog1](/assets/images/Git/gitcommitlog1.png)

붱이의 commit log를 보면 type도 소문자로 작성하고, 영문자가 subject의 첫글자인 경우에는 대문자로 작성해주는 것 등의 규칙들을 commit log에서 꾸준하게 지켜가며 작성하는 것을 볼 수 있습니다.

또 다른 리뷰어 선배인 흰의 commit log를 보도록 하겠습니다. 기꺼이 commit log 공유를 허락해주신 흰에게 다시 한번 감사의 인사를 드립니다~ 🙇‍♂️🙇‍♂️ 흰도 붱이와 마찬가지로 우리 캠프의 리뷰어로 여러분의 코드를 아주 세세하게 잘 리뷰해 줄 멋진 선배랍니다!! 👏👏

![gitcommitlog2](/assets/images/Git/gitcommitlog2.png)

흰의 commit log는 영문으로 작성되어있는데, 위에서 말한 여러 규칙들 중에서 type 명은 소문자로 작성, subject의 시작은 대문자로 하며 명령어로 작성한 것을 확인해 볼 수 있습니다. 이와 같이 혼자 프로젝트를 하는 경우에도 commit log를 어떠한 방식으로 작성할 지 결정하고 이에 맞춰 작성하는 습관을 들이면 좋을 것 같습니다.

## **참고 문헌**

1. [Karma git commit msg](http://karma-runner.github.io/latest/dev/git-commit-msg.html)
2. [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)
3. [Chris Beams - How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
4. [Uno's Blog - 좋은 git 커밋 메시지를 작성하기 위한 8가지 약속](https://djkeh.github.io/articles/How-to-write-a-git-commit-message-kor/)
5. [붱이🦉 commit log 예시](https://github.com/O-O-wl/store-app)
6. [흰 commit log 예시](https://github.com/daheenallwhite/WeatherApp/commits/master)