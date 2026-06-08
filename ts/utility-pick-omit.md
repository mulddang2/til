# 유틸리티 타입 ② Pick / Omit

`Partial`, `Required`, `Readonly`가 **속성의 성질**(옵셔널·필수·읽기 전용)을 바꾸는 유틸리티라면,
`Pick`과 `Omit`은 **속성 자체를 골라내거나 덜어내는** 유틸리티다.
원래 타입에서 필요한 부분만 잘라 새 타입을 만들 때 쓴다.

기준이 될 타입 하나를 두고 시작한다.

```ts
interface User {
  id: number;
  name: string;
  age: number;
  email: string;
  password: string;
}
```

## Pick<T, K> — 필요한 속성만 골라낸다

`Pick<T, K>`는 `T`에서 `K`로 지정한 속성만 뽑아 새 타입을 만든다.
`K`는 `T`의 키 중 하나(또는 유니온)여야 한다.

```ts
type UserPreview = Pick<User, "id" | "name">;
// {
//   id: number;
//   name: string;
// }

const 짱구: UserPreview = { id: 1, name: "짱구" }; // ✅ id, name만
```

목록 화면처럼 일부 필드만 필요할 때 딱 맞는다.
없는 키를 넘기면 바로 에러가 나서 오타도 잡아준다.

```ts
type Wrong = Pick<User, "nickname">; // ❌ 'nickname'은 User 키가 아님
```

## Omit<T, K> — 특정 속성만 덜어낸다

`Omit<T, K>`는 `Pick`의 반대다. `K`로 지정한 속성을 **빼고** 나머지를 전부 가져온다.

```ts
type SafeUser = Omit<User, "password">;
// {
//   id: number;
//   name: string;
//   age: number;
//   email: string;
// }

const 둘리: SafeUser = {
  id: 2,
  name: "둘리",
  age: 5,
  email: "dooly@test.com",
}; // ✅ password 없음
```

응답 객체에서 민감한 필드(`password` 등)만 빼고 내려줄 때 자주 쓴다.
여러 개를 빼고 싶으면 유니온으로 넘긴다.

```ts
type PublicUser = Omit<User, "password" | "email">;
```

## Pick과 Omit은 동전의 양면이다

같은 결과를 `Pick`으로도, `Omit`으로도 만들 수 있다. 어느 쪽이 더 짧은지로 고르면 된다.

```ts
// 둘 다 { id, name } 만 남긴다
type A = Pick<User, "id" | "name">;
type B = Omit<User, "age" | "email" | "password">;
```

남길 게 적으면 `Pick`, 뺄 게 적으면 `Omit`이 읽기 편하다.

## 구현 원리

`Pick`은 매핑된 타입(mapped type)으로, `Omit`은 `Pick`과 `Exclude`를 조합해 정의돼 있다.

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

`Omit`은 "`T`의 키 전체에서 `K`를 뺀 키"로 다시 `Pick`하는 구조다.
그래서 `Omit`의 동작이 헷갈리면 `Pick + Exclude`로 풀어 생각하면 된다.

## 비교표

| 유틸리티 | 효과 | 주 용도 |
|----------|------|---------|
| `Pick<T, K>` | `K` 속성만 남김 | 목록·요약 등 일부 필드만 |
| `Omit<T, K>` | `K` 속성만 제거 | 민감 필드 제외한 응답 |

## 헷갈렸던 포인트

- **`Pick`의 `K`는 키를 검사하지만, `Omit`의 `K`는 그렇지 않다.** `Pick<T, K>`는 `K extends keyof T`라 없는 키를 넘기면 에러가 난다. 반면 `Omit<T, K>`는 `K extends keyof any`라서 **존재하지 않는 키를 빼도 에러가 안 난다.** 오타가 조용히 통과할 수 있으니 주의한다.

  ```ts
  type X = Omit<User, "pasword">; // ⚠️ 오타지만 에러 안 남 — 아무것도 안 빠짐
  ```

- **유니온 타입에 `Omit`을 쓰면 의도와 다르게 동작한다.** `Omit`은 키를 분배(distribute)하지 않아서, 판별 유니온에 그대로 쓰면 공통 속성만 남는 경우가 있다. 유니온엔 직접 만든 `DistributiveOmit`을 쓰기도 한다.

- **`Pick`은 옵셔널·`readonly` 표시를 그대로 가져온다.** 원래 `name?: string`이었다면 `Pick` 후에도 옵셔널이 유지된다. 성질을 바꾸는 게 아니라 골라내기만 한다.

## 요약

- `Pick<T, K>`: `K`로 지정한 속성만 남김 → 일부 필드 타입
- `Omit<T, K>`: `K`로 지정한 속성만 제거 → 민감 필드 제외
- 둘은 서로 반대, 같은 결과를 양쪽으로 표현 가능
- `Omit = Pick + Exclude` 구조
- `Omit`은 없는 키를 빼도 에러를 안 내니 오타에 주의

참고: https://www.typescriptlang.org/docs/handbook/utility-types.html
