---
title: JavaScript에서 Array 객체의 reduce 함수의 특징
tags:
  - JavaScript
categories:
  - SW-Development
  - Programming
date: 2016-06-27 01:33:44

---

`reduce()` 함수는 `Array` 객체의 `prototype`에 이미 정의되어 있는 메소드이다. `reduce()` 함수는 배열의 값들을 1개의 값으로 ’감소(reduce)’시키는 함수로, 가장 대표적인 사용예는 다음과 같다.

```js
[1,2,3].reduce((previous, current) => previous + current)
// print 6 (= 1 + 2 + 3)
```

<!--more-->

위의 사용예에서 `reduce()` 함수는 1, 2, 3 값을 갖고 있는 배열을 6 이라는 값으로 ‘감소’ 시켰다. 이 함수에 대한 자세한 정의 및 예제는 아래 링크를 참조하자.

[MDN: Array#Reduce](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

나는 이 함수를 이용해서 object들로 구성된 배열에서 배열 내의 모든 object들이 갖고 있는 프로퍼티들의 값을 더하려고 했다. 예를 들면, `[{a:1, b:2}, {a:3, b:4}, {a:2, b:1}]` 배열에서 배열 구성원들의 a 프로퍼티 값을 모두 더하려 했다. 이를 하기 위해 콜백 함수를 짜다가 몇차례 의도치 않은 결과를 몇번 맛보고는, 이 함수의 작동 원리가 약간의 특징을 갖고 있음을 알게 되었다. 내가 겪었던 실패는 예를 들면 다음 그림과 같다.

![첫번째 실패한 예제](array-reduce-fail1.png)

숫자 배열을 더하듯이 평범하게 콜백 함수를 넣어주었지만, 결과는 NaN. 결론부터 얘기하자면, 이 함수가 의도대로 작동하게 하기 위해서는 다음과 같이 콜백함수를 작성해주어야 한다.

![두번째 성공한 예제](array-reduce-success1.png)

왜 이런 차이점이 발생하는지를 설명하기 전에 다음과 같이 `reduce()` 함수의 특징을 설명해보자.

*   **초기값 인자(콜백 함수 뒤에 받는 optional 인자)의 유무와 관계없이 공통적인 특징**

    *   콜백함수의 previous 인자(이하 `prev`)는 앞서 수행된 콜백함수의 반환값이다.

*   **초기값 인자가 없을 때**

    *   배열 index는 1부터 시작한다. 즉, index 값이 1일 때부터 콜백함수를 호출한다.
    *   index 값이 1일 때, `prev`값은 배열의 첫번째 값(index가 0인 값)이고, 콜백함수의 current 인자(이하 `curr`)는 배열의 두번째 값(index가 1인 값)이다.
    *   배열의 길이가 1일 경우, 콜백함수를 호출하지 않고 배열의 첫번째 값을 바로 반환한다.

*   **초기값 인자가 있을 때**

    *   배열의 index는 0부터 시작한다.
    *   index 값이 0일 때, `prev`는 초기값 인자의 값이 되고, `curr`는 배열의 첫번째 값이 된다.
    *   배열의 길이가 1이어도, 콜백함수를 호출하여 값을 반환한다.

이 규칙들을 잘 생각하고, 첫번째 실패한 예제를 다시 보자. 첫번째 예제에서는 초기값 인자가 없다. 그래서 index 가 1일 때부터 콜백함수를 호출하는데, 이 경우 `prev` 값은 `{a: 1, b:2}`인 object이고, `curr` 값은 `{a:3, b:4}` 인 object이다. 첫번째 콜백은 잘 작동한다. 첫번째 콜백은 1+3 값인 4를 반환한다. 문제는 두번째 콜백부터이다. 두번째 콜백 때, `prev` 값은 number 타입인 4가 된다. 그리고 콜백함수 내의 `prev.a`에서 `4.a` 값을 호출하려고 시도한다. 위의 크롬 콘솔창에서는 에러를 뱉지 않고 `NaN`만 출력하고 끝났지만, `Electron`(정확히는 `Node`) 환경에서는 `4.a` 부분에서 정확히 에러를 던진다.

이렇듯, 배열이 숫자로 구성되어 있지 않을 경우, `reduce()` 함수는 의도치 않은 결과를 내놓을 수 있다. 그렇기 때문에, `reduce()` 함수를 의도대로 사용하기 위해서는 안전하게 초기값 인자를 넣어주는 것이 좋다. 두번째 성공한 예제에서 보면, 초기값 인자로 0을 넣어준 덕분에, 콜백함수는 index가 0일 때부터 호출되었고, `prev` 에는 첫 콜백함수 호출부터 항상 number 타입의 값이 들어가기 때문에 에러없이 작동한다.