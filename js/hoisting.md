# 호이스팅 (Hoisting)

자바스크립트 엔진은 코드를 실행하기 전에 **선언**을 스코프 맨 위로 끌어올린다.
이 동작을 호이스팅이라 한다.

## 무엇이 호이스팅되나

| 대상 | 호이스팅 | 초기화 |
|---|---|---|
| `var` 변수 | O | `undefined`로 초기화 |
| `let` / `const` | O | 초기화 안 됨 (TDZ) |
| `function` 선언식 | O | 함수 전체가 올라감 |
| `function` 표현식 (`const fn = function...`) | 변수만 | 값(함수)은 안 올라감 |

## var — 선언 + undefined 초기화

```js
console.log(name); // undefined  ← 에러 아님
var name = "짱구";
console.log(name); // "짱구"
```

엔진이 실제로 처리하는 방식:

```js
var name;          // 선언이 위로 올라감
console.log(name); // undefined
name = "짱구";     // 할당은 제자리에 남음
```

## function 선언식 — 함수 전체가 올라감

```js
greet(); // "안녕!" ← 선언 전에 호출해도 동작

function greet() {
  console.log("안녕!");
}
```

함수 선언식은 선언과 본문이 통째로 올라가서, 파일 어디서든 호출 가능.

## function 표현식 — 변수만 올라감

```js
greet(); // TypeError: greet is not a function

var greet = function () {
  console.log("안녕!");
};
```

`var greet`는 호이스팅되지만 `undefined` 상태라 함수가 아님 → TypeError.

`const` / `let`으로 선언하면 TDZ 때문에 ReferenceError로 더 명확하게 실패한다.

## let / const — 호이스팅 되지만 TDZ

```js
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 1;
```

`let`/`const`도 호이스팅은 된다. 단지 선언 줄에 도달하기 전까지는 TDZ 구간이라 접근이 막힌다.

## 클래스도 호이스팅된다

```js
const p = new Person(); // ReferenceError (TDZ)
class Person {}
```

클래스 선언도 `let`/`const`처럼 호이스팅되지만 TDZ가 있다.

## 헷갈렸던 포인트

- `var`의 `undefined` 호이스팅은 **버그처럼 보이는 동작을 만든다**. "선언 전인데 왜 에러가 안 나지?"라는 질문의 원인.
- 함수 선언식과 함수 표현식의 호이스팅 차이는 면접 단골. "둘 다 호이스팅되냐"고 물으면 "둘 다 되는데 결과가 다르다"가 정확한 답.
- 코드 품질을 위해 **쓰기 전에 선언하는 습관**이 호이스팅 관련 혼란을 없애준다.

참고: https://developer.mozilla.org/ko/docs/Glossary/Hoisting
