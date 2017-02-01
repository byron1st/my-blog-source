---
title: 자바스크립트 코드의 실행
tags:
  - ES6
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-06-21 09:35:26

---

자바스크립트는 실행 중에 많은 것이 결정되는 스크립트 언어다. 그렇기 때문에 자바스크립트의 코드가 어떻게 실행되는지 아는 것은 매우 중요하다. 자바스크립트는 ECMAScript 표준을 따르는 언어이기 때문에, ECMAScript에 묘사되어 있는 코드의 실행에 관련된 내용을 학습하면, 자바스크립트 코드가 실제로 어떻게 실행되는지 알 수 있다.

ECMAScript는 코드의 실행과 관련해서 Edition 3(이하 ES3)과 Edition 5(이하 ES5), 6(이하 ES6)의 내용이 많이 다르다. 사실 구체적으로 작동하는 원리는 크게 다르지 않지만 용어가 많이 다르기 때문에 표준을 읽을 때 혼동이 오기 쉽다. 대표적으로 ES3에서 활성 객체(Activation Object)는 ES5, 6에서는 실행 문맥(Execution Context)[^1]의 `VariableEnvironment` 컴포넌트와 매우 유사하다. 이러한 변화는 내부 함수와 클로져 개념을 구현하고, 함수형 프로그래밍 언어를 지원하기 위함으로, Environment frames model 이라고 칭하고 있다[^2].<!--more-->

## 실행 문맥(Execution Context)

> An _execution context_ is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation.[^3]

실행 문맥은 실행시 코드 실행(evaluation)을 추적하기 위해 사용되는 명세 장치(device)이다. ECMAScript를 따르는 프로그래밍 언어들의 인터프리터들은 이 실행 문맥을 실제적으로 구현하여 코드를 실행시키게 된다. 자바스크립트 또한 예외는 아니다.

실행 문맥은, 그 단어의 의미에서 짐작할 수 있듯이, 대상이 되는 코드를 실행시키기 위한 각종 정보들을 담고 있다. 한번에 1개의 실행 문맥이 로드 되어 코드를 실행시키며, 실행 문맥들은 스택(stack) 형태의 LIFO 데이터 구조에 저장된다. Java Virtual Machine의 Stack과 Frame을 상상하면 이해하기가 수월하다.

![스택에 실행 문맥이 저장되는 모습. 가장 아래에는 global 실행 문맥이 있다.](execution-stack.png)

ES6에 따르면, 실행 문맥은 여러 상태 컴포넌트(State Component)들로 구성되어 있는데, 코드를 실행시키기 위한 실행 문맥은 최소한 `code evaluation state`, `Function`, `Realm`, `LexicalEnvironment`, `VariableEnvironment` 상태 컴포넌트들을 포함해야 한다. 자바스크립트에서 상태 컴포넌트들은 보통 객체 형태(식별자(identifier)와 그 식별자가 가리키는 값)로 구현된다.

![실행 문맥의 개괄적인 구조](execution-context.png)

실행 문맥을 구성하는 상태 컴포넌트들 중 가장 핵심은 `LexicalEnvironment`와 `VariableEnvironment`이다. 이들은 실행에 필요한 지역변수, arguments 객체, 외부 실행 문맥으로의 참조 등을 식별자들의 집합으로 기록한다. `LexicalEnvironment`와 `VariableEnvironment`는 사실상 같은 것으로 `with` 표현을 사용할 때를 제외하고는 둘의 값이 같다. Dmitry Soshnikov는 `with` 표현 자체가 deprecated 되고 있으므로(그의 예상과는 다르게 ES6에서도 살아남았지만) 둘은 사실상 같은 취급을 해도 큰 무리는 없다. 본 글에서는 이하 `VariableEnvironment`로 통일하도록 하겠다.

## Lexical Environment

> A _Lexical Environment_ is a specification type used to define the association of _Identifiers_ to specific variables and functions based upon the lexical nesting structure of ECMAScript code.[^3]

Lexical Environment은 명세 타입 중 하나로, 식별자들과 이들이 가리키는 특정 값들(변수들, 함수들)의 조합이다. 즉, Key-Value 형태로 조합된 바인딩(binding)들의 집합이라고 쉽게 생각하면 된다. 실행 문맥에서 `LexicalEnvironment`와 `VariableEnvironment` 컴포넌트가 바로 Lexical Environment 타입의 컴포넌트들이다.

![Lexical Environment의 구조](lexical-environment.png)

Lexical Environment은 1개의 Environment Record과 외부 Lexical Environment으로의 참조(이하 `outer`)로 구성되어 있다. Environment Record에는 지역 변수값, arguments 객체 등이 기록되고, `outer` 참조를 통해 Lexical Environment 사이의 계층 구조(내부 함수 등)를 구현한다. Environment Record는 크게 2가지 타입을 갖고 있다. 하나는 Declarative Environment Record로 엄밀히 말하면 ES3의 활성 객체는 바로 이 Declarative Environment Record와 가장 유사하다[^2]. 다른 하나는 Object Environment Record로 전역 문맥(global context)과 `with` 표현 내에 등장하는 변수들과 함수들을 기록한다.

Environment Record는 3가지 종류가 있는데, Function Environment Record, Global Environment Record, 그리고 Module Environment Record 이다. Function/Module Environment Record는 Declarative Environment Record 타입이며, Global Environment Record는 Declarative/Object 두 타입을 결합하여 구성된다.

가장 친숙하고 가장 자주 사용될 Function Environment Record는 추가로 `this` 바인딩을 제공한다. (단, arrow function이 아닐 경우에만) 또한, `super`에 대한 바인딩도 제공한다.

![함수의 실행 문맥 구조](execution-context-rev.png)

## 스코프 체인과 Identifier resolution

변수의 값을 찾는건 엄밀히 말해 식별자(Identifier)가 가리키는 값을 찾는 행위이며, 이를 Identifier resolution 이라고 부른다. Identifier resolution은 기존 ES3에서는 `[[scope]]` 프로퍼티가 가리키는 리스트(스코프 체인)를 통해 수행되었다. 하지만 ES5, 6에서 Identifier resolution은 Lexical Environment의 `outer` 값을 통해 수행된다.

ES6에 따르면, 현재 Lexical Environment의 Environment Record 안의 값들을 찾고, 만약 존재하지 않으면 `outer`가 참조하고 있는 Lexical Environment의 Environment Record 안의 값들을 찾는다. 그리고 이 과정이 값을 찾을 때까지 재귀적으로 호출된다. 즉, 기존의 스코프 체인 개념은 `outer` 체인으로 대체되었다.

### `[[scope]]` 프로퍼티와 `outer` 참조값

새로운 Lexical Environment가 생성되고, 코드 범위 내의 변수들을 Environment Record에 기록할 때(함수 호이스팅이 일어나는 지점), 함수 선언을 마주할 경우, 함수 객체가 생성되고, 함수 객체 내부에는 `[[scope]]` 프로퍼티가 생성된다[^4]. 이 `[[scope]]` 프로퍼티는 현재 Lexical Environment, 즉, `VariableEnvironment` 를 참조하게 된다. 이후 이 함수가 호출되면, 이 함수를 실행시키기 위한 새로운 실행 문맥과 Lexical Environment가 만들어지고, 이 Lexical Environment의 `outer` 값은 이 함수 객체의 `[[scope]]` 값을 그대로 할당한다.

![함수의 실행 모습](scope.png)

코드 실행 후 실행 문맥이 종료 되더라도, 함수 객체 내의 `[[scope]]`가 참조하는 `VariableEnvironment`는 파괴되지 않고 남아있기 때문에,(`[[scope]]`이 참조하고 있기 때문에 가비지 컬렉팅 되지 않고 남아있는다.) 이는 자바스크립트의 클로져가 작동할 수 있게끔 하는 원리가 된다.

## 참고자료

본인의 미숙한 글 외에도 관련하여 찾아볼 수 있는 좋은 자료들이 많다.

*   “[ECMA–262–5 in detail](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-2-lexical-environments-ecmascript-implementation/#identifier-resolution).” by Dmitry Soshnikov

    *   정말 자세히 잘 설명된 글이다. 예제도 소스를 통해 잘 설명 되었고, ES3와의 변경점에 대해서도 간략하게나마 중간중간에 언급하고 있다.

*   “[Understanding JavaScript Closures](https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/)” by Angus Croll

    *   자바스크립트에서 클로져가 작동하는 원리를 설명한 글로, 깔끔하게 핵심만 잘 설명이 되었다. 클로져에 대한 잘못된 인식과 좋은 예제가 많으니, 읽어보면 좋은 글이다.

*   송형주, 고현준, _인사이드 자바스크립트_. 한빛미디어, 2014.

    *   이해하기 쉽게 그림과 함께 설명된 좋은 책이다. 다만 ES3 기준으로 설명이 되어 있어 다소 용어와 개념이 다르다. 하지만 ES3도 ES5,6과 근본 원리는 통하기 때문에 봐두면 이해하기 좋다.


* * *

[^1]: 서지수님의 글에서 실행 문맥으로 번역. (서지수. (2014). _JavaScript: Scope 이해_. Available: [http://www.nextree.co.kr/p7363/](http://www.nextree.co.kr/p7363/))
[^2]: Dmitry Soshnikov. (2010). _ECMA–262–5 in detail. Chapter 3.1\. Lexical environments: Common Theory_. Available: [http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/)
[^3]: _ECMAScript 2015 Language Specification_, ECMA–262, 2015\.
[^4]: Angus Croll. (2010). _Understanding JavaScript Closures_. Available: [https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/](https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/)