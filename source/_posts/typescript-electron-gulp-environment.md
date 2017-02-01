---
title: 'TypeScript, Electron, Gulp 개발 환경 구축'
tags:
  - Electron
  - JavaScript
  - TypeScript
categories:
  - SW-Development
  - Programming
date: 2016-07-08 00:18:28

---

새롭게 Electron 어플리케이션 개발을 해야할 것이 생겼다. (어느정도 진행이 되면, 이 또한 본 블로그에 공개하도록 하겠다.) 지난 번에 [DM4REVA](/dm4reva/index.html)를 개발하면서, 아쉬웠던 부분은 바로 정적 타입 부분이었다. JavaScript 자체는 동적 타입 언어이지만, 어플리케이션이 좀 커지기 시작하니, 한창 개발 중인 나 조차도 변수, 함수의 인수 등의 타입이 햇갈리기 시작했다. 그 결과, 결국 한참 실행 중일 때 버그가 속출하는 일이 많아졌다. 그 당시에는, 이제와서 정적 타입 시스템을 가져오기도 일이고, [DM4REVA](/dm4reva/index.html)는 개인적으로 그냥 연구에 가볍게 사용하는 도구에 불과해서 그냥 불편한대로 쓰며 나중에 개선하기로 마음을 먹었었다. 하지만 이제 새로운 어플리케이션 개발을 시작하는 입장이니, 이번에는 시작부터 정적 타입 시스템을 적용해보기로 했다.<!--more-->

## 1\. JavaScript를 위한 정적 타입 언어: TypeScript

[TypeScript](https://www.typescriptlang.org)는 Microsoft에서 개발하는 JavaScript 용 정적 타입 언어이다.

> TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.

TypeScript에는 정적 타입과 이를 지원해주는 각종 문법들(함수/클래스 선언시 타입을 결정해주는 문법 등)을 지원하고, 이를 보통 JavaScript 스크립트로 변환해주는 컴파일러를 제공한다. 즉, TypeScript로 작성된 스크립트가 실행되는 것이 아니라, TypeScript 컴파일러(이하 `tsc`)를 통해 변환된 보통 JavaScript 스크립트 파일이 실행되는 것이다. ([Babel](http://babeljs.io)을 상상하면 이해하기 편하다.) `tsc`는 컴파일 시 각종 타입이 제대로 들어가있는지 확인해준다.

TypeScript는 2012년 10월 처음 발표되었고, 그 후 꾸준히 발전하여 현재 버전 1.8(2016년 1월 발표)까지 발표하였다. 최근에는 [Angular 2가 TypeScript를 이용하여 개발되어](https://blogs.msdn.microsoft.com/typescript/2015/03/05/angular-2-built-on-typescript/) 화제를 모으기도 했었다. TypeScript는 Microsoft가 꽤 의욕적으로 지원을 하고 있고, 덕분에 Visual Studio 시리즈(Visual Studio 2015, Visual Studio 2013, Visual Studio Code)에서 거의 IDE 수준으로 지원이 되며, 그 외에도 [Atom 등의 에디터에서 사용 가능한 플러그인들을 지원하고](https://www.typescriptlang.org/#download-links) 있다. TypeScript에 대한 자세한 내용은 [공식 홈페이지를](https://www.typescriptlang.org) 참고하자.

### 개발 환경 설치

TypeScript의 설치는 매우 간편하다. 엄밀히는 TypeScript의 컴파일러, 즉, `tsc`, 를 설치하는 것이다. `tsc`는 `npm`을 통해서 설치할 수 있다 (`npm install -g typescript`). 또는, [Visual Studio Code](https://code.visualstudio.com)를 설치하면 안에 내장되어 있다. 본인이 해당 프로그램들이 없어 확인은 못했지만, 다른 Visual Studio들 (2013, 2015) 또한 안에 내장하고 있을 것으로 생각된다. 본인은 Visual Studio Code를 선택했다.

TypeScript 컴파일러는 `tsconfig.json` 이라는 파일을 설정파일로 이용한다. Babel의 `.babelrc` 파일, 혹은 npm 프로젝트의 `package.json` 파일과 유사한 역할을 한다. `tsconfig.json` 파일은 프로젝트 폴더의 루트에 위치하며, Visual Studio Code는 이 경우 TypeScript 프로젝트로 인식한다. `tsconfig.json` 파일의 구체적인 내용은 [여기서](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) 확인하자.

#### Typings 설치

[Typings](https://github.com/typings/typings)는 TypeScript definition manager로 유명 JavaScript 프레임워크(라이브러리)들의 타입, 함수 등의 정의들을 관리할 수 있도록 도와준다. Visual Studio Code는 Typings를 이용하여 TypeScript에 대한 IntelliSense 기능을 제공해준다. 기본 단축키는 <kbd>F12</kbd>이다.

Typings에는 다양한 TypeScript definition들이 존재하는데, 이들을 검색, 설치하는 방법은 [공식 사이트](https://github.com/typings/typings)의 설명을 참고하자. 참고로 Typings가 없다면, Visual Studio Code에서는 IntelliSense 기능을 제공하지 못할 뿐 아니라, node, electron 등과 같은 외부 라이브러리에 대해, 해당 타입들을 찾을 수 없다며 에디터에서 빨간 줄을 그어대어 상당히 불편하다.(아래 그림 참조) 그러니 Typings 설치를 꼭 해주도록 하자. 본인은 현재 `node`, `electron`, `react`, `react-dom`을 설치해놓고 있다.

![](main-ts.jpg)

### Flow

[Flow](https://flowtype.org)는 JavaScript의 진영의 막강한 서포터인 Facebook에서 개발한 JavaScript를 위한 정적 타입 언어이다. null 처리 부분에서 다른 점을 제외하고는 TypeScript와 아주 유사한 문법 구조를 갖고 있다. TypeScript에 비해 좀 더 쉽게 기존의 JavaScript 코드들에 적용할 수 있다는 장점이 있다. 하지만 아직 개발된 역사가 짧고, TypeScript에 비해 안정화가 덜 되었다는 평가가 많다.(타입 검사가 불안정하다는 보고가 많다.) 하지만 왕성이 개발되고 있는 만큼, 주시하고 있을 필요는 있다. TypeScript와 Flow의 비교에 관해선 다음과 같은 좋은 블로그들이 많으니 참고하자.

*   [Flow vs TypeScript - 왜 나는 아직 타입스크립트를 쓰는가](http://blog.seulgi.kim/2016/01/flow-vs-typescript.html) by Seulgi Kim
*   [Flow vs TypeScript: Type Systems for JavaScript](http://djcordhose.github.io/flow-vs-typescript/2016_hhjs.html#/) by Oliver Zeigermann
*   [TypeScript vs Flow - results from our investigation](https://www.reddit.com/r/javascript/comments/39cere/typescript_vs_flow_results_from_our_investigation/) by Ismailman

여담인데, Flow는 [Nuclide](https://nuclide.io)라는 IDE를 통해 지원한다. Nuclide는 Facebook에서 개발한 IDE로, React, React Native, Flow 등을 지원한다. Nuclide는 Atom 에디터의 플러그인 형태로 지원되는데, TypeScript를 지원하는 Visual Studio Code 역시 Atom (정확히는 Electron 기반) 기반이다. JavaScript 쪽은 Atom 에디터가 장악해나가는 느낌이다.

## 2\. Gulp 설정

Electron은 TypeScript의 스크립트 파일들 외에도 HTML, CSS 파일들 또한 필요하다. `tsc`만 쓸 경우, HTML이나 CSS 파일을 처리해주기가 힘들기 때문에, Gulp를 이용하여 빌드해주면 편하다. 즉, TypeScript 파일들은 ES5와 호환되는 JavaScript 파일로 변환해주고, HTML이나 CSS 파일들은 적절한 위치에 복사해주면 된다. 나같은 경우, `src` 에 TypeScript, HTML, CSS 파일들을 놓고, Gulp를 이용하여 `src` 폴더의 구조를 그대로 유지한채 `dist` 폴더로 TypeScript 파일들은 변환하고, HTML, CSS 파일들은 복사한다. 이 경우 나중에 Electron 어플리케이션을 배포할 때, 이 `dist` 폴더를 그대로 `resource` 폴더 내로 카피해주면 된다.

Gulp는 [`gulp-typescript`](https://www.npmjs.com/package/gulp-typescript) 모듈을 통해 TypeScript 컴파일러를 사용할 수 있다. TypeScript 컴파일 시 SourceMap 파일들을 생성해준다면, Visual Studio Code에서 디버깅할 때 TypeScript 파일을 참조해주어 편리하다. 하지만, `tsconfig.json` 설정 파일의 `sourceMap` 항목은 `gulp-typescript`에서 지원되지 않는다. 그래서 SourceMap 파일도 함께 생성해주고 싶다면, [`gulp-sourcemaps`](https://www.npmjs.com/package/gulp-sourcemaps) 모듈을 이용해야 한다. 이를 이용한 아주 기초적인 `gulpfile.js`의 모습은 아래와 같다.

```js gulpfile.js
const gulp = require('gulp')
const del = require('del')
const ts = require('gulp-typescript')
const sourcemaps = require('gulp-sourcemaps')
const tsProject = ts.createProject('tsconfig.json')

gulp.task('clean', () =&amp;gt; {
  return del.sync([
    'dist'
  ])
})

gulp.task('copy', () =&amp;gt; {
  return gulp.src([
      './src/view/*.html'
    ])
    .pipe(gulp.dest('./dist/view'))
})

gulp.task('ts', () =&amp;gt; {
  return tsProject.src()
    .pipe(sourcemaps.init())
    .pipe(ts(tsProject)).js
    .pipe(sourcemaps.write('.'))
    .pipe(gulp.dest('./dist'))
})

gulp.task('default', ['clean', 'ts', 'copy'])
```

`ts` Task는 TypeScript 파일을 TypeScript 컴파일러를 통해 변환해주는 역할을 한다. `tsProject`는 `gulp-typescript` 모듈의 `createProject` 함수를 통해 생성되는데, 이 때 `tsconfig.json` 파일을 입력받는다. 물론 Babel 설정처럼 직접 `pipe` 안에 쓰는 방법도 있다. SourceMap 파일 생성을 위해 `sourcemaps.init()`과 `sourcemaps.write()`가 사용된 위치도 확인하자. `write` 인자로 `.`을 넘길 경우, 생성된 `js` 파일과 동일한 폴더 위치에 SourceMap 파일을 생성한다. 이렇게 SourceMap 파일을 따로 생성해야만 Visual Studio Code에서 디버깅할 때 사용할 수 있다.

## Visual Studio Code의 Task Runner를 이용하여 Gulp 변환 후 Electron 실행 및 디버그

Electron 어플리케이션을 실행할 때, 나는 `package.json`에서 `npm start` 부분에 `gulp && ./node_modules/.bin/electron .`을 설정해 놓은 후 Terminal에서 `npm start`를 입력하여 실행하였다.

```json package.json
{
  ...
  "main": "dist/main.js",
  ...
  "script": {
    "test": "...",
    "start": "gulp && ./node_modules/.bin/electron ."
  }
  ...
}
```

하지만, 매번 Terminal에서 이 커맨드를 치는 것은 불편하다. 이를 다른 IDE들처럼 단축키로 실행시키기 위해 Visual Studio Code의 Task Runner 기능을 이용할 수 있다. 만약 Visual Studio Code에서 따로 Task Runner 설정을 하지 않았다면, <kbd>cmd+shift+b</kbd> 를 눌렀을 때, `.vscode` 폴더 내에 `tasks.json` 이라는 설정 파일을 생성시킨다. 이 파일을 아래와 같이 설정할 경우, Terminal을 열고 `npm start`를 입력하던 행위를 <kbd>cmd+shift+b</kbd> 단축키로 해결할 수 있다.

```json tasks.json
{
  ...
  "command": "npm",
  "isShellCommand": true,
  "args": ["start"],
  "showOutput": "always"
}
```

Task Runner와 유사하게, Visual Studio Code에서 디버그 모드는 `.vscode` 폴더 내의 `launch.json` 파일에서 설정할 수 있다. Visual Studio Code에서는 디버그 모드로 `node` 타입, VS Extension 타입 등을 제공하고 있다. Electron을 디버그 하기 위해서는 디버그 모드 중 `node` 타입의 설정을 약간 변경해야 한다. 아래 그림을 참고하면, 디버그 모드 때 node 서버 대신 Electron을 실행하여 디버그할 수 있다.

![launch.json 설정](launch-json.png)

만약 윈도우의 경우라면 `runtimeExecutable` 위치에 `node_modules/electron-prebuilt/dist/electron.exe`을 적어주어보자. (본인이 테스트해보지는 않았다.)

아래 링크는 Visual Studio Code의 디버그 모드에서 Electron을 실행시키는 방법을 자세히 설명하고 있다. (나도 여기를 보고 참고했다.)

*   [A Better Way To Launch Electron from Visual Studio Code](http://www.mylifeforthecode.com/a-better-way-to-launch-electron-from-visual-studio-code/) by Shawn Rakowski

다만, 디버그는 `dist` 폴더의 변환된 JavaScript 파일들을 이용하니, Gulp를 통해 미리 빌드하는 것을 잊지 말자. Breakpoint 또한 변환된 JavaScript 파일에 찍어주어야 작동한다. 다만 변환된 파일에 Breakpoint를 찍더라도, 디버그 모드에서는 SourceMap 파일을 이용하여 변환 전 TypeScript 파일에 해당 위치를 찍어준다.

![](ts-debug-1.png)

![](ts-debug-2.png)

여기까지 설정을 해두면 (많아보이지만, 사실 몇줄 되지 않는다.) Visual Studio Code를 통해 편리하게 Electron을 개발할 준비가 끝난다. <kbd>cmd+shift+B</kbd>를 통해 Gulp로 빌드 후 Electron 어플리케이션 실행까지 수행할 수 있고, Breakpoint를 통해 디버그 모드도 활용할 수 있다. 여기에 [Devtron](http://electron.atom.io/devtron/)까지 깔아두면 든든하다.

개인적으로 여기에 React 설정까지 완료는 하였는데, 분량이 길어져 일단 여기서 줄이도록 한다. React와 TypeScript의 조합 또한 상당히 멋지며, 과거 동적 타입에서 느끼던 불안감을 대부분 해소하며 개발할 수 있는 좋은 장치가 준비되어 있다고 느껴진다. 개인적으로 여기에 Facebook의 Flux Architecture를 적용해보는 것이 목표이다. Facebook은 현재 [`Dispatcher`](https://facebook.github.io/flux/docs/dispatcher.html#content)와 [`Store`](https://facebook.github.io/flux/docs/flux-utils.html#content) 부분까지 본인들의 구현체를 공개하여, Flux Architecture의 프레임워크화에 박차를 가하고 있다. Flux Architecture, React, TypeScript의 조합이라면, 상당히 큰 프로그램도 그 복잡도를 상당히 억제할 수 있을거라 기대된다.