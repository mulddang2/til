# 유틸리티 타입 ③ Record / Exclude / Extract

앞에서 본 `Pick`/`Omit`이 **객체 타입을 가공**하는 도구였다면,
이번 세 개는 결이 조금 다르다.
`Record`는 **키-값 형태의 객체 타입을 만들고**, `Exclude`/`Extract`는 **유니온을 걸러낸다**.

## Record<K, V> — 키-값 객체 만들기

`Record<K, V>`는 키가 `K`, 값이 `V`인 객체 타입을 만든다.
"이 키들에 전부 이런 타입의 값이 들어간다"를 한 줄로 표현한다.

```ts
type Role = "admin" | "user" | "guest";

type RolePermission = Record<Role, boolean>;
// { admin: boolean; user: boolean; guest: boolean }

const 권한표: RolePermission = {
  admin: true,
  user: true,
  guest: false,
};
```

`Role`에 있는 키 하나라도 빠뜨리면 에러가 난다.
모든 키를 빠짐없이 채우게 강제하므로 매핑 테이블 정의에 딱 좋다.

값 타입이 객체여도 된다.

```ts
interface Info {
  age: number;
  job: string;
}

type CharacterMap = Record<string, Info>;

const 캐릭터: CharacterMap = {
  짱구: { age: 5, job: "유치원생" },
  둘리: { age: 100, job: "공룡" },
};
```

키를 `string`으로 두면 인덱스 시그니처(`{ [k: string]: Info }`)와 같은 효과다.

## Exclude<T, U> — 유니온에서 빼기

`Exclude<T, U>`는 유니온 `T`에서 `U`에 해당하는 멤버를 **빼낸다**.

```ts
type Status = "loading" | "success" | "error";

type Settled = Exclude<Status, "loading">;
// "success" | "error"
```

`"loading"`을 뺀 나머지만 남는다. 여러 개를 빼고 싶으면 유니온으로 넘긴다.

```ts
type OnlyError = Exclude<Status, "loading" | "success">;
// "error"
```

타입 종류로도 거를 수 있다.

```ts
type Mixed = string | number | boolean;
type NoBoolean = Exclude<Mixed, boolean>;
// string | number
```

## Extract<T, U> — 유니온에서 뽑기

`Extract`는 `Exclude`의 반대다. `T`에서 `U`에 **들어맞는 멤버만 골라낸다**.

```ts
type Mixed = string | number | boolean;

type OnlyString = Extract<Mixed, string>;
// string
```

교집합만 남긴다고 보면 된다. 둘은 정확히 거울 관계다.

```ts
type Status = "loading" | "success" | "error";

Exclude<Status, "loading">; // "success" | "error"
Extract<Status, "loading">; // "loading"
```

## 비교표

| 유틸리티 | 하는 일 | 대상 |
| --- | --- | --- |
| `Record<K, V>` | 키 `K`, 값 `V`인 객체 타입 생성 | 키 유니온 → 객체 |
| `Exclude<T, U>` | `T`에서 `U` 멤버 **제거** | 유니온 |
| `Extract<T, U>` | `T`에서 `U` 멤버만 **추출** | 유니온 |

## 헷갈렸던 포인트

**1. `Exclude`/`Extract`는 유니온 전용이다**

객체 타입에서 키를 빼는 `Omit`과 헷갈리기 쉽다.
`Omit`은 **객체 속성**을 건드리고, `Exclude`는 **유니온 멤버**를 건드린다.

```ts
// 객체 속성 제거 → Omit
type A = Omit<{ a: number; b: string }, "a">; // { b: string }

// 유니온 멤버 제거 → Exclude
type B = Exclude<"a" | "b", "a">; // "b"
```

실제로 `Omit`은 내부에서 `Exclude`를 쓴다. `Pick<T, Exclude<keyof T, K>>` 구조다.

**2. `Record`의 키는 채우지 않으면 에러**

```ts
type RolePermission = Record<"admin" | "user", boolean>;

const x: RolePermission = { admin: true };
// ❌ user가 없다
```

이 강제성이 `Record`의 장점이다. 역할이 늘면 빠뜨린 키를 컴파일러가 잡아준다.

**3. `Extract`는 조건에 맞는 게 없으면 `never`**

```ts
type T = Extract<"a" | "b", "c">; // never
```

걸리는 멤버가 하나도 없으면 빈 유니온, 즉 `never`가 된다.

## 요약

- `Record<K, V>` → 키 `K` 전부에 값 `V`를 강제하는 객체 타입
- `Exclude<T, U>` → 유니온 `T`에서 `U` **빼기**
- `Extract<T, U>` → 유니온 `T`에서 `U`만 **뽑기**
- `Exclude`/`Extract`는 유니온 전용, 객체 속성은 `Pick`/`Omit`
- `Omit`도 내부적으로 `Exclude`로 구현돼 있다

참고: https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeys-type
