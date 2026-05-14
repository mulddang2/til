# 얕은 복사 / 깊은 복사 (Shallow Copy / Deep Copy)

JS에서 객체·배열을 복사할 때 "주소를 복사하냐, 값을 복사하냐"가 핵심이다.

- **얕은 복사(shallow copy)**: 최상위 프로퍼티만 새로 만든다. 중첩 객체는 **참조를 공유**한다.
- **깊은 복사(deep copy)**: 중첩까지 포함해 **모두 새 메모리**에 만든다.

## 참조 타입의 기본 동작

```js
const a = { name: "짱구", score: 100 };
const b = a; // 복사가 아니라 같은 주소를 가리킴

b.name = "둘리";
console.log(a.name); // "둘리" — a도 같이 바뀜
```

`=` 대입은 객체를 복사하지 않는다. 주소(참조)만 넘긴다.

## 얕은 복사 방법

### 1. spread (`...`)

```js
const original = { name: "짱구", age: 5 };
const copy = { ...original };

copy.age = 6;
console.log(original.age); // 5 — 최상위는 독립됨
```

### 2. Object.assign

```js
const copy = Object.assign({}, original);
```

`spread`와 동작이 같다. 최상위만 복사.

### 3. 배열 얕은 복사

```js
const arr = [1, 2, 3];
const copy1 = [...arr];
const copy2 = arr.slice();
const copy3 = Array.from(arr);
```

셋 다 얕은 복사다.

### 얕은 복사의 한계

```js
const original = { user: { name: "짱구" } };
const copy = { ...original };

copy.user.name = "또치";
console.log(original.user.name); // "또치" — 중첩 객체는 참조를 공유함
```

`user` 프로퍼티가 가리키는 객체 자체는 새로 만들어지지 않는다.

## 깊은 복사 방법

### 1. structuredClone (권장)

```js
const original = { user: { name: "짱구" }, scores: [100, 90] };
const copy = structuredClone(original);

copy.user.name = "둘리";
copy.scores.push(80);

console.log(original.user.name); // "짱구" — 독립됨
console.log(original.scores);    // [100, 90]
```

브라우저와 Node.js 17+ 모두 지원. 함수, `Symbol`, DOM 노드는 복사 못 한다.

### 2. JSON 직렬화 (한계 있음)

```js
const copy = JSON.parse(JSON.stringify(original));
```

빠르고 간단하지만 **함수, `undefined`, `Date`, `Map`, `Set`, 순환 참조**는 처리를 못 한다.

```js
const obj = { fn: () => {}, date: new Date(), val: undefined };
const copy = JSON.parse(JSON.stringify(obj));
console.log(copy); // { date: "2026-05-14T..." } — fn, val 사라짐, date는 문자열로 변환
```

### 3. 라이브러리 (lodash `cloneDeep`)

```js
import cloneDeep from "lodash/cloneDeep";
const copy = cloneDeep(original);
```

복잡한 자료구조까지 처리해야 한다면 가장 안전하다.

## 비교 표

| 방법 | 깊이 | 함수 | Date | 순환 참조 |
| --- | --- | --- | --- | --- |
| spread / assign | 얕음 | — | — | — |
| slice / Array.from | 얕음 | — | — | — |
| `structuredClone` | 깊음 | ❌ | ✅ | ✅ |
| JSON 직렬화 | 깊음 | ❌ | ❌ | ❌ |
| lodash `cloneDeep` | 깊음 | ✅ | ✅ | ✅ |

## 헷갈렸던 포인트

- **spread는 1단계만** 새로 만든다. 중첩이 하나라도 있으면 공유된다.
- **`Date`는 JSON 직렬화하면 문자열로 바뀐다.** 받은 쪽에서 `new Date(str)`로 되돌려야 한다.
- **순환 참조 객체를 JSON으로 직렬화하면 TypeError**가 난다. `structuredClone`은 이를 처리한다.
- **React state 업데이트**에서 얕은 복사를 쓸 때, 중첩 객체까지 바꾸려면 단계별로 spread해야 한다.

  ```js
  // ❌ 중첩 객체를 직접 수정 — state가 바뀐 것처럼 보이지 않을 수 있음
  setUser(prev => { prev.profile.name = "또치"; return { ...prev }; });

  // ✅ 중첩도 새 객체로
  setUser(prev => ({ ...prev, profile: { ...prev.profile, name: "또치" } }));
  ```

## 요약

- `=` 대입은 참조 복사 — 같은 객체를 가리킴
- spread / `Object.assign` → 얕은 복사, **중첩은 공유**
- `structuredClone` → 깊은 복사, 함수 제외하고 대부분 처리
- JSON 직렬화 → 빠르지만 함수·`Date`·`undefined` 등 손실
- 깊은 복사가 자주 필요하다면 lodash `cloneDeep` 검토

참고: https://developer.mozilla.org/ko/docs/Web/API/structuredClone
