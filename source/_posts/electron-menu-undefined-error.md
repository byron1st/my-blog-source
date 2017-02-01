---
title: Electron의 어플리케이션 메뉴 접근 시 undefined 에러 발생 문제
tags:
  - Electron
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-06-22 10:42:18

---

![](error-screenshot.png)

> Error: Attempting to call a function in a renderer window that has been closed or released. Function provided here: undefined.

<!--more-->

[DM4REVA](/dm4reva/index.html) 버전 1.0.1에서 발생하던 에러이다. 에러 내용은 이미 종료된 Renderer 프로세스의 함수를 부르려고 시도했다는 것이다. 에러가 발생하게 된 단계는 다음과 같다.

1.  Renderer 프로세스가 2개 이상 (정확히는 Renderer 윈도우) 떠 있는 상태에서 1개를 종료한다. (윈도우를 닫는다.)
2.  어플리케이션 메뉴(macOS의 경우 상단바에 위치한 메뉴)의 항목을 선택한다.
3.  에러 발생.

원인은 매우 간단하다.

```js
const menu = Menu.buildFromTemplate(template);
Menu.setApplicationMenu(menu);
```

어플리케이션 메뉴를 생성하는 코드(위 코드 참조)를 Renderer 프로세스에 집어넣어서 그렇다. 이 코드를 Main 프로세스를 생성하는 코드(예를 들면, `main.js` )로 옮기면 해결된다.

상식적으로 생각해보면, 어플리케이션 메뉴는 Main 프로세스와 함께 하는 것이 옳다. 윈도우와 함께 따라다니는 컨텍스트 메뉴가 아니기 때문에 그렇다. 조금만 생각해보면 상식적인 건데, [Electron의 API 문서에 나온 예제](http://electron.atom.io/docs/api/menu/)에 Renderer 프로세스에서 컨텍스트 메뉴를 정의하는 것만 나와있어서 실수한거라고 스스로 위안 삼아본다. [DM4REVA](/dm4reva/index.html)에서는 위의 코드를 `ready` 이벤트 콜백 함수에 삽입함으로써 해결했다.

```js
import {app, BrowserWindow, ipcMain, dialog, Menu} from 'electron'
...
app.on('ready', () => {
  const menu = Menu.buildFromTemplate(mainmenu)
  Menu.setApplicationMenu(menu)
  ...
})
```
