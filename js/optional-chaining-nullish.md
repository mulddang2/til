# 옵셔널 체이닝 / Nullish Coalescing (`?.` / `??`)

두 연산자 모두 `null`이나 `undefined`를 다룰 때 코드를 간결하게 만들어 준다.

- `?.` — 중간 값이 `null`/`undefined`이면 **거기서 멈추고 `undefined` 반환**
- `??` — 왼쪽 값이 `null`/`undefined`일 때만 **오른쪽 값으로 대체**

## 옵셔널 체이닝 (`?.`)

### 기본 사용

```js
const user = null;

// 기존 방식
const name = user && user.profile && user.profile.name;

// 옵셔널 체이닝
const name = user?.profile?.name;
console.log(name); // undefined — 에러 없이 그냥 빠져나옴
```

중간에 `null` 또는 `undefined`를 만나면 TypeError 없이 `undefined`를 돌려준다.

### 메서드 호출

```js
const user = { getName: () => "짱구" };

user.getName?.();      // "짱구"
user.getAge?.();       // undefined — 메서드가 없어도 에러 안 남
```

### 배열 요소 접근

```js
const users = null;

users?.[0]?.name; // undefined
```

### 실전 예시

```js
const res = {
  data: {
    user: { name: "둘리", address: null },
  },
};

const city = res.data?.user?.address?.city;
console.log(city); // undefined
```

서버 응답처럼 **중첩이 깊고 일부 필드가 없을 수 있는 구조**에서 특히 유용하다.

## Nullish Coalescing (`??`)

### 기본 사용

```js
const input = null;
const name = input ?? "익명";
console.log(name); // "익명"
```

`input`이 `null`이나 `undefined`일 때만 오른쪽 값을 쓴다.

### `||`와 차이점

`||`는 **falsy 값 전체**(`0`, `""`, `false`, `NaN` 포함)를 대체한다.  
`??`는 **오직 `null`/`undefined`만** 대체한다.

```js
const score = 0;

score || 100;  // 100 — 0이 falsy라서 대체됨 (의도와 다를 수 있음)
score ?? 100;  // 0  — 0은 null/undefined가 아니라서 그대로
```

숫자 `0`이나 빈 문자열 `""`도 **유효한 값으로 취급**하고 싶을 때는 `??`를 써야 한다.

### 기본값 설정

```js
function greet(name) {
  const displayName = name ?? "또치";
  return `안녕, ${displayName}!`;
}

greet("짱구"); // "안녕, 짱구!"
greet(null);   // "안녕, 또치!"
greet(0);      // "안녕, 0!" — 0도 유효한 값으로 처리
```

## 두 연산자 같이 쓰기

```js
const user = null;

const city = user?.address?.city ?? "서울";
console.log(city); // "서울"
```

`?.`로 안전하게 파고 들어간 뒤, 결과가 `undefined`이면 `??`로 기본값을 준다. 자주 나오는 패턴.

## 비교 표

| 연산자 | 대체 조건               | 예시                    |
| ------ | ----------------------- | ----------------------- |
| `\|\|`  | falsy 전체 (`0`, `""`, `null`, `undefined`, `false`, `NaN`) | `0 \|\| 1` → `1` |
| `??`   | `null` / `undefined` 만 | `0 ?? 1` → `0`          |
| `?.`   | 접근 대상이 `null`/`undefined` 이면 멈춤 | `null?.x` → `undefined` |

## 헷갈렸던 포인트

- **`?.`의 결과는 항상 `undefined`**, `null`이 아니다. `null?.x`도 `undefined` 반환.
- **`??`는 `||`의 대체가 아님**. 숫자나 빈 문자열을 유효한 값으로 봐야 하는 상황이면 반드시 `??`.
- **`?.`는 단락 평가(short-circuit)**라 뒤 표현식이 아예 실행되지 않는다.
  ```js
  let count = 0;
  null?.foo(count++); // count는 여전히 0
  ```
- **할당에는 못 쓴다**. `obj?.x = 1` 같은 건 SyntaxError.
- `??`와 `&&`, `||`를 괄호 없이 섞으면 SyntaxError.
  ```js
  a || b ?? c   // ❌ SyntaxError
  (a || b) ?? c // ✅
  ```

## 요약

- `?.` — 중간에 null/undefined 있으면 **조용히 undefined 반환**, TypeError 방지
- `??` — null/undefined일 때만 **기본값 적용**, `||`와 달리 0이나 `""` 건드리지 않음
- 두 연산자를 조합하면 **깊은 중첩 + 기본값** 처리를 한 줄로 해결 가능

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Optional_chaining
