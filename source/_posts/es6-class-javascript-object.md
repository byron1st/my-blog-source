---
title: ES6의 class 문법과 기존 JavaScript 객체 정의의 연관성
tags:
  - ES6
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-05-21 06:11:01

---

JavaScript의 최신 표준 문법인 ECMAScript 6(이하 ES6)에서 새로운 class 문법이 소개되었다. 과거 JavaScript는 객체지향으로 사용은 할 수 있었지만, 그 방법이 직관적이지 못했다. 우선 constructor를 함수로 정의하고, 모든 객체에 공통적으로 사용될 변수, 메소드(본 글에서 객체에 제한되어 있는 함수를 ’메소드’라고 부르도록 하겠다.)들은 prototype의 property들로 선언을 해야 했다. 이 모든 과정은 객체지향처럼 보이지 않았고, 직관적이지도 않았으며, 유지보수에도 썩 좋지 않았다.

<!--more-->

이러한 문제점들을 해결하기 위해, ES6에서 class 문법을 새롭게 추가하였다. 이는 JavaScript 언어에 완전히 새로운 모델을 추가하는 것이 아니다. 다만 기존의 객체 정의, 선언을 좀 더 깔끔하고 일관되게 처리할 수 있도록 단순하고 명확한 문법을 제공한 것이다.

> JavaScript class는 ECMAScript 6을 통해 소개되었으며, 기존 prototype 기반의 상속을 보다 명료하게 표현 합니다. Class 문법은 새로운 객체지향 상속 모델을 제공하는 것은 **아닙니다**. JavaScript class는 객체를 생성하고 상속을 다루는데 있어 훨씬 더 단순하고 명확한 문법을 제공합니다. ([MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes))

새로운 문법이 추가된 만큼, 이 새로운 문법으로 정의할 경우, 내부에서 어떻게 객체가 정의되고 생성되는지 궁금하다. 즉, `class` 키워드로 선언할 경우, prototype과의 관계는 어떻게 되는 것인지? 클래스 자체는 무엇인지? `static` 키워드를 붙일 경우, 이게 내부에서 어떻게 표현이 되는 것인지?등이 궁금할 수 있다. 그래서 다음과 같이 정리해보았다.

![JavaScript의 class 키워드 사용과 실제 생성되는 객체의 모습. 내부의 코드는 아래 링크에서 참고.](javascript-class.jpg)

[링크](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/)

`class` 키워드를 통해 정의된 메소드, property 들이 실제로 구현되는 모습은 사실 매우 직관적이다. `constructor` 키워드를 통해 구현된 함수는 자연스럽게 `prototype` 객체의 `constructor`가 되고, `static` 키워드가 없이 선언된 메소드들은 `prototype` 객체의 property로 정의된다. `static` 키워드와 함께 선언된 메소드들은 `Circle` 객체에 직접 property로 정의된다. 그리고 [`class` 자체도 사실 함수이다](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes).

위의 그림을 보면 명확하지만, ES6의 새로운 클래스 문법은 전에도 우리가 이미 하나하나 수동(?)으로 선언해주던 것을 `class`라는 명료한 키워드를 통해 구현해 놓은 것이다. 기능상의 차이는 없고, 프로그래머들의 readability를 향상시켜주는 새로운 키워드라고 볼 수 있겠다.

#### 함께 볼 자료

*   [Classes - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes)
*   [ES6 In Depth: Classes](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/)
