# var / let / const 와 TDZ (Temporal Dead Zone)

세 키워드의 차이는 **스코프 / 재할당 / 호이스팅 동작** 세 축으로 정리된다.

| 키워드 | 스코프 | 재선언 | 재할당 | 호이스팅 |
|---|---|---|---|---|
| `var` | 함수 | O | O | 선언+`undefined` 초기화 |
| `let` | 블록 `{}` | X | O | 선언만 (TDZ) |
| `const` | 블록 `{}` | X | X | 선언만 (TDZ) |

## 블록 스코프 vs 함수 스코프

```js
if (true) {
  var a = 1;
  let b = 2;
}
console.log(a); // 1     ← var는 if 블록을 무시
console.log(b); // ReferenceError: b is not defined
```

`var`는 `function` 단위로만 갇히고, `let`/`const`는 `{}` 안에 갇힌다.

## TDZ (Temporal Dead Zone)

`let`/`const`도 호이스팅은 되지만 **선언 줄에 도달하기 전까지 접근 불가**.
이 "선언 전 구간"을 TDZ라고 한다.

```js
console.log(x); // ReferenceError (TDZ)
let x = 10;

console.log(y); // undefined  ← var는 TDZ가 없음
var y = 10;
```

핵심: `let`/`const`는 "변수가 없는 것"이 아니라 "있긴 한데 만지면 에러" 상태다.

## const는 "재할당 금지"이지 "불변"이 아니다

```js
const arr = [1, 2];
arr.push(3);     // ✅ OK — arr 자체를 다시 할당하는 게 아님
arr = [9];       // ❌ TypeError — 재할당 금지

const obj = { a: 1 };
obj.a = 2;       // ✅ OK
```

객체/배열의 내부 값을 진짜 못 바꾸게 하려면 `Object.freeze()` 또는 TS의 `readonly`.

## 헷갈렸던 포인트

- `let`/`const`도 호이스팅은 **된다**. 단지 TDZ 때문에 "선언 전 사용"이 막힐 뿐.
- `for (var i = 0; ...)` 안의 `i`는 함수 전체에 살아있어서, 비동기 콜백에서 마지막 값 하나만 잡힘. `let`을 쓰면 매 반복마다 새 바인딩이라 해결됨.

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
```

## 요약

기본은 `const`, 재할당이 필요하면 `let`, `var`는 안 쓴다.

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/let
