# 구조 분해 할당 (Destructuring)

배열이나 객체의 값을 **각각의 변수에 한 번에 꺼내** 담는 문법.
반복적인 인덱스/프로퍼티 접근을 줄여준다.

## 배열 구조 분해

```js
const arr = [1, 2, 3];

const [a, b, c] = arr;
console.log(a, b, c); // 1 2 3
```

위치 기준이라 순서가 중요하다. 건너뛰려면 쉼표만 남긴다.

```js
const [, , third] = [1, 2, 3];
console.log(third); // 3
```

## 객체 구조 분해

```js
const user = { name: "짱구", age: 5 };

const { name, age } = user;
console.log(name, age); // "짱구" 5
```

이름(키) 기준이라 순서는 상관없다.

### 이름 바꾸기 (rename)

```js
const { name: nickname } = { name: "둘리" };
console.log(nickname); // "둘리"
// console.log(name);  // ReferenceError — 원래 이름은 안 만들어짐
```

### 기본값

```js
const { age = 0 } = {};
console.log(age); // 0
```

기본값은 값이 `undefined`일 때만 적용된다. `null`은 그대로 `null`.

```js
const { age = 0 } = { age: null };
console.log(age); // null
```

### rename + 기본값 같이 쓰기

```js
const { name: nickname = "익명" } = {};
console.log(nickname); // "익명"
```

## 중첩 구조 분해

```js
const res = { data: { user: { name: "짱구" } } };

const { data: { user: { name } } } = res;
console.log(name); // "짱구"
```

중간 객체가 `undefined`면 터진다 → 옵셔널 체이닝(`?.`) 또는 기본값으로 방어.

```js
const { data: { user: { name } = {} } = {} } = {};
console.log(name); // undefined
```

## rest 패턴 (`...`)

남은 값을 한꺼번에 모은다.

```js
const [head, ...tail] = [1, 2, 3, 4];
console.log(head); // 1
console.log(tail); // [2, 3, 4]

const { name, ...rest } = { name: "둘리", age: 5, job: "공룡" };
console.log(rest); // { age: 5, job: "공룡" }
```

객체에서는 `rest`가 **얕은 복사**라는 점에 주의 (중첩 객체는 참조 공유).

## 함수 파라미터에서

가장 많이 쓰이는 패턴. 옵션 객체를 받을 때 깔끔하다.

```js
function createUser({ name, age = 0, role = "guest" } = {}) {
  return { name, age, role };
}

createUser({ name: "짱구" });
// { name: "짱구", age: 0, role: "guest" }

createUser(); // 인자 없이 호출해도 에러 안 남 (= {} 덕분)
```

`= {}`가 없으면 인자 없이 호출했을 때 `undefined`를 구조 분해하다가 TypeError.

## 헷갈렸던 포인트

- **객체 구조 분해를 한 줄로 시작할 때는 괄호 필요**. 이미 선언된 변수에 할당하는 경우.

  ```js
  let name;
  // { name } = { name: "짱구" };  // ❌ SyntaxError ({ 가 블록으로 해석됨)
  ({ name } = { name: "짱구" });    // ✅
  ```

- **기본값은 `undefined`에만 적용**. `null`, `0`, `""`는 그대로 들어온다.
- **rest는 마지막에만 한 번**. `[a, ...rest, b]` 같은 건 불가.

## 요약

- 위치 기준 → 배열 분해 `[a, b]`
- 이름 기준 → 객체 분해 `{ a, b }`
- 기본값은 `undefined`일 때만 채워짐
- 함수 옵션 객체 받을 땐 `= {}`까지 묶어서 방어

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
