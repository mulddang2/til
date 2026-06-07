# 유틸리티 타입 ① Partial / Required / Readonly

TypeScript는 기존 타입을 변형해서 새 타입을 만들어주는 **유틸리티 타입(utility type)**을 내장하고 있다.
직접 매핑된 타입을 작성하지 않아도, 자주 쓰는 변형은 표준으로 제공된다.
그중 가장 기본이 되는 세 가지가 `Partial`, `Required`, `Readonly`다.

먼저 기준이 될 타입 하나를 두고 시작한다.

```ts
interface User {
  name: string;
  age: number;
  email: string;
}
```

## Partial<T> — 모든 속성을 선택적으로

`Partial<T>`는 `T`의 모든 속성을 옵셔널(`?`)로 바꾼다.
즉 일부 속성만 들어 있는 객체도 허용된다.

```ts
type PartialUser = Partial<User>;
// {
//   name?: string;
//   age?: number;
//   email?: string;
// }

const 짱구: PartialUser = { name: "짱구" }; // ✅ age, email 없어도 OK
```

수정(update) 함수에서 특히 유용하다. 일부 필드만 받아서 기존 값에 덮어쓸 때 딱 맞는다.

```ts
function updateUser(user: User, patch: Partial<User>): User {
  return { ...user, ...patch };
}

const 둘리: User = { name: "둘리", age: 5, email: "dooly@test.com" };
updateUser(둘리, { age: 6 }); // ✅ age만 넘겨도 됨
```

## Required<T> — 모든 속성을 필수로

`Required<T>`는 `Partial`의 반대다. 옵셔널 속성을 전부 필수로 바꾼다.

```ts
interface Config {
  host?: string;
  port?: number;
}

type FullConfig = Required<Config>;
// {
//   host: string;
//   port: number;
// }

const 설정: FullConfig = { host: "localhost", port: 3000 };
// const 잘못된설정: FullConfig = { host: "localhost" }; // ❌ port 없음
```

옵셔널로 받은 설정값을 기본값으로 다 채운 뒤, "이제 전부 있다"고 보장할 때 쓴다.

## Readonly<T> — 모든 속성을 읽기 전용으로

`Readonly<T>`는 모든 속성에 `readonly`를 붙인다. 한 번 만든 뒤 값을 바꿀 수 없다.

```ts
type ReadonlyUser = Readonly<User>;

const 또치: ReadonlyUser = { name: "또치", age: 7, email: "ddochi@test.com" };
또치.age = 8; // ❌ 에러: 읽기 전용 속성이라 할당 불가
```

단, **얕은(shallow) 읽기 전용**이라는 점에 주의한다. 중첩 객체 내부까지는 막아주지 않는다.

```ts
interface Team {
  name: string;
  members: string[];
}

const 팀: Readonly<Team> = { name: "A팀", members: ["짱구"] };
팀.name = "B팀";          // ❌ 막힘
팀.members.push("둘리");  // ⚠️ 막히지 않음 — 배열 내부는 그대로 변경됨
```

## 셋 다 결국 매핑된 타입이다

세 유틸리티는 내부적으로 매핑된 타입(mapped type)으로 정의돼 있다. 구현을 보면 원리가 한눈에 들어온다.

```ts
type Partial<T>  = { [K in keyof T]?: T[K] };
type Required<T> = { [K in keyof T]-?: T[K] };  // -? 로 옵셔널 제거
type Readonly<T> = { readonly [K in keyof T]: T[K] };
```

`-?`가 옵셔널 표시를 떼어내는 문법이라는 것만 알면, `Required`가 왜 그렇게 동작하는지 바로 이해된다.

## 비교표

| 유틸리티 | 효과 | 주 용도 |
|----------|------|---------|
| `Partial<T>` | 모든 속성 옵셔널 | 부분 업데이트(patch) |
| `Required<T>` | 모든 속성 필수 | 기본값 채운 뒤 보장 |
| `Readonly<T>` | 모든 속성 읽기 전용 | 불변 객체 |

## 헷갈렸던 포인트

- **`Readonly`는 얕다.** 중첩된 객체나 배열 내부의 변경은 막지 못한다. 깊은 불변이 필요하면 `DeepReadonly` 같은 걸 따로 만들거나 라이브러리를 써야 한다.

- **`Partial`을 남발하면 위험하다.** 전부 옵셔널로 풀어버리면 정작 필요한 필드가 빠져도 컴파일러가 못 잡는다. 업데이트용 같은 명확한 자리에만 쓰는 게 좋다.

- **`readonly`는 컴파일 타임 검사일 뿐이다.** 런타임에서 실제로 객체를 얼리는 게 아니다. 진짜 변경을 막으려면 `Object.freeze`를 따로 써야 한다.

- **`Required`는 `?`만 떼지, `undefined` 유니온은 못 뗀다.** `name: string | undefined`처럼 명시적으로 `undefined`가 들어간 타입은 `Required`로도 그대로 남는다. 옵셔널(`?`)과 `undefined` 유니온은 다른 개념이다.

## 요약

- `Partial<T>`: 전부 옵셔널 → 부분 업데이트에 사용
- `Required<T>`: 전부 필수 → `-?` 매핑으로 옵셔널 제거
- `Readonly<T>`: 전부 읽기 전용 → 단, 얕은 불변
- 셋 다 매핑된 타입으로 만들어진 표준 유틸리티

참고: https://www.typescriptlang.org/docs/handbook/utility-types.html
