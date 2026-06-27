# infer 키워드 (infer)

조건부 타입 안에서 **타입을 변수처럼 뽑아내는** 키워드다.
`T extends ... infer U ... ? X : Y` 꼴로 쓰고, TypeScript가 패턴 매칭을 하면서
`U` 자리에 들어갈 타입을 알아서 추론(infer)해준다.
조건부 타입(conditional types)을 먼저 알고 보면 이해가 빠르다.

## 왜 필요한가

배열에서 원소 타입만 꺼내고 싶다고 해보자.
조건부 타입만으로는 "배열이면 그 안의 원소"를 표현할 방법이 없다.
이때 `infer`로 원소 자리에 이름을 붙여두고 그 이름을 결과로 돌려준다.

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;

type A = ElementType<string[]>; // string
type B = ElementType<number>;   // number
```

`(infer U)[]` 는 "원소가 `U`인 배열"이라는 패턴이다.
`string[]`이 들어오면 `U`가 `string`으로 잡히고, 배열이 아니면 그냥 자기 자신을 돌려준다.

## 함수 반환 타입 뽑기

`infer`가 가장 많이 쓰이는 곳은 함수 시그니처 분해다.
실제로 표준 유틸리티 타입 `ReturnType`이 이 방식으로 만들어져 있다.

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { name: "짱구", age: 5 };
}

type User = MyReturnType<typeof getUser>; // { name: string; age: number }
```

반환 위치를 `infer R`로 표시해두면, 함수의 리턴 타입이 통째로 `R`에 담긴다.
파라미터 쪽도 똑같이 뽑을 수 있다.

```ts
type FirstParam<T> = T extends (first: infer P, ...rest: any[]) => any ? P : never;

type P = FirstParam<(id: number, name: string) => void>; // number
```

## Promise 안의 타입 풀기

비동기 코드에서 `Promise<T>`의 `T`만 꺼내고 싶을 때도 `infer`를 쓴다.
표준 유틸리티 `Awaited`의 핵심 원리다.

```ts
type Unwrap<T> = T extends Promise<infer V> ? V : T;

type C = Unwrap<Promise<string>>; // string
type D = Unwrap<number>;          // number
```

## 여러 개 동시에 뽑기

한 패턴 안에서 `infer`를 여러 번 써도 된다.

```ts
type Swap<T> = T extends [infer A, infer B] ? [B, A] : T;

type E = Swap<[string, number]>; // [number, string]
```

튜플의 첫 번째와 두 번째 자리를 각각 `A`, `B`로 잡아 순서를 뒤집었다.

## 같은 이름으로 여러 번 추론하면?

같은 자리에 같은 이름을 두 번 쓰면 추론된 후보들이 **유니온으로 합쳐진다**.

```ts
type Merge<T> = T extends { a: infer U; b: infer U } ? U : never;

type F = Merge<{ a: string; b: number }>; // string | number
```

`a`에서 `string`, `b`에서 `number`가 잡혀 `string | number`가 된다.

## 비교표

| 패턴 | 뽑아내는 것 |
| --- | --- |
| `T extends (infer U)[] ? U : T` | 배열 원소 타입 |
| `T extends (...a: any[]) => infer R ? R : never` | 함수 반환 타입 |
| `T extends Promise<infer V> ? V : T` | Promise 결과 타입 |
| `T extends [infer A, infer B] ? ... ` | 튜플 각 자리 타입 |

## 헷갈렸던 포인트

- **`infer`는 조건부 타입 안에서만 쓸 수 있다.** `T extends ... ? ... : ...`의
  `extends` 조건부 안에서만 등장한다. 제네릭 제약(`<T extends infer U>`)이나
  일반 타입 자리에서는 못 쓴다.

- **추론이 안 되면 `false` 분기로 빠진다.** 패턴이 안 맞으면 `infer`로 뽑을 게
  없으니 `:` 뒤의 타입이 결과가 된다. 그래서 보통 안전한 기본값(`never`나 자기 자신)을 둔다.

- **반환 타입 추론은 마지막 오버로드만 잡힌다.** 함수 오버로드가 여러 개면
  `infer R`은 가장 마지막 시그니처의 반환 타입만 가져온다. 의도와 다를 수 있으니 주의한다.

## 요약

- `infer`는 조건부 타입 안에서 타입을 변수처럼 뽑아내는 키워드다.
- `T extends 패턴<infer U> ? U : 기본값` 꼴로 패턴 매칭을 한다.
- `ReturnType`, `Parameters`, `Awaited` 같은 표준 유틸리티가 전부 `infer` 기반이다.
- 조건부 타입 밖에서는 쓸 수 없다.
- 패턴이 안 맞으면 `false` 분기로 빠지니 기본값을 잘 둬야 한다.

참고: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
