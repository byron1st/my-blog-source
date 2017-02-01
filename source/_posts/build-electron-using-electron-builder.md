---
title: electron-builder로 Electron 앱 간단 빌드하기
tags:
  - Electron
  - Gulp
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-12-22 21:13:00

---

일전에 gulp를 이용하여 macOS용 Electron 앱을 빌드하는 방법을 글로 남긴바 있다 ({% post_link build-electron-app-using-gulp %}). 그 방법은 Electron 홈페이지에 제시되어 있는, Electron 앱을 패키징 하는 가장 기본적인 방법을 gulp를 이용하여 자동화한 것이었다. 이번에는 [electron-builder](https://github.com/electron-userland/electron-builder) 라는 좋은 3rd party 도구를 이용하여 간단히 빌드하는 방법을 적어보았다. 본인도 아직 여러 옵션들을 조정해가며 사용하지는 않고, default 설정을 최대한 이용하는 수준에서 간단히 활용하고 있다. 그럼에도 멀티 플랫폼 지원, DMG 파일 제작, icon 자동 설정 등의 편리한 점이 많으니, Electron 앱을 제작한다면, 한번쯤 살펴볼만한 좋은 도구이다.<!--more-->

## 준비

### electron-builder 설치

우선 electron-builder를 설치해야 한다. electron-builder는 여타 JavaScript 도구들처럼 NPM을 통해 설치할 수 있다. `npm install electron-builder --save-dev` 명령어를 이용하여 설치해주자.

이 외에 electron-builder는 빌드 과정 중에 7zip-bin 패키지를 이용하는데 electron-builder 외에 추가로 설치를 해주어야 한다. 7zip-bin 패키지는 macOS와 Windows 버전이 따로 있으며, 각각 해당하는 OS에서만 설치가 된다. 그렇기 때문에 다음과 같이 `optionalDependencies`로 선언하여 `package.json` 파일에 넣어주어야 한다.

```json
"optionalDependencies": {
  "7zip-bin-mac": "^1.x.x",
  "7zip-bin-win": "^2.x.x"
}
```

`optionalDependencies`로 선언될 경우, `npm install`을 통해 한꺼번에 설치하더라도 OS에 맞는 패키지만 설치된다.

### 아이콘 파일 준비

electron-builder는 default로 `/build` 폴더에서 아이콘과 관련 이미지 파일을 찾는다. macOS 용 아이콘은 `icon.icns`로, Windows 용 아이콘은 `icon.ico` 라는 파일명으로 저장되어야 한다. (Linux의 경우, macOS용 아이콘에서 자동 생성된다.) 또한 `background.png` 는 macOS 용 DMG 파일 생성 시 배경에 들어간 그림 파일이다. 이 파일들을 준비해서 `/build` 폴더 밑에 넣어주자.

### 패키징 될 코드 준비

electron-builder는 default로 `/app` 폴더 안에 있는 파일 및 코드들을 `app.asar` 로 패키징한다. 그렇기 때문에 `/app` 폴더로 `package.json` 파일을 비롯하여 실행에 필요한 소스코드들과 외부 라이브러리들 파일들을 복사해와야 한다. `node` 모듈들의 경우, `package.json`에 정의되어 있으면, 빌드 중에 자동으로 `/app/node_modules` 폴더에 넣어준다.

이 작업은 수동으로 하기엔 번거롭기 때문에 본인은 다음 과정들을 gulp task로 작성하여 사용한다. 우선, 1) `/app` 폴더 내부를 깔끔히 삭제한 후, 2) 3rd party 라이브러리들을 복사한다. 예를 들면, `./bower_components` 를 `./app/bower_components` 로 복사하는 형태다. gulp에서 복사는 아주 쉽다.

```js
const PACKAGE_DEST = 'app'
...
gulp.task('copy:bower', ['del:package'], () => {
  return gulp.src(['bower_components/**/*'])
    .pipe(gulp.dest(PACKAGE_DEST + '/bower_components'))
})
...
```

그리고 3) 컴파일 된 소스코드를 복사하는데, 미리 복사한 라이브러리들과 폴더의 계층 구조가 헝크러지지 않도록 주의한다.

여기까지 완료되면, `/app` 폴더 내에는 `package.json` 파일을 제외하고는 모두 준비가 된 상태일 것이다. `package.json` 파일은 루트 폴더의 `package.json` 파일을 읽어들인 후 필요한 내용만(예를 들면, `devDependencies`는 삭제) 복사한다. 단, `name`, `version`, `description`, `author` 는 반드시 있어야 한다. 예를 들면 아래와 같은 식이다.

```js gulpfile.js
const PACKAGE_DEST = 'app'
const PREPACKAGE = ['copy:compiled', 'copy:public']
...
gulp.task('copy:compiled', ...
gulp.task('copy:public', ...
...
gulp.task('package', PREPACKAGE, () => {
  let devPkg = JSON.parse(fs.readFileSync('package.json'))
  let prodPkg = {
    name: devPkg.name,
    version: devPkg.version,
    description: devPkg.description,
    main: 'dist/src/app/main.js',
    author: devPkg.author,
    license: devPkg.license,
    dependencies: devPkg.dependencies
  }
  fs.writeFileSync(PACKAGE_DEST + '/package.json', JSON.stringify(prodPkg))
})
```

여기까지 오면, 이제 `/app` 폴더 내에는 필요한 파일들이 모두 준비된 상태가 된다. 이 과정은 [MyWorkshop: Stock](/myworkshop-stock/index.html) 앱 만들 때 사용한 것으로, 시각적인 빌드 구조 다이어그램은 다음 링크의 '빌드 구조' 항목을 참고하자

[코드관점(Code Viewpoint) 아키텍처 문서](https://github.com/byron1st/my-workshop-stock-doc/blob/master/code-viewpoint.md)

## 실행

실행에 앞서 다음과 같은 기본 설정을 **루트 폴더**의 `package.json` 파일 내에 추가해야 한다. 그 외에도 풍부한 옵션들이 있는데, 이 [사이트](https://github.com/electron-userland/electron-builder/wiki/Options)를 참고하자.

```json package.json
{
...
  "build": {
    "appId": "byron1st.my-workshop-stock",
    "copyright": "Copyright © 2016 Hwi Ahn",
    "mac": {
      "category": "public.app-category.productivity" // electron-builder 홈페이지 참고.
    },
    "win": {
      "iconUrl": (URL...)
    }
  },
...
}
```

그리고 다음 script를 역시 `package.json` 에 추가하자.

```json package.json
{
...
  "scripts": {
    ...
    "dist": "gulp package &amp;&amp; build"
  },
...
}
```

본인 같은 경우, 앞서 필요한 파일들을 복사하는 과정을 gulp의 `package` task에 할당을 해두었기 때문에, `build` 명령어에 앞서 gulp의 해당 task를 호출해준다.

이제 `npm run dist` 를 실행해주면, `/dist` 폴더 내에 해당 OS에 맞는 파일들을 빌드해준다.  macOS에서 실행할 경우, DMG, xx.app.zip, xx.app 파일 해서 총 3개가 생성된다.

![... 테스트는 하나도 안 만들고 그냥 휙휙 개발해서 테스트가 0 뜨는게 매우 부끄럽...](build-processing.png)

![](build-result.png)

지금까지 소개한 내용은 electron-builder의 가장 기본적인 내용들이다. 이 외에도 자동 업데이트, Code signing 등 실제 상용 앱 제작에도 도움이 되는 좋은 기능들이 많다. 또한, `/app` 폴더에 따로 asar로 패키징하는 파일 및 코드들을 모으고 `package.json` 을 루트 폴더의 것과 분리함으로써 좀 더 관리가 깔끔하다는 장점이 있다. 그렇기 때문에 Electron 공식 홈페이지에서도 Electron 앱의 빌드 도구로 소개하고 있다. 공식 홈페이지에 도큐먼트도 꽤 충실히 되어 있는 편이니 참고하자.

[electron-builder](https://github.com/electron-userland/electron-builder)