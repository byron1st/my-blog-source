---
title: Dash
tags:
  - macOS
categories:
  - Review
  - Software
date: 2016-06-03 13:36:35

---

### Dash란?

[Dash](https://kapeli.com/dash)는 API 도큐먼트 뷰어 프로그램이다. Dash는 개발자들이 구글 검색을 통해 수많은 비공식 웹페이지들을 헤맬 필요 없이 공식 API 도큐먼트를 바로 찾아볼 수 있도록 해준다. Dash는 Atom, Sublime Text 등과 같은 텍스트 에디터를 선호하는 개발자들이 API 찾을 때 큰 도움이 된다. 매번 웹 브라우저를 열 필요도 없고 (웹 브라우저를 실행시키면, 괜히 클리앙 같은 곳 한번 들어가고 그런다.), 속도도 빠르며, 공식 문서의 내용만 빠르고 깔끔하게 살펴볼 수 있다.<!--more-->

![](dash-screenshot.png)

Dash는 현재 OS X 버전과 iOS 버전을 제공한다. 또한, 각종 텍스트 에디터들(Atom, Visual Studio Code, Sublime Text, Brackets 등)과 IDE들(JetBrains 사의 IDE들, Xcode, Eclipse 등), 그리고 그 외에 Alfred, LaunchBar, Terminal, AppleScript 등과 함께 사용할 수 있는 플러그인도 제공한다. 제공되는 API 도큐먼트들은 150개 이상이며, Java, Javascript, C, C++, C#과 같은 프로그래밍 언어들은 물론, Express, Node.js, Chai, Akka, Jade, Bootstrap 등과 같은 프레임워크들의 API도 제공하며, Vim, Emacs, Bash 등의 단축키 및 사용 가이드도 제공한다. 마지막으로 코드 Snippets 기능도 제공한다. 그야말로 개발자를 위한 종합 선물 세트이며, 텍스트 에디터를 IDE 수준으로 쓸 수 있도록 도와주는 어플리케이션이다.

하지만 Dash의 진정한 힘은 iOS 버전과 OS X 버전을 함께 통합하여 사용할 때 발휘된다.

### ~~Dash for iOS와의 연동~~

_현재 Dash 개발자와 Apple 사이의 분쟁(?)으로 인해 iOS 앱이 앱스토어에서 내려가서 더 이상 구매할 수 없는 상황이다. 개발자가 iOS 버전을 오픈소스로 풀어버렸기 때문에 향후에도 앱스토어에 복귀할 가능성이 높지 않아보인다._

Dash의 iOS 버전은 universal 앱으로, 아이패드에서도 잘 작동한다. (아이폰에서 쓸 일은 많지 않을 것이다.) Dash for OS X는 1) 아이패드와 맥이 같은 와이파이에 접속해 있고, 2) Dash for iOS가 실행되어 있을 때, Dash for iOS와 연동 될 수 있다. Dash for OS X의 설정에는 Remote 탭이 있는데, 위의 1), 2) 조건이 맞을 경우, 해당 Dash for iOS가 리스트에 나타난다.

![](dash-settings.png)

설정의 두 체크 박스들 중, 첫번째 체크 박스가 Dash for iOS의 활용도를 100% 올려주는 중요한 설정이다. 이 체크 박스를 설정할 경우, Alfred, 또는 텍스트 에디터의 플러그인을 활용해 검색해도 Dash for OS X이 활성화되지 않고 자동으로 아이패드의 Dash for iOS에 검색 결과가 표시된다. 즉, OS X의 모니터는 텍스트 에디터를 벗어날 일이 없다.

![Alfred를 이용하여 Dash 검색](dash-alfred-search.png)

![Atom의 플러그인을 활용한 Dash 검색. Atom의 Dash 플러그인은 Ctrl+Shift+H를 누를 경우, Dash로 화면 전환을 하지 않고 해당 키워드를 이용하여 검색 쿼리만 Dash로 전달한다. 검색 결과는 Dash for OS X이 아니라 Dash for iOS에 나타나게 된다.](dash-atom-search.png)

이렇게 활용할 경우, 아이패드는 API 전용 뷰어로 변모하고, 맥에서는 온전히 프로그래밍에 집중 할 수 있게 된다. 특히, 맥북을 이용하여 밖에서 작업을 할 경우에 그 활용도가 배가 된다. 본인은 12인치 맥북은 물론, 27인치 모니터를 갖고 있는 맥 미니에서 작업할 때도 Dash for iOS는 항상 실행시켜두고 아이패드를 API 전용 모니터로 활용한다. 이는 마치 디지털 API 책을 옆에 두고 작업하는 느낌이고, 직접 경험해본 바 개발자 생산성 향상에도 상당히 좋다.

Dash for OS X는 기본적으로 무료 앱이다. 가격은 $24.99로 측정되어 있지만, 돈을 지불하지 않아도 모든 기능을 기한 제약 없이 사용할 수 있다. 다만, 사용 도중에 무작위로 8초간 도큐먼트가 표시되지 않고 타이머가 돌아간다. 개발사는 결제를 유도하기 위한 레깅(lagging)이라고 밝히고 있다.

![lagging 페이지의 모습. 8초간 기다리는게 생각보다 지루하다. 윈도우가 포커스를 잃으면 카운트도 멈추기 때문에 계속 보고 있어야 한다.](dash-lagging-page.png)

Dash for iOS는 $9.99에 판매되고 있다. Dash for iOS를 위에 설명한 방식으로 OS X 버전과 연동하여 사용할 경우, OS X 버전이 무료 버전이라도 결제를 위한 lagging은 나타나지 않는다. 정확히는, OS X 버전에 lagging 페이지가 떠 있는 상태라도 연동된 iOS 버전에는 잘 표시된다.

### 그 밖에 뛰어난 점들

#### 서브 목차

Dash는 단순히 문서들만 나열 해주는 것이 아니라, 그 문서 내의 서브 목차를 분석하여 왼쪽 하단에 띄워준다. 이 서브 목차는 보통 메소드/함수의 리스트, 또는 가이드의 서브 섹션으로 구성되기 때문에 문서를 탐색하는 시간을 월등히 줄여준다.

![](dash-subcontent.png)

#### 정말 다양한 도큐먼트들

API 도큐먼트들의 다양함도 놀랍지만, Cheat Sheets라는 어플리케이션들 단축키/사용 가이드, 각종 팁 등을 모아둔 항목들도 놀랍다. 잘 찾아보면 가려운 곳을 딱 긁어주는 Cheat Sheet들이 있는데, 본인의 경우 HTTP Status Code, Markdown, Bash Shortcuts, Vim이 그러했다. 매번 ’아.. 이게 뭐였더라..’하면서 구글에 ‘markdown code block’ 이런 식으로 검색하던, 바로 그런 내용들이다.

![](dash-various-docsets.png)

이 외에도 코드 Snippets (변수 지정도 가능하다), 동기화, Annotation, 검색 시 Stackoverflow와 Google 검색 내용 포함 등의 뛰어난 기능들이 존재한다. 자세한 내용은 공식 [User Guide](https://kapeli.com/dash_guide)를 참고하자.