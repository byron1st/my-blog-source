---
title: Markdown을 이용한 간단한 문서화 환경 구축
date: 2017-01-25 16:55:05
tags: 
- Markdown
- Documentation
- SW-Architecture
categories:
- SW-Development
- SW-Engineering

---
[MyWorkshop: Stock](/myworkshop-stock/index.html) 프로젝트의 코드가 어느정도 궤도에 오르면서 기본적인 아키텍처 요소들에 대한 문서화의 필요성을 느끼기 시작했다. 일단 1) 프로그램 코드가 공개되어 있으니, 보는 사람들이 이해할 수 있도록 구조를 설명해주는 최소한의 문서가 필요하고, 2) 코드가 확장 되어 감에 따라 내가 지켜야 할 구조적인 제약사항들을 편하게 기록해두는 것이 필요했다. 나란 놈은 ~~전공이 전공이다보니,~~ 코드가 1,000줄만 넘어가도 일단 논리적 수준의 아키텍처라도 어디다 끄적거려둬야 마음이 편한것이었다. 그래서 다음과 같은 요구사항을 만족시킬 수 있는 환경을 구축하기로 했다.
<!--more-->

1. 변경내역 추적 가능
2. 문서 요소간 링크 가능
3. Rich Text 기반 편집기 짱시룸

나는 워드 기반의 문서화를 참 싫어한다. 우선 변경내역 추적이 어렵고, 다이어그램 붙이기도 어려우며(좀만 커져도 찌그러지고 눌리고..), 문서 요소간 링크를 좀 하려면 일단 한문서에 있어야 한다. (하나의 거대한 아키텍처 문서가 탄생하는 순간이다.) 내가 워드의 기능을 풍부하게 몰라서 그런 것일 수도 있는데, 일단 켜는 순간 배우기가 참 싫다보니 어쩔수 없다.

그렇다보니, 3번을 만족시키기 위해서는 역시 내가 사랑해마지않는 Markdown이 나와주어야 했다.(일전에도 한번 시도해본바 있다.) Markdown을 사용하면, 자연스럽게 1,2,3번이 모두 만족이 되는데, 우선 plain text이기 때문에 변경내역 추적이 아주 쉽고, 문서 요소간 링크도 `[]()` 문법으로 간단히 해결이되며, Rich Text 기반 편집기로부터 아주 멀리 떠나갈 수도 있다.

## 변경내역 추적

변경내역 추적에는 역시 Git이다. 또 때마침 GitHub이 xx.md 파일을 자동으로 파싱해주니 얼마나 좋은가. 유일하게 아쉬운점은 GitHub을 쓰게 되면, GFM (Github-Flavored-Markdown) 을 써야 하고, 이 문법은 MultiMarkdown에서 지원되는 Footnote가 지원이 안된다는 점이다.

GitHub은 README.md 파일을 자동으로 Repository 전면에 파싱해서 띄워주므로, 이 파일에 문서 제목, 개요, 목차 등을 넣으면 좋다. 목차를 넣을 때, 다른 문서들에 대한 링크는 상대경로로 넣어주면 된다.

## 문서 요소간 링크

```markdown logical-viewpoint.md
### <a name="AD-AS-1"></a>AD-AS-1: Client-Server 아키텍처 스타일 적용
Electron은 기본적으로 Client-Server 아키텍처 스타일로 구성되어있다. ...
...
[AD-AS-1](#AD-AS-1)을 적용한다. LC03은 사용자가 요청할 때마다 창을 여러개 늘릴 수도 있고, ...
...
```

`<a name="AD-AS-1"></a>AD-AS-1: ...` 는 `AD-AS-1`라는 anchor를 생성해준다. 그리고 여기로 링크를 하기 위해선 위와같이 Markdown의 링크 문법을 사용하면된다. 다른 문서에서 호출할 때도 아래와같이 동일하다.

```markdown module-viewpoint.md
[AD-LV-1](./logical-viewpoint.md#AD-LV-1)
```

`<a name="...`를 매번 치는 것은 매우 번거로우니 snippet을 하나 만들어두는 것을 잊지 말자. 나는 Dash 앱(리뷰)에서 아래와같이 작은 snippet 하나를 만들어두었다.

```
<a name="__id__"></a>__id__
```

## Markdown 문서 작성 환경

내가 준비한 문서 작성 환경은 3개의 앱으로 구성되어 있다: Sublime Text, Terminal (iTerm2), Marked 2. Sublime Text에서 글을 작성하고, Marked 2에서 프리뷰를 확인하며, Terminal에서 Git을 통해 GitHub으로 퍼블리쉬하는 형태다.

Sublime Text 3는 가볍고 빠르며, 원하는대로 커스터마이즈가 가능한 아주 좋은 텍스트 에디터다. 보통 프로그래밍 도구로 많이 사용되지만, Markdown 지원도 당연히 잘된다. 하지만 그냥 사용하면 왠지 허전하니, 각종 기능으로 무장한 플러그인을 하나 깔아주자.

Package Control에서 Markdown으로 검색을 하면, 가장 유명한 Markdown 플러그인들이 3개 정도 나온다.

![](./sublime-text-markdown-plugins.png)

이 중 Markdown Preview는 문자 그대로 프리뷰를 보여주는데 집중한 플러그인이고, MarkdownEditing과 Markdown Extended가 Markdown 편집 관련 기능을 확장하는데 주력한 플러그인이다. 나는 [MarkdownEditing](https://packagecontrol.io/packages/MarkdownEditing)을 설치했다.

나는 [Marked 2](http://marked2app.com/)라는 앱을 통해 문서 프리뷰를 보기 때문에, 손쉽게 Marked 2를 열 수 있는 플러그인도 추가로 설치했다. [Marked App Menu](https://packagecontrol.io/packages/Marked%20App%20Menu)라는 플러그인으로, Sublime Text의 커맨드 팔레트(<kbd>Cmd+Shift+P</kbd>)에서 Marked를 입력하면 바로 해당 파일이 Marked 2에서 열린다. Marked 2는 여러 기능(각종 문서 통계, 목차, 여러 Markdown 테마 적용 등)을 지원하는 훌륭한 Markdown 프리뷰 앱이지만, 가격이 상당하고 macOS만 있으며 프리뷰’만’ 가능하다는 점이 다소 아쉬운 앱이다.

다 설치하면 아래와같은 문서 작성 환경이 갖춰진다. 왼쪽이 Marked 2이고 오른쪽이 MarkdownEditing 플러그인이 설치된 Sublime Text 3이다.

![](./marked2-sublime-text.png)

마지막으로 Sublime Text 3는 터미널과의 궁합도 좋은데, 터미널에서 Git을 통해 버전관리, 커밋 등을 수행하다가 문서 작성을 바로 하고 싶을 때는 마치 vim으로 문서를 열듯이 터미널에서 Sublime Text 3로 문서를 바로 열 수 있다. 아래 커맨드를 사용할 경우 터미널에서 해당 문서를 바로 열 수 있다.

```bash "" https://www.sublimetext.com/docs/3/osx_command_line.html sublime-text-cli
subl <문서이름>
```

## 결론

간만에 마음에 드는 문서 작성환경이 갖추어져 기분이 좋다. 유일하게 마음에 걸리는 것이 다이어그램 작성을 텍스트로 하지 못해 변경 추적이 쉽지 않는 점이다. 현재는 OmniGraffle로 작성 한 후 이를 jpg로 assets 폴더에 export 한다. mermaid 라이브러리를 활용하거나 dot 언어를 활용하면 좋겠다는 생각은 하지만, 이를 위해서는 추가적인 도구가 필요하니 현 시점에서는 일단 이 정도 수준에서 만족하기로 하자.

이 문서작성 환경에 만족해서 헐레벌떡 만들어본 문서 repository는 아래 링크에서 확인할 수 있다.

https://github.com/byron1st/my-workshop-stock-doc
