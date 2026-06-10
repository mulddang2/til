# 유틸리티 타입 ③ Record / Exclude / Extract

앞에서 `Pick`/`Omit`으로 객체 타입을 가공했다.
이번엔 **키-값 매핑을 만드는 `Record`**와, **유니온에서 멤버를 골라내는 `Exclude`/`Extract`**다.
앞의 둘과 결이 좀 다르다. `Record`는 객체를 만들고, `Exclude`/`Extract`는 유니온을 다룬다.

## Record<K, V> — 키-값 타입 찍어내기

`Record<K, V>`는 **키가 `K`, 값이 `V`인 객체 타입**을 만든다.
"이런 키들에 전부 같은 모양의 값을 붙이고 싶다"일 때 쓴다.

```ts
type Role = "admin" | "guest" | "member";

type RolePermission = Record<Role, boolean>;
// { admin: boolean; guest: boolean; member: boolean }

const 권한: RolePermission = {
  admin: true,
  guest: false,
  member: true,
};
```

키를 하나라도 빠뜨리면 에러가 난다. 유니온 키를 전부 채우도록 강제하는 게 핵심이다.

```ts
const 잘못: RolePermission = { admin: true }; // ❌ guest, member 빠짐
```

값에 객체를 넣어 좀 더 실전스럽게 쓸 수도 있다.

```ts
interface Pet {
  age: number;
  owner: string;
}

type PetBook = Record<string, Pet>;

const 도감: PetBook = {
  짱구: { age: 5, owner: "신형만" },
  흰둥이: { age: 1, owner: "짱구" },
};
```

`Record<string, Pet>`는 키를 미리 못 박지 않고 "문자열 키 아무거나, 값은 `Pet`"이라는 뜻이다.
딕셔너리(dictionary) 형태의 데이터를 타입으로 표현할 때 자주 쓴다.

## Exclude<T, U> — 유니온에서 빼기

`Exclude<T, U>`는 유니온 `T`에서 **`U`에 해당하는 멤버를 빼낸다**.

```ts
type Status = "loading" | "success" | "error";

type DoneStatus = Exclude<Status, "loading">;
// "success" | "error"
```

`Omit`이 객체의 키를 뺀다면, `Exclude`는 유니온의 멤버를 뺀다. 대상이 다르다.

```ts
type T = Exclude<string | number | boolean, boolean>;
// string | number
```

## Extract<T, U> — 유니온에서 골라내기

`Extract<T, U>`는 반대다. 유니온 `T`에서 **`U`에 할당 가능한 멤버만 골라 남긴다**.

```ts
type Status = "loading" | "success" | "error";

type EndStatus = Extract<Status, "success" | "error" | "idle">;
// "success" | "error"  (idle은 Status에 없어서 무시된다)
```

`Extract`는 교집합을 남기는 느낌이다. `T`에도 있고 `U`에도 있는 것만 통과한다.

```ts
type T = Extract<string | number | boolean, number | boolean>;
// number | boolean
```

## 세 줄 비교표

| 타입 | 대상 | 하는 일 |
| --- | --- | --- |
| `Record<K, V>` | 키 유니온 | 키마다 값 타입 `V`인 객체 생성 |
| `Exclude<T, U>` | 유니온 | `T`에서 `U` 멤버 **제거** |
| `Extract<T, U>` | 유니온 | `T`에서 `U`에 맞는 멤버만 **남김** |

## 헷갈렸던 포인트

**1. `Omit` vs `Exclude` — 대상이 다르다**

이름이 비슷해서 헷갈리는데, 둘은 다루는 대상이 완전히 다르다.

```ts
// Omit: 객체 타입에서 "키"를 뺀다
type A = Omit<{ id: number; name: string }, "name">; // { id: number }

// Exclude: 유니온에서 "멤버"를 뺀다
type B = Exclude<"id" | "name", "name">; // "id"
```

사실 `Omit`은 내부적으로 `Exclude`를 써서 만들어진다. `Exclude`가 더 원시적인 도구다.

**2. `Extract`는 없는 멤버를 조용히 버린다**

`Extract`에 원본에 없는 값을 넘기면 에러 없이 그냥 무시된다.
위 예시에서 `"idle"`을 넣어도 결과에 안 나온 게 그 때문이다.

**3. `Record`의 키로 좁은 유니온을 쓰면 누락을 잡아준다**

```ts
type Lang = "ko" | "en";
type Dict = Record<Lang, string>;

const msg: Dict = { ko: "안녕" }; // ❌ en 누락 → 에러
```

`Record<string, V>`는 아무 키나 허용해서 느슨하지만,
키를 좁은 유니온으로 주면 **모든 케이스를 빠짐없이 채우도록** 강제할 수 있다. 이게 `Record`의 진짜 강점이다.

## 요약

- `Record<K, V>` → 키 `K`마다 값 `V`인 객체 타입. 좁은 유니온 키로 누락 방지
- `Exclude<T, U>` → 유니온 `T`에서 `U` 멤버 빼기
- `Extract<T, U>` → 유니온 `T`에서 `U`에 맞는 멤버만 남기기
- `Omit`은 "키", `Exclude`는 "유니온 멤버"를 다룬다 — 대상이 다르다
- `Omit`은 사실 `Exclude`로 구현돼 있다

참고: https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeys-type
