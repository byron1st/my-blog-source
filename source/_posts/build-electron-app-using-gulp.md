---
title: Gulp로 macOS 용 Electron 앱 빌드하기
tags:
  - Electron
  - Gulp
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-07-14 20:26:12

---

본 글은 [Electron](http://electron.atom.io) 앱 소스코드들에서 자동으로 바로 실행 가능한 Electron 앱을 빌드해주는 Gulp 스크립트를 소개한다.

macOS용 Electron 앱을 빌드하기 위해서는 다음과 같은 과정이 필요하다. ([참고](http://electron.atom.io/docs/tutorial/application-distribution/))

1.  `electron-prebuilt` npm 모듈 내의 `dist` 폴더에 있는 `Electron.app`을 카피한다.
2.  `Electron.app` 내의 `Contents/Resources/app` 폴더 내에 `package.json`을 포함한 모든 JavaScript 소스코드를 복사한다.

<!--more-->

여기까지 완료하면 다음과 같은 구조가 된다.

```
electron/Electron.app/Contents/Resources/app/
├── package.json
├── main.js
└── index.html
```
(출처: 참고 링크)

(Optional) [`asar`](https://github.com/electron/asar) npm 모듈을 이용하여 `app.asar` 파일로 패키징하여 `Contents/Resources/` 폴더에 복사해도 된다. 이 경우, 다음과 같은 폴더 구조가 된다.

```
electron/Electron.app/Contents/Resources/
└── app.asar
```

앱을 리브랜딩하기 위해서는 (이름, 아이콘, 버전명 등을 변경) 다음과 같은 과정을 거친다.

1.  `Electron.app` 이름을 변경한다. (예. `DM4REVA.app`)
2.  `Electron.app/Contents/Info.plist` 내의 `CFBundleDisplayName`, `CFBundleIdentifier`, `CFBundleName`을 변경한다. `Electron Helper` 파일도 변경하려면, [링크](http://electron.atom.io/docs/tutorial/application-distribution/)의 ‘Rebranding with Downloaded Binaries’ 항목을 참고한다. Helper 파일들은 변경하지 않아도 큰 문제는 없다.
3.  아이콘을 변경하려면, 일단 아이콘 파일(`.icns` 파일)을 `Contents/Resources/` 폴더에 카피한다.
4.  `Electron.app/Contents/Info.plist`의 `CFBundleIconFile`을 아이콘 파일 명으로 변경한다.
5.  버전을 변경하려면, `Electron.app/Contents/Info.plist`의 `CFBundleVersion`, `CFBundleShortVersionString` 항목을 변경한다.

이 모든 과정은 꽤 번거롭지만, 어렵지는 않다. 손이 많이 갈 뿐이다. 그래서 이 스탭들을 자동화해줄 수 있는 Gulp 스크립트를 작성해보았다.

## Gulp 스크립트

### 1\. 빌드 폴더 삭제

우선 Gulp Task의 이름은 `package`로, 빌드가 이루어지는 폴더는 `./build` 폴더로 정하였다. `package` Task가 실행될 때 마다, 우선적으로 `./build` 폴더를 비워주어야 한다.

```js
gulp.task('clean:build', () => {
  return del.sync([
    'build/**/*'
  ])
})
```

[`del`](https://www.npmjs.com/package/del)은 폴더/파일을 삭제해주는 편리한 npm 모듈이다.

### 2\. 소스코드 빌드

개발에 Gulp를 사용하였다면, JavaScript 코드들과 HTML, CSS 파일들을 Gulp를 이용해 빌드(정확히는 트랜스파일)하는 것은 익숙할 것이다. 나는 ES6을 따르는 JavaScript 소스코드들은 Babel을 이용해서 변경하고, HTML, CSS 파일은 단순히 목표 폴더로 카피한다. 아래 코드는 JavaScript를 Babel을 이용하여 ES6과 React 코드들을 변경하는 Gulp Task이다. Babel은 [`gulp-babel`](https://www.npmjs.com/package/gulp-babel) npm 모듈을 활용하였다.

```js
gulp.task('pack:compile', ['clean:build'], () => {
  return gulp.src(srcList)
    .pipe(babel({
      'presets': ['es2015', 'react']
    }))
    .pipe(gulp.dest('./build/dist'))
})
```

단순 파일 복사 Task는 `gulp.src`에서 `pipe(gulp.dest())`로 바로 연결하면 되므로, 따로 코드를 설명하지 않겠다.

### 3\. npm 모듈 복사

의존성이 있는 npm 모듈들 또한 복사해주어야 정상적으로 Electron 앱을 작동시킬 수 있다. 여기서 문제는 `node_modules` 폴더에는 개발 시에만 의존적인 모듈들(`package.json` 파일 내의`devDependencies` 항목에 정의된 모듈들)도 섞여있다는 것이다. 나는 아직 여기서 `devDependencies` 들만 따로 분리하는 법을 알지 못하여, `package.json` 을 배포용으로 따로 준비했다. 배포용 `package.json` 파일에는 민감한 정보(개인 코드저장소 라든지) 등을 삭제하고 개발 시에만 의존적인 모듈들도 삭제했다. Gulp Task를 이용하여 이 `package.json` 파일을 `./build` 폴더 내로 카피하고, 다음 쉘 명령어를 실행하여 npm 모듈들을 `./build/node_modules` 폴더에 다시 받아주었다.

```bash
npm --prefix ./build install ./build
```

Gulp Task에서 쉘 명령어를 실행하기 위해서는 [`gulp-shell`](https://www.npmjs.com/package/gulp-shell) 이라는 npm 모듈을 사용했다. 코드는 다음과 같다.

```js
gulp.task('pack:npm', ['clean:build'], () => {
   return gulp.src('*.js', {read: false})
  .pipe(shell([
    'npm --prefix ./build install ./build'
  ]))
})
```

쉘 명령어를 2개 이상 연달아 실행해야 한다면, `shell` 내의 배열에 연속으로 적어주면 된다.

### 4\. asar로 패키징

여기까지 진행되었다면, `./build/dist` 내에 모든 필요한 소스코드 파일들이 빌드되어 있을 것이고, 필요한 npm 모듈들과 `package.json` 파일도 카피되어 있을 것이다. 개인적으로는 앞으로의 편의를 위해 asar 패키징을 하였다. (계속 복사를 해야 하는데, asar로 패키징 하면 `app.asar` 파일만 복사하면 된다)

asar 파일의 이름은 반드시 app 이어야 한다. 그렇지 않으면 Electron 앱이 인식하지 못한다. asar 패키징은 다음 쉘 커맨드를 실행하면 만들수 있다. 다만 그 전에, Global로 asar npm 모듈이 설치되어야 한다.

```bash
npm install -g asar
```

그 후 다음 쉘 커맨드를 이용하자.

```bash
asar pack ./build ./build/app.asar
```

이 쉘 커맨드는 앞서 3번에서 소개했던 `gulp-shell` 모듈을 동일하게 사용하면 손쉽게 실행시킬 수 있다.

### 5\. Electron 바이너리 복사

이미 빌드가 된 Electron 바이너리는 `electron-prebuilt` npm 모듈에 있다. `./node_modules/electron-prebuilt/dist/Electron.app`이 그것이다. 이 파일을 `./build` 파일로 복사하고, 여기에 asar 패키징 파일을 복사하고 각종 리브랜딩 작업을 하면 된다. 우선 복사를 해보자.

처음에는 HTML, CSS 파일 복사와 동일하게 Gulp Task를 이용하여 시도했지만, 앱 내의 복잡한 폴더구조 때문인지 계속 에러를 뱉으며 뻗었다. 그래서 결국 속편하게 `gulp-shell` 모듈의 쉘 커맨드를 이용했다.

```bash
cp -R ./node_modules/electron-prebuilt/dist/Electron.app/. ./build/electron/DM4REVA.app/
```

macOS에서 .app 파일은 폴더로 취급되기 때문에 이러한 복사가 가능하다. 복사와 동시에 앱 이름도 자동으로 리브랜딩 된 것은 덤이다.

### 6\. 리브랜딩 작업

제일 처음에 설명한 리브랜딩 작업을 하기 위해서는 우선 `Info.plist` 파일을 변경해야 한다. `plist` 파일을 읽고 쓰는데에 아주 편리한 npm 모듈이 바로 [`plist`](https://www.npmjs.com/package/plist) 모듈이다. `plist` 모듈로 `plist` 파일을 읽어올 경우, JSON 처럼 읽고 쓸 수 있다. 예를 들면, 다음과 같다.

```js
  let plistObj = plist.parse(fs.readFileSync('./build/electron/DM4REVA.app/Contents/Info.plist').toString())
  plistObj.CFBundleDisplayName = programName
  plistObj.CFBundleIdentifier = identifier
  plistObj.CFBundleName = programName
  plistObj.CFBundleShortVersionString = versionNumber
  plistObj.CFBundleVersion = versionNumber
  plistObj.CFBundleIconFile = iconFile
  fs.writeFileSync('./build/electron/DM4REVA.app/Contents/Info.plist', plist.build(plistObj))
```

위 코드에서 `programName`, `identifier`, `versionNumber`, `iconFile` 변수들에 원하는 값만 잘 저장되어 있다면, 위 코드는 정상적으로 필요한 `plist` 파일을 생성해서 Electron 앱 내에 쓴다. asar 패키징 파일 복사와 함께 쓰면 다음 코드와 같다.

```js
gulp.task('finalize:electron', ['copy:electron'], () => {
  let plistObj = plist.parse(fs.readFileSync('./build/electron/DM4REVA.app/Contents/Info.plist').toString())
  plistObj.CFBundleDisplayName = programName
  plistObj.CFBundleIdentifier = identifier
  plistObj.CFBundleName = programName
  plistObj.CFBundleShortVersionString = versionNumber
  plistObj.CFBundleVersion = versionNumber
  plistObj.CFBundleIconFile = iconFile
  fs.writeFileSync('./build/electron/DM4REVA.app/Contents/Info.plist', plist.build(plistObj))

  return gulp.src([
    './build/app.asar',
    './dm4reva.icns'
  ])
  .pipe(gulp.dest('./build/electron/DM4REVA.app/Contents/Resources'))
})
```

### 7\. 한 개의 Gulp Task로 연결

지금까지 진행했다면 Gulp Task가 여러개 될 것이다. 나의 경우, 8개의 Gulp Task를 작성했다. 이들을 단순히 나열한다면, Gulp는 이 Task들을 동시에 실행하기 때문에, 서로 간의 의존성을 고려하지 못한다. 예를 들면, 모든 Task 들은 반드시 `./build` 폴더를 비워 준 후 실행되어야 할 것이고, 필요한 소스코드와 npm 모듈들이 모두 준비 된 후 asar 패키징 Task가 실행되어야 할 것이다. Gulp에서 `task` 함수를 작성할 때 2번째 인자로 Task 이름을 넣어주면, 해당 Task에 의존하게 된다. 예를 들면, 다음과 같다.

```js
gulp.task('pack:asar', ['pack:compile', 'pack:npm', 'pack:copy', 'pack:view'],() =&gt; {
  return gulp.src('*.js', {read: false})
  .pipe(shell([
    'asar pack ./build ./build/app.asar'
  ]))
})
```

`pack:asar` Task는 asar 패키징을 실행하는 Task이다. 이 Task는 `task` 함수의 2번째 인자인 `['pack:compile', 'pack:npm', 'pack:copy', 'pack:view']` 배열에 의해, 이 배열에 나열된 Task 들이 모두 종료 된 후 시작한다. 이런 식으로 Task 들 사이의 의존관계를 잘 배치하고, 최종적으로 `gulp.task('package', ['finalize:electron'])` 코드를 작성해주면, 완료된다.

```bash
gulp package
```

여기까지 완료되면, 위의 쉘 명령어를 입력하면, 실행 가능한 macOS 용 Electron 앱이 작성된어 `./build` 폴더 내에 위치한다. 내가 Windows나 Linux를 사용하지 않아 해당 OS 용 Electron 앱을 빌드해보지는 못했다. 하지만, 두 버전들 모두 유사한 빌드 과정을 거치기 때문에, 대동소이 할 것으로 생각된다.

[DM4REVA](/dm4reva/index.html) 개발에 사용한 Gulp 스크립트 전문은 다음 링크를 통해 확인해볼 수 있다. Gulp를 이용하여 Electron을 개발하는 개발자들에게 참고가 되길 바란다.

[DM4REVA/gulpfile.js](https://github.com/byron1st/dm4reva/blob/master/gulpfile.js)

## Electron 오픈소스 패키징 도구들

사실 Electron 홈페이지에는 이러한 과정들을 도와주는 오픈소스 패키징 도구들을 소개하고 있다.

*   [electron-builder](https://github.com/electron-userland/electron-builder)
*   [electron-packager](https://github.com/electron-userland/electron-packager)

하지만 설명이 충분치 않고, 본인의 경우, 이 도구들을 쓰는 것은 닭 잡는데 소 잡는 칼을 쓰는 겪이라, 위와 같이 Gulp로 작성해서 사용하고 있다. Electron 상용 앱을 개발 중인 사람들은 위의 두 앱들을 참고해보자.