# 리터럴 타입 & as const (Literal Types & as const)

타입 자리에 `string`, `number` 같은 큰 타입 말고 **딱 그 값 하나**를 적을 수 있다.
이걸 리터럴 타입(literal type)이라고 한다. `as const`는 이 리터럴 타입을 자동으로 끌어내는 도구다.

## 리터럴 타입 — 값 자체가 타입

```ts
let status: "loading" = "loading";
status = "error"; // ❌ "loading"만 허용
```

문자열뿐 아니라 숫자, 불리언도 된다.

```ts
type Dice = 1 | 2 | 3 | 4 | 5 | 6;
type Answer = true | false;
```

혼자 쓰면 거의 쓸모가 없고, **유니온과 묶을 때** 진가가 나온다.
정해진 값만 받게 막아주는 enum 비슷한 역할을 한다.

```ts
type Direction = "up" | "down" | "left" | "right";

function move(dir: Direction) {
  // ...
}

move("up"); // OK
move("U"); // ❌ 오타를 컴파일 단계에서 잡는다
```

## 타입 추론은 기본적으로 "넓힌다" (widening)

여기서 헷갈리는 지점이 나온다. `const`로 선언하면 리터럴 타입,
`let`으로 선언하면 넓은 타입으로 추론된다.

```ts
const a = "up"; // 타입: "up"  (리터럴)
let b = "up"; // 타입: string  (넓어짐)
```

`let b`는 나중에 값이 바뀔 수 있으니 TypeScript가 알아서 `string`으로 넓힌다.
이걸 타입 넓히기(widening)라고 한다.

문제는 객체다. 객체는 `const`로 선언해도 **속성값은 넓혀진다**.

```ts
const config = { method: "GET" };
// 타입: { method: string }  ← "GET"이 아니라 string

function request(method: "GET" | "POST") {}
request(config.method); // ❌ string은 "GET" | "POST"에 못 들어감
```

객체 자체는 `const`라 재할당이 안 될 뿐, 속성은 바꿀 수 있어서 그렇다.

## as const — 전부 리터럴로 고정

`as const`를 붙이면 그 값을 **가장 좁은 리터럴 타입으로, 그리고 readonly로** 고정한다.

```ts
const config = { method: "GET" } as const;
// 타입: { readonly method: "GET" }

request(config.method); // OK
```

배열에 붙이면 `readonly` 튜플이 된다.

```ts
const arr1 = [1, 2, 3]; // number[]
const arr2 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

## 활용: 객체에서 유니온 타입 뽑아내기

`as const`가 가장 빛나는 패턴이다. 상수 객체를 만들고 거기서 타입을 끌어낸다.

```ts
const ROLES = {
  admin: "admin",
  user: "user",
  guest: "guest",
} as const;

// 값들의 유니온 타입을 뽑는다
type Role = (typeof ROLES)[keyof typeof ROLES];
// "admin" | "user" | "guest"

function setRole(role: Role) {}
setRole(ROLES.admin); // OK
```

값(런타임)과 타입(컴파일 타임)을 한 곳에서 관리할 수 있어서 enum 대신 많이 쓴다.

## 비교표

| | `as const` 없음 | `as const` 있음 |
|---|---|---|
| `const s = "GET"` | `"GET"` | `"GET"` |
| `const o = { m: "GET" }` | `{ m: string }` | `{ readonly m: "GET" }` |
| `const a = [1, 2]` | `number[]` | `readonly [1, 2]` |
| 속성 수정 | 가능 | ❌ readonly |

## 헷갈렸던 포인트

- **`const`라고 다 리터럴이 아니다.** 원시값을 `const`로 직접 담으면 리터럴이지만,
  객체·배열은 `const`여도 속성값이 넓혀진다. 이때 `as const`가 필요하다.

- **`as const`는 readonly까지 같이 붙인다.** 단순히 타입만 좁히는 게 아니라
  값 변경 자체를 막는다. 그래서 상수 테이블에 딱 맞다.

- **`as const`는 런타임 코드가 아니다.** 컴파일되면 사라지는 타입 단언일 뿐,
  실제 객체를 동결(`Object.freeze`)하지는 않는다. 런타임 불변성은 별개다.

- **`(typeof OBJ)[keyof typeof OBJ]` 패턴.** 처음 보면 외계어 같은데,
  "객체 값들을 유니온으로 뽑기"라고 외워두면 두고두고 쓴다.

## 요약

- 리터럴 타입 = 값 하나가 곧 타입, 유니온과 묶어 "정해진 값만" 받게 한다
- 추론은 기본적으로 넓혀진다(widening) — `let`, 객체 속성은 `string`이 된다
- `as const` = 가장 좁은 리터럴 + readonly로 고정
- 상수 객체 + `as const` + `typeof`로 enum 없이 유니온 타입을 뽑는다
- `as const`는 타입 단언일 뿐, 런타임 동결은 아니다

참고: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
