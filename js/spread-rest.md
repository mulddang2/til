# spread / rest (`...`)

같은 점 세 개(`...`)지만 **위치에 따라 의미가 정반대**다.

- **spread**: 모여있는 값을 **펼쳐서** 흩뿌린다 (오른쪽, 인자, 리터럴 안)
- **rest**: 흩어진 값을 **모아서** 하나로 묶는다 (왼쪽, 함수 파라미터, 구조 분해)

## spread — 펼치기

### 배열 펼치기

```js
const a = [1, 2, 3];
const b = [0, ...a, 4];
console.log(b); // [0, 1, 2, 3, 4]
```

### 객체 펼치기 (ES2018+)

```js
const base = { name: "짱구", age: 5 };
const updated = { ...base, age: 6 };
console.log(updated); // { name: "짱구", age: 6 }
```

같은 키가 있으면 **뒤에 오는 값이 이긴다**. 위에서는 `age: 5`가 `age: 6`으로 덮였다.

### 함수 인자로 펼치기

```js
const nums = [1, 2, 3];
Math.max(...nums); // 3
```

`Math.max([1, 2, 3])`은 NaN 나온다 — 배열을 하나의 인자로 받아버리니까.

## rest — 모으기

### 함수 파라미터

```js
function sum(...nums) {
  return nums.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3); // 6
```

`arguments` 객체와 달리 **진짜 배열**이라 `reduce`, `map` 같은 게 바로 된다.

다른 파라미터와 같이 쓸 때는 **항상 마지막**.

```js
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(" & ")}`;
}

greet("안녕", "짱구", "둘리"); // "안녕, 짱구 & 둘리"
```

### 구조 분해에서

```js
const [first, ...others] = [1, 2, 3, 4];
console.log(first);  // 1
console.log(others); // [2, 3, 4]

const { name, ...rest } = { name: "둘리", age: 5, job: "공룡" };
console.log(rest); // { age: 5, job: "공룡" }
```

## spread vs rest 한눈 비교

| 구분  | 위치              | 역할     | 예시                           |
| ----- | ----------------- | -------- | ------------------------------ |
| spread | 값/리터럴 안     | 펼치기   | `f(...arr)`, `[...a, ...b]`    |
| rest  | 선언/구조 분해 좌측 | 모으기 | `function f(...args)`, `[a, ...r]` |

## 얕은 복사라는 점에 주의

spread로 객체/배열을 복사하면 **1단계만 새로 만들어진다**.

```js
const original = { user: { name: "짱구" } };
const copy = { ...original };

copy.user.name = "둘리";
console.log(original.user.name); // "둘리" — 중첩 객체는 같이 바뀜
```

깊은 복사가 필요하면 `structuredClone(original)` 사용.

## 헷갈렸던 포인트

- **rest는 마지막에만 한 번**. `[...a, b]`(spread)는 되지만 `[...r, b]`(rest로 받기)는 안 된다.
- **spread는 iterable에만 동작**. 일반 객체를 `[...obj]`로 펼치려면 TypeError. 대신 `Object.values(obj)` 등을 거쳐야 한다. (객체 spread `{...obj}`는 별개 문법)
- **함수 호출 시 `...arr`는 위치 인자로 펼침**. `f(...[1, 2, 3])`은 `f(1, 2, 3)`과 같다.
- **state 업데이트에서 자주 쓰임**. React에서 `setUser({ ...user, age: 6 })`처럼 불변성을 지키면서 일부만 바꿀 때.

## 요약

- `...` 위치로 spread/rest 구분: **오른쪽이면 펼침, 왼쪽이면 모음**
- 객체 spread는 **나중 키가 이긴다** → 부분 업데이트 패턴
- spread 복사는 **얕은 복사** → 중첩까지 필요하면 `structuredClone`
- rest 파라미터는 진짜 배열이라 배열 메서드 바로 가능

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax
