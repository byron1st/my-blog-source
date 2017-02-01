---
title: VSCode에서 Electron의 메인 프로세스 디버그하기
date: 2017-01-09 23:45:33
tags:
- Electron
- VSCode
categories:
- SW-Development
- Programming

---
<kbd>F5</kbd>로 눌러서 바로 디버깅을 실행하고, breakpoint를 찍어서 원하는 지점에서 멈춰놓고 스택 등을 살펴보는 작업은 Eclipse 같은 IDE를 통해 Java 등을 개발하던 사람들에게는 자연스러운 일이다. Electron 개발에서도 이런 작업 흐름을 자연스럽게 할 수 없을까 하여 조사를 좀 해보았다. ~~더 이상 `console.log`는 그만… ㅜㅜ~~

<!--more-->

## Visual Studio Code

[Visual Studio Code](https://code.visualstudio.com/)(이하 VSCode)는 마이크로소프트에서 Electron 기반으로 제작한 텍스트 에디터이다. 경쟁 도구들로 흔히 Sublime Text, Atom 등이 언급되지만, 이 도구들에 비해 월등히 많은 기능들을 번들링한 상태로 나온 도구라 IDE와 텍스트 에디터 중간 지점 쯤에 위치한 개발 도구라고 보면 되겠다. VSCode는 태생 자체가 JavaScript 지원(정확히는 Node.js와 TypeScript)을 최우선으로 하고 있기 때문에, Node.js를 기반으로 하고 있는 Electron 개발을 위한 준IDE급 도구로써 손색이 없다. 특히, TypeScript 개발을 위해 디버깅에서 Source Maps를 잘 지원해주는데, 덕분에 Babel을 통해 변경한 ES6 코드들을 대상으로도 breakpoint 등을 잘 지원해준다.

### Electron 프로그램을 디버깅 모드로 실행

[TypeScript, Electron, Gulp 개발 환경 구축](http://byron1st.pe.kr/?p=527)

VSCode의 디버깅 모드는 위의 글에서도 간단히 소개한 바 있다. 디버깅 모드는 <kbd>cmd+shift+D</kbd>를 누르면 진입한다. 처음 실행할 경우, `.vscode` 폴더 내에 `launch.json` 파일을 만들고 기본 설정을 적는다.

```json launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "프로그램 시작",
      "program": "${workspaceRoot}/compiled/main/main.js",
      "cwd": "${workspaceRoot}",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron",
      "preLaunchTask": "test",
      "sourceMaps": true,
      "outFiles": [
        "${workspaceRoot}/compiled/**/*.js"
      ]
    }
  ]
}
```

위의 `launch.json`은 내가 [MyWorkshop: Stock](/myworkshop-stock/index.html) 프로그램에서 사용하는 설정이다. 우선 프로그램 타입(`type`)은 node로 설정하고, `program` 항목에는 Electron이 가장 먼저 실행시키는 파일(기본은 `index.js` 파일이다.)의 경로를 잡아준다. 그리고 `runtimeExecutable` 항목에는 `node_modules` 폴더 내의 Electron을 잡아준다. 즉, “node 타입으로 해당 프로젝트를 실행(launch)하는데, 다만 node를 Electron의 node로 실행하겠다”라는 의미가 되겠다.

이외의 항목들에 대한 설명은 다음과 같다.

* `preLaunchTask` : VSCode는 `tasks.json`을 통해 Task Runner들을 설정한다. 나같은 경우, npm 의 스크립트들을 설정해두었다. 그 중 `test` 스크립트는 ES6/React 파일들을 Babel을 이용하여 ES5 파일로 변환해주는 과정이 포함된 Gulp task이다. 그렇기 때문에 디버깅 모드 실행에 앞서 이 Gulp task를 실행해주게된다.
* `sourceMaps`: Babel로 변환된 ES5 파일에 직접 breakpoint를 찍는 매우 난해한 행동을 하기 싫다면, Source Maps를 통해 ES6 소스코드 파일과 변환된 ES5 파일들을 매핑해주어야 한다.
* `outFiles`: 만약 Babel을 통해 변환된 ES5 파일들이 동일한 폴더 내에 없다면, 그 파일들이 위치한 경로를 표시해주어야 한다. 예전에는 `outDir` 이라는 이름으로 폴더 경로를 설정하였지만, 지금은 glob 패턴으로 파일들의 경로를 표시해주어야 한다.

### Source Maps 생성을 위한 Gulp Task

Source Maps은 1) 변환된 `xx.js` 파일 내의 마지막 줄에 매핑 정보를 추가하는 방법과 2) 따로 `xx.js.map` 파일을 생성하는 방법이 있다. VSCode는 둘 다를 모두 지원하지만, 나는 2번 방법을 선택했다. 그 이유는;

* 변환된 파일을 나중에 패키징할 때 그대로 카피하기 때문에, 불필요한 Source Maps 정보가 파일 내에 남는 것이 싫고,
* 소스코드 파일의 길이가 길 경우, VSCode가 해당 파일을 읽는데 많은 시간이 걸린다. 실수로 클릭하면 짜증 대박이다. 이는 Electron 기반 모든 도구들(Atom 포함)의 고질병인데, 한줄에 매우 길이가 긴 텍스트가 있는 파일을 로드할 때 엄청 버벅거린다.

Gulp를 이용해서 Source Maps 파일을 생성하기 위해서는 [gulp-sourcemaps](https://www.npmjs.com/package/gulp-sourcemaps) 패키지가 필요하다.

```js gulpfile.js
const gulp = require('gulp')
const sourcemaps = require('gulp-sourcemaps')
const babel = require('gulp-babel')

const JS_FILE_LIST = ['src/**/*.js', 'src/**/*.jsx']

gulp.task('compile:js', () => {
  return gulp.src(JS_FILE_LIST)
    .pipe(sourcemaps.init())
      .pipe(babel({
        'presets': ['es2015', 'react']
      }))
    .pipe(sourcemaps.write('../maps', {includeContent: false, sourceRoot: '../src'}))
    .pipe(gulp.dest(COMPILED_DEST))
})
```

Source Maps 파일들의 생성은 gulp-babel 패키지를 이용해서 변환하기 전에, 우선 초기화(`init()` 함수를 호출)를 해주고, Babel로 변환을 수행한 후, `write()` 함수를 통해 Source Maps 정보를 기록하는 형태로 이루어진다. 이때, 첫번째 인자로 문자열을 넣으면 출력 경로로 인식하여, 해당 경로에 `xx.js.map` 파일들을 생성한다(첫번째 인자로 문자열을 넣지 않으면, 변환된 파일 마지막에 해당 정보를 append 한다.). 두번째 인자는 옵션인데, 이에 대한 설명을 하기 전에 생성된 `xx.js.map` 파일 내용을 먼저 살펴보자.

```js app.mode.js.map
{
  ...
  "sources":["main/app.mode.js"],
  "file":"../../src/main/app.mode.js",
  "sourceRoot":"../../src"
  ...
}
```

VSCode의 디버깅 모드는 여기서 `file` 의 경로를 통해 매핑 위치를 찾는다. `sourceRoot` 가  `sources` 앞에 prepend 되어서 `file` 경로값이 결정이 되기 때문에, 이를 잘 고려해서 `sourceRoot` 옵션을 넣어주어야 한다. 이게 경로 기준이 다소 애매해서 나는 그냥 일단 한번 해보고 직접 파일을 열어서 살펴본 후 수정해주었다. ~~(경로를 잘못 지정해서 얼마나 헤맸던가..)~~

### 실행!

자, Source Maps 파일들도 준비가 되었고 `launch.json` 파일도 준비되었다면, ES6으로 작성된 (Babel로 변환되기 전인) 코드에서 적당한 위치에 breakpoint를 잡고 <kbd>F5</kbd>를 눌러보자.

![VSCode에서 메인 프로세스를 디버그](debug-electron-on-vscode.png)

짜잔. breakpoint에서 잘 멈추고, 옆에 변수값들과 호출 스택들이 잘 뜬다. 상단에는 Eclipse에서 보던 친숙한 디버깅 메뉴들도 준비되어 있다. 이로서 Electron의 메인 프로세스를 디버깅할 준비가 되었다.

랜더러 프로세스의 경우, 나는 이 도구보다는 [Devtron](http://electron.atom.io/devtron/)과 [React Developer Tools](https://github.com/facebook/react-devtools)를 이용한다. 이에 대한 포스팅도 따로 준비해보도록 하겠다.
