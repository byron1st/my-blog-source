---
title: JavaScript의 non-arrow function과 arrow function의 차이점
tags:
  - ES6
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-05-08 14:22:56

---

JavaScript의 문법을 규정하는 표준이라고 할 수 있는 ECMAScript의 최신판인 ECMAScript 2015(이하 ES6)에는 arrow function이 새롭게 추가되었다. arrow function은 함수를 정의하는데 있어, 다음과 같이 정의하는 것을 말한다.

```js
() => console.log('Hello, world.')
```

이 함수는 다음과 같다.

```js
function () {
    console.log('Hello, world.')
}
```

<!--more-->

arrow function을 처음 보았을 때, 나는 사실 arrow function이 기존의 function 표현(이하 non-arrow function)의 별칭(alias)인 줄 알았다. 즉, 의미 관점에서 둘이 동일한 표현으로 이해하고 있었다. 하지만, Stackoverflow에 React 관련하여 [질문글](http://stackoverflow.com/questions/36527253/why-is-this-undefined-when-i-bundled-the-jsx-file-react-js-using-babel-prese)을 올렸다가 다음과 같은 답변을 받으면서, 저 둘이 의미가 동일한 별칭 관계가 아님을 깨달았다.

> You can’t use fat-arrow functions just because they are “cool and convenient”– they have different semantics than normal functions. Specifically, they share a lexical `this` with their containing scope, instead of creating a new `this`– and in a React component which will be in a module, the outer `this` will be `undefined`.
> 
> 당신은 단순히 “멋지고 편하다”는 이유로 arrow function들을 사용할 수 없습니다. – 그들(arrow function)은 일반 함수(non arrow function)들과 다른 의미를 갖고 있습니다. 특히, 그들(arrow function)은 새로운 `this`를 생성하기 보다는, 그들의 `this`를 그들을 포함하고 있는 범위(scope)와 공유합니다. – 그리고 모듈에 있는 React 컴포넌트에서는 외부의 `this`는 `undefined`가 될 것입니다.

arrow function과 non-arrow function의 주된(그리고 거의 유일한) 차이점은, 위의 답변에서도 말하듯이, `this`에 있다. 이에 관해서 아래 링크의 ‘What’s `this`?’ 항목에서 자세한 내용을 볼 수 있었다.

[ES6 In Depth: Arrow functions](https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/)

요약하자면, ES6 전에 사용되던 일반적인 non-arrow function에서 `this` 값은 일반적으로 다음과 같았다. (자세한 내용은 [여기](http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)를 참고하자.)

1.  Object의 property로서 정의될 때는 Object가 곧 `this`가 되었다.
2.  Global 범위(scope)에서 사용될 때는 `window`, 또는 `undefined`가 되었다.

하지만 arrow function의 `this`는 이 arrow function을 감싸고 있는(enclosing) 범위(scope)의 `this`를 그대로 상속한다.[^1] 위의 링크에서 나오는 다음 예제를 보자.

```js
{
    ...
    addAll: function addAll(pieces) {
        var self = this
        _.each(pieces, function (piece) {
            self.add(piece)
        })
    },
    ...
}
```

이 예제에서 `each` 함수의 인자로 넘겨진 non-arrow function의 `this`는 `window`, 혹은 `undefined` 였다. (위의 2번을 보자) 그렇기에 이 함수를 감싸고 있는 `addAll` 함수의 `this`를 사용하기 위해서는 `self`라는 변수에 이 `this`를 저장한 후 클로져 특성을 사용해서 넘겨주어야 했다. 하지만, ES6의 arrow function을 사용할 경우, 다음과 같이 쓸 수 있다.

```js
{
    ...
    addAll: function addAll(pieces) {
        _.each(pieces, piece => this.add(piece))
    },
    ...
}
```

`each` 함수에서 인자로 넘겨진 `piece => this.add(piece)` 함수에서 `this`는 본인을 감싸고 있는 `addAll` 함수의 `this`로부터 그대로 상속받는다.

이러한 차이점 때문에, 위의 링크에서는 non-arrow function과 arrow function을 사용하는 기준을 다음과 같이 제시하고 있다.

*   non-arrow function 사용: Object의 property로서 함수를 정의할 경우에 사용. (함수의 `this`가 Object를 잘 가르키므로.)
*   arrow function 사용: 그 외 모든 상황에서 사용.

arrow function과 non-arrow function의 이 차이점을 잘 고려하여 필요한 곳에 잘 사용하는 것이 중요하다. 둘은 의미적으로 다른 부분이 있는, 분명히 서로 다른 문법이므로 고민없이 섞어 쓰다가 버그가 발생할 경우, 이를 찾기 매우 어려워 질 수 있다. 그러므로 주의할 필요가 있다.

p.s. 이 외에도 작은 차이점으로서, arrow function은 `arguments` 오브젝트를 갖지 않는다.

* * *

[^1]: 위의 [Stackoverflow 질문글](http://stackoverflow.com/questions/36527253/why-is-this-undefined-when-i-bundled-the-jsx-file-react-js-using-babel-prese)에서 내가 겪었던 오류가 바로 이 것과 관련된 오류였다. 나는 React 컴포넌트 내의 `render` 함수 안에서 `this`를 불렀는데, 이 때 `render` 함수를 arrow function으로 정의했었다. 이 경우 arrow function의 this는 이 arrow function을 감싸고 있는(enclosing) 외부로부터 상속받고, React 컴포넌트의 외부는 undefined이다.(만약 html에 임베드 되는 JavaScript 코드를 짜는 중이었으면 window 오브젝트가 됬을 것이다.) 그래서 `undefined.props....` 이 실행되다 에러를 뱉었었다. 이 arrow function을 non-arrow function으로 교체해주는 것으로 에러를 해결했었다.