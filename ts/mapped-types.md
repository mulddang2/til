# 매핑된 타입 (Mapped Types)

이미 있는 타입의 키를 하나씩 돌면서 새 타입을 만드는 문법이 매핑된 타입(mapped type)이다.
배열의 `map`이 원소를 하나씩 변환하듯, 매핑된 타입은 **속성을 하나씩 변환한다**.
`Partial`, `Readonly` 같은 유틸리티 타입도 전부 이 문법으로 만들어져 있다.

## 기본 문법: `[K in keyof T]`

`keyof`로 키 유니온을 뽑고, `in`으로 그 키들을 하나씩 순회한다.

```ts
interface User {
  name: string;
  age: number;
}

type Clone<T> = {
  [K in keyof T]: T[K];
};

type UserClone = Clone<User>;
// { name: string; age: number } — User와 똑같다
```

`[K in keyof T]`에서 `K`는 키 이름을 하나씩 받고, `T[K]`(인덱스 접근)로 그 키의 값 타입을 꺼낸다.
위 예시는 그대로 복사만 하지만, 여기에 손을 대면 변형이 시작된다.

## 수식어 추가: `readonly`, `?`

각 속성에 `readonly`나 옵셔널(`?`)을 붙일 수 있다.

```ts
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

type ReadonlyUser = MyReadonly<User>;
// { readonly name: string; readonly age: number }

type PartialUser = MyPartial<User>;
// { name?: string; age?: number }
```

표준 라이브러리의 `Readonly`, `Partial`이 정확히 이렇게 생겼다. 직접 구현해보면 유틸리티 타입이 마법이 아니란 게 보인다.

## 수식어 제거: `-readonly`, `-?`

`-`를 앞에 붙이면 수식어를 **떼어낸다**. 옵셔널을 강제로 필수로 만드는 `Required`가 대표 예시다.

```ts
type MyRequired<T> = {
  [K in keyof T]-?: T[K];
};

interface Config {
  host?: string;
  port?: number;
}

type StrictConfig = MyRequired<Config>;
// { host: string; port: number } — ? 가 사라짐

type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

## 값 타입 바꾸기

값 타입 자리에 `T[K]` 대신 다른 타입을 넣으면 모든 속성의 타입을 한 번에 바꾼다.

```ts
type Booleanize<T> = {
  [K in keyof T]: boolean;
};

type UserFlags = Booleanize<User>;
// { name: boolean; age: boolean }
```

## 키 재매핑: `as`

`as`를 쓰면 키 이름 자체를 바꾼다(키 리매핑, key remapping). 템플릿 리터럴 타입과 같이 쓰면 강력하다.

```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }
```

`as never`로 매핑하면 그 키를 결과에서 **빼버릴** 수도 있다.

```ts
type RemoveAge<T> = {
  [K in keyof T as K extends "age" ? never : K]: T[K];
};

type NoAge = RemoveAge<User>; // { name: string }
```

## 비교표

| 문법 | 효과 | 표준 유틸리티 |
| --- | --- | --- |
| `[K in keyof T]: T[K]` | 그대로 복사 | - |
| `readonly [K in keyof T]` | 읽기 전용 추가 | `Readonly` |
| `[K in keyof T]?` | 옵셔널 추가 | `Partial` |
| `[K in keyof T]-?` | 옵셔널 제거 | `Required` |
| `-readonly [K in keyof T]` | 읽기 전용 제거 | - |
| `[K in keyof T as ...]` | 키 이름 변경 | - |

## 헷갈렸던 포인트

- **`in`은 `for...in`이 아니다.** 매핑된 타입의 `in`은 타입 레벨에서 유니온을 순회하는
  전용 문법이다. 런타임 `for...in`과는 이름만 같고 동작은 다르다.

- **`as`로 키를 바꿀 때 원래 키는 `string | number | symbol`일 수 있다.** 그래서
  `Capitalize<K>`처럼 문자열 함수에 바로 못 넣고, `Capitalize<string & K>`로
  `string`만 추려서 넣어야 한다.

- **매핑된 타입엔 다른 속성을 같이 못 적는다.** `{ [K in keyof T]: ... ; extra: string }`
  같은 건 안 된다. 하나의 매핑 블록만 쓸 수 있다. 추가 속성이 필요하면 인터섹션(`&`)으로 합친다.

- **`-?`와 `?`를 헷갈리지 말 것.** `?`는 옵셔널을 붙이고, `-?`는 떼어낸다. 부호가 의미를 뒤집는다.

## 요약

- 매핑된 타입은 `[K in keyof T]`로 키를 순회하며 새 타입을 찍어낸다.
- `readonly` / `?`를 붙이거나 `-readonly` / `-?`로 떼어낼 수 있다.
- 값 타입을 `T[K]` 대신 다른 타입으로 바꿔 전체 속성 타입을 갈아끼운다.
- `as`로 키 이름을 리매핑하고, `as never`로 키를 제거한다.
- `Partial` / `Readonly` / `Required` 전부 이 문법으로 만들어진 매핑된 타입이다.

참고: https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
