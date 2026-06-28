# 템플릿 리터럴 타입 (Template Literal Types)

문자열 리터럴 타입을 백틱(`` ` ``)으로 조합해서 **새로운 문자열 타입을 만들어내는** 기능이다.
자바스크립트의 템플릿 리터럴(``` `${}` ```)과 문법이 똑같지만, 값이 아니라 **타입**을 다룬다.
TypeScript 4.1에서 들어왔다.

## 기본 문법

리터럴 타입을 `${}` 안에 끼워 넣으면 문자열이 합쳐진 타입이 나온다.

```ts
type Greeting = `안녕, ${string}`;

const a: Greeting = "안녕, 짱구"; // ✅
const b: Greeting = "잘가, 둘리"; // ❌ "안녕,"으로 시작 안 함
```

`${string}` 자리에는 아무 문자열이나 들어갈 수 있지만, 앞의 `"안녕, "` 패턴은 반드시 지켜야 한다.

## 유니온과 조합하면 폭발한다

`${}` 안에 유니온 타입을 넣으면, 가능한 모든 조합이 자동으로 만들어진다.

```ts
type Color = "red" | "blue";
type Size = "sm" | "lg";

type ClassName = `${Color}-${Size}`;
// "red-sm" | "red-lg" | "blue-sm" | "blue-lg"
```

유니온 2개를 곱하면 2 × 2 = 4가지 조합이 나온다.
디자인 시스템의 클래스명이나 토큰 이름을 타입으로 강제할 때 잘 맞는다.

## 이벤트 핸들러 이름 만들기

가장 흔한 활용은 키 이름을 가공하는 것이다.
아래는 `on` 접두사를 붙인 핸들러 이름을 만드는 예시다.

```ts
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickHandler = EventName<"click">; // "onClick"
type FocusHandler = EventName<"focus">; // "onFocus"
```

## 내장 문자열 유틸리티 타입

템플릿 리터럴 타입과 함께 쓰라고 만들어진 4개의 내장 타입이 있다.

```ts
type A = Uppercase<"hello">;   // "HELLO"
type B = Lowercase<"WORLD">;   // "world"
type C = Capitalize<"toss">;   // "Toss"
type D = Uncapitalize<"Toss">; // "toss"
```

`Capitalize`는 첫 글자만 대문자로 바꾼다. 위 핸들러 예시에서 `on` 뒤에 붙일 때 유용하다.

## 객체 키 변환에 활용

매핑된 타입(mapped types)의 키 재매핑(`as`)과 같이 쓰면, 객체의 키 이름을 통째로 바꿀 수 있다.

```ts
type Person = {
  name: string;
  age: number;
};

type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }
```

`name` → `getName`, `age` → `getAge`로 키가 바뀐다.
`string & K`는 키가 `string`임을 좁혀주는 관용구다. (객체 키는 `string | number | symbol`이라 그냥은 `Capitalize`에 못 넣는다.)

## 비교표

| 도구 | 하는 일 |
| --- | --- |
| `` `a-${T}` `` | 문자열 패턴 결합 |
| 유니온 + 템플릿 | 가능한 조합 전부 생성 |
| `Uppercase` / `Lowercase` | 전체 대/소문자 |
| `Capitalize` / `Uncapitalize` | 첫 글자만 대/소문자 |
| `as` 키 재매핑 + 템플릿 | 객체 키 이름 변환 |

## 헷갈렸던 포인트

- **유니온 조합은 금방 커진다.** 유니온 3개를 곱하면 경우의 수가 곱셈으로 불어난다.
  너무 큰 조합(보통 10만 개 이상)은 컴파일러가 거부한다. 패턴을 좁혀서 쓰는 게 좋다.

- **`${number}` 는 숫자 검증이 아니다.** `` `id-${number}` `` 타입은 `"id-123"`을 받아주지만,
  런타임에 실제 숫자인지 검사하진 않는다. 어디까지나 **타입 수준의 패턴**일 뿐이다.

- **키 재매핑에서 `string & K` 를 빼먹기 쉽다.** `keyof T`는 `symbol`도 포함할 수 있어서
  `Capitalize<K>`가 바로 안 된다. `string &`로 한 번 좁혀줘야 컴파일된다.

## 요약

- 템플릿 리터럴 타입은 문자열 리터럴을 백틱으로 조합해 새 문자열 타입을 만든다.
- 유니온을 끼우면 가능한 모든 조합이 자동 생성된다.
- `Uppercase` / `Lowercase` / `Capitalize` / `Uncapitalize` 내장 타입과 함께 쓴다.
- 매핑된 타입의 키 재매핑(`as`)과 합치면 객체 키 이름을 변환할 수 있다.
- 어디까지나 타입 수준의 패턴이라 런타임 검증은 해주지 않는다.

참고: https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
