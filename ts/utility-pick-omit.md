# 유틸리티 타입 ② Pick / Omit

객체 타입에서 **일부 속성만 골라 쓰거나, 특정 속성만 빼고 쓰고 싶을 때** 있다.
이럴 때 매번 새 타입을 다시 정의하지 않고 기존 타입에서 가공해 쓰는 게 `Pick`과 `Omit`이다.

## 기준 타입

아래 `User` 타입을 예시로 쭉 쓴다.

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}
```

## Pick<T, K> — 골라 뽑기

`Pick`은 타입 `T`에서 **원하는 키 `K`만 골라** 새 타입을 만든다.

```ts
type UserProfile = Pick<User, "id" | "name">;
// { id: number; name: string }

const 짱구: UserProfile = { id: 1, name: "짱구" };
```

여러 키를 고를 땐 유니온(`|`)으로 넘긴다.
`email`이나 `password`는 빠졌으니 넣으면 에러가 난다.

```ts
const 둘리: UserProfile = { id: 2, name: "둘리", email: "x@x.com" };
// ❌ email은 UserProfile에 없다
```

## Omit<T, K> — 빼고 나머지

`Omit`은 반대다. 타입 `T`에서 **키 `K`만 빼고** 나머지를 전부 가져온다.

```ts
type PublicUser = Omit<User, "password">;
// { id: number; name: string; email: string }

const 또치: PublicUser = {
  id: 3,
  name: "또치",
  email: "ddochi@example.com",
};
```

비밀번호처럼 외부로 내보내면 안 되는 필드를 떼어낼 때 자주 쓴다.
빼고 싶은 키가 여러 개면 역시 유니온으로 넘긴다.

```ts
type IdOnly = Omit<User, "name" | "email" | "password">;
// { id: number }
```

## Pick vs Omit, 언제 뭘 쓰나

둘은 사실상 같은 일을 정반대 방향에서 한다. **남길 게 적으면 `Pick`, 뺄 게 적으면 `Omit`**.

| 상황 | 추천 | 이유 |
| --- | --- | --- |
| 4개 중 2개만 필요 | `Pick` | 고를 게 더 적다 |
| 4개 중 1개만 빼고 싶다 | `Omit` | 뺄 게 더 적다 |
| 속성이 추가돼도 자동 반영 | `Omit` | 새 속성이 자동 포함된다 |

마지막 줄이 중요하다. `User`에 나중에 `phone` 속성이 생기면,
`Pick`으로 만든 타입은 그대로지만 `Omit`으로 만든 타입엔 `phone`이 자동으로 들어온다.

## 실전 예시 — 폼 입력 타입

회원가입 폼은 보통 서버가 만들어주는 `id`만 빼고 받는다.

```ts
type SignupForm = Omit<User, "id">;
// { name: string; email: string; password: string }

function signup(form: SignupForm) {
  // id는 서버에서 생성하므로 폼에는 없다
}
```

## 헷갈렸던 포인트

**1. `Omit`은 없는 키를 넣어도 에러를 안 낸다**

```ts
type T = Omit<User, "nickname">; // ❌ 에러 안 남, 그냥 User 전체
```

`Pick`은 없는 키를 고르면 바로 에러가 나지만, `Omit`은 오타가 나도 조용히 통과한다.
오타가 있으면 아무것도 안 빼지므로 주의해야 한다.

**2. `Pick`/`Omit`은 `Exclude`로 구현돼 있다**

`Omit`의 내부 정의는 대략 이렇다. `keyof`로 키를 다 뽑고 `Exclude`로 빼는 구조다.

```ts
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

즉 `Omit`은 `Pick`과 `Exclude`의 조합이다.

**3. 속성 추가/제거 후 깨지는 지점이 다르다**

`Pick`으로 만든 타입은 원본에서 키가 사라지면 컴파일 에러로 바로 잡힌다.
타입 안정성을 더 빡빡하게 가져가고 싶으면 `Pick`이 유리한 경우도 있다.

## 요약

- `Pick<T, K>` → `T`에서 키 `K`만 **골라** 새 타입
- `Omit<T, K>` → `T`에서 키 `K`만 **빼고** 나머지
- 남길 게 적으면 `Pick`, 뺄 게 적으면 `Omit`
- `Omit`은 원본에 속성이 추가되면 자동 반영, 오타엔 관대하니 주의
- `password` 같은 민감 필드 제거, 폼 타입 정의에 자주 쓴다

참고: https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys
