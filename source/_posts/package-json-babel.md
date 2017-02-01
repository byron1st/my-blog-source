---
title: package.json에서 babel 항목
tags:
  - Babel
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-05-24 06:22:36

---

새로운 기술을 배우며 코딩을 할 때는 boilerplate, template, quickstart tutorial 등을 통해 일단 직접 코딩을 하며 배우는 경우가 많다. 이 경우, 빠르게 핵심 내용을 익힐 수는 있지만, 내용의 자세한 부분을 하나하나 이해하며 넘어가기는 힘들다. 예를 들어, `package.json`의 각 항목 하나하나가 왜 필요한지, 어떻게 configure 해야하는지 모르고, 일단 template에서 제공하는대로 사용하는 경우가 그렇다.

오늘 개인적인 코딩을 진행하다 package.json의 `"babel"` 항목이 있는 것을 발견했다. [mocha](http://mochajs.org)로 테스팅을 할 때, 테스트 코드에서 ES6을 쓰기 위한 설정을 인터넷에서 찾은 후 그것을 그대로 사용하다보니, 얘가 왜 존재해야하는지도 모른체 그냥 적어둔 설정이었다.<!--more-->

```js package.json
{
 ...
  "scripts": {
    "test": "./node_modules/.bin/mocha --compilers js:babel-core/register",
    "start": "./node_modules/.bin/electron ."
  },
  "babel": {
    "presets": [ "es2015" ] // 바로 이 항목.
  },
  ...
  "author": "byron1st",
  "license": "MIT",
  "devDependencies": {
    "babel-cli": "^6.8.0",
    "babel-core": "^6.8.0",
    "babel-preset-es2015": "^6.6.0",
    "chai": "^3.5.0",
    "electron-prebuilt": "^1.1.1",
    "mocha": "^2.5.1"
  },
  "dependencies": {
    ...
  }
}
```

이 부분은 Babel의 global configuration을 위한 부분이다. Babel의 global configuration은 프로젝트 루트 폴더의 `.babelrc` 파일에 정의된다. `.babelrc`에는 Babel의 [option](http://babeljs.io/docs/usage/options/)들에 대한 값을 정의해놓는다. 이 global configuration은 mocha, [gulp](http://gulpjs.com) 등의 도구들이 사용한다. (이 말은 저 도구들이 Babel에 대한 설정을 공유한다는 의미가 된다.) 물론, Webpack 처럼 따로 Babel 설정을 품고 있는 도구들도 있다.

그리고, `package.json` 파일의 `"babel"` 항목은 `.babelrc` 와 동일한 의미를 갖는다. 즉, 사용자는 `.babelrc` 파일에 명시하는 대신, `package.json`의 `"babel"` 항목에 Babel option 들을 명시해줄 수 있다.

즉, 본인의 경우, `mocha --compilers js:babel-core/register`를 통해 테스트를 실행하면, `package.json`의 Babel 설정에 따라 ES6 코드를 변환하고, 정상적으로 테스트를 진행한다. 만약 저 설정이 없을 경우, Babel은 별다른 컴파일을 진행하지 않고, ES6 코드는 해석되지 못한체 에러를 뿜는다. `package.json`에 명시하는 대신 `.babelrc`에 명시해도, (물론) 테스트는 잘 돌아간다. 만약 설정이 두 군데 모두 존재한다면, `.babelrc`를 우선한다. 즉, `.babelrc`이 존재하면 `package.json`의 설정은 무시된다.