# 조건부 타입 (Conditional Types)

타입 레벨에서 쓰는 삼항 연산자라고 보면 된다.
`T extends U ? X : Y` 꼴로 쓰고, **`T`가 `U`에 할당 가능하면 `X`, 아니면 `Y`** 가 된다.
런타임 `condition ? a : b`와 모양은 똑같은데, 값이 아니라 타입을 고른다는 점만 다르다.

## 기본 문법: `extends ? :`

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"짱구">; // true
type B = IsString<5>;       // false
```

`"짱구"`는 `string`에 할당 가능하니 `true`, `5`는 아니니 `false`가 된다.
여기서 `extends`는 상속이 아니라 **"할당 가능한가(assignable)"** 를 묻는 의미다.

## 제네릭과 같이 쓰기

조건부 타입은 제네릭(generic)과 붙을 때 진짜 쓸모가 생긴다.
입력 타입에 따라 반환 타입이 갈리는 함수 시그니처를 짤 수 있다.

```ts
type ArrayOrSingle<T> = T extends any[] ? T[number] : T;

type C = ArrayOrSingle<string[]>; // string
type D = ArrayOrSingle<number>;   // number
```

`T`가 배열이면 원소 타입을, 아니면 자기 자신을 돌려준다.

## 분배 조건부 타입 (Distributive Conditional Types)

`T`가 **네이키드(naked) 타입 파라미터**이고 유니온이 들어오면, 조건부 타입이
유니온의 각 멤버에 **하나씩 따로** 적용된다. 이걸 분배(distribution)라고 한다.

```ts
type ToArray<T> = T extends any ? T[] : never;

type E = ToArray<string | number>;
// 기대: (string | number)[] 가 아니라
// 실제: string[] | number[]
```

`string | number`가 통째로 들어가는 게 아니라 `string`, `number` 각각에 적용된 뒤
다시 유니온으로 합쳐진다. 이 동작은 유틸리티 타입 `Exclude`, `Extract`의 핵심이기도 하다.

```ts
type MyExclude<T, U> = T extends U ? never : T;

type F = MyExclude<"a" | "b" | "c", "b">; // "a" | "c"
```

## 분배를 막고 싶을 때

분배가 싫으면 `T`를 대괄호로 한 번 감싼다. `[T] extends [U]` 꼴로 쓰면
유니온이 풀리지 않고 통째로 비교된다.

```ts
type IsAny<T> = [T] extends [never] ? "비었음" : "값 있음";

type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type G = ToArrayNonDist<string | number>;
// (string | number)[] — 분배 안 됨
```

## 중첩해서 분기 늘리기

조건부 타입을 이어 붙이면 `else if`처럼 여러 갈래로 나눌 수 있다.

```ts
type TypeName<T> = T extends string
  ? "string"
  : T extends number
  ? "number"
  : T extends boolean
  ? "boolean"
  : "object";

type H = TypeName<true>;   // "boolean"
type I = TypeName<() => void>; // "object"
```

## 비교표

| 표현 | 의미 |
| --- | --- |
| `T extends U ? X : Y` | `T`가 `U`에 할당되면 `X`, 아니면 `Y` |
| `extends` (조건부) | 상속 X, "할당 가능한가" 판정 |
| `T extends any ? ...` | 네이키드 → 유니온 분배 발생 |
| `[T] extends [U] ? ...` | 대괄호로 감싸 분배 차단 |

## 헷갈렸던 포인트

- **`extends`가 두 가지 의미로 쓰인다.** 제네릭 제약(`<T extends string>`)에서는
  "이 범위로 제한"이고, 조건부 타입(`T extends string ? ...`)에서는 "할당 가능한지 판정"이다.
  문맥에 따라 역할이 다르다.

- **분배가 의도치 않게 터진다.** `ToArray<string | number>`가
  `(string | number)[]`일 거라 기대했다가 `string[] | number[]`가 나와서 당황한다.
  네이키드 타입 파라미터 + 유니온이면 무조건 분배된다고 기억하면 된다.

- **`never`도 분배된다 — 그래서 사라진다.** 분배 조건부 타입에 `never`를 넣으면
  순회할 멤버가 없어서 결과가 `never`가 된다. `[T] extends [never]`로 감싸야
  "T가 never인지"를 제대로 검사할 수 있다.

  ```ts
  type Test<T> = T extends string ? "yes" : "no";
  type J = Test<never>; // "yes"도 "no"도 아닌 never
  ```

## 요약

- 조건부 타입은 `T extends U ? X : Y`, 타입 레벨 삼항 연산자다.
- 여기서 `extends`는 상속이 아니라 "할당 가능한가" 판정이다.
- 네이키드 타입 파라미터 + 유니온이면 각 멤버에 분배된다.
- 분배를 막으려면 `[T] extends [U]`로 대괄호를 씌운다.
- `Exclude` / `Extract`가 분배 조건부 타입으로 만들어져 있다.

참고: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
