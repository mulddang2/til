# 제네릭 제약 (Generic Constraints)

제네릭 타입 파라미터 `T`는 기본적으로 어떤 타입이든 받는다.
그런데 때로는 **"최소한 이런 구조를 가진 타입만 허용해"** 라고 제한을 걸고 싶을 때가 있다.
이때 `extends`와 `keyof`를 조합해 제네릭 제약(generic constraints)을 건다.

## extends로 타입 제약 걸기

`<T extends SomeType>` 형태로 쓰면 `T`는 `SomeType`을 만족하는 타입만 허용된다.

```ts
function printName<T extends { name: string }>(item: T): void {
  console.log(item.name);
}

printName({ name: "짱구", age: 5 });  // ✅ OK
printName({ name: "둘리" });           // ✅ OK
printName({ age: 5 });                 // ❌ 에러: name 속성 없음
printName("짱구");                     // ❌ 에러: string에 name 없음
```

`T`가 어떤 타입인지는 모르지만, `name: string`은 반드시 있다고 보장된다.
그래서 함수 안에서 `item.name`에 안전하게 접근할 수 있다.

## 인터페이스로 제약 표현

```ts
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(value: T): void {
  console.log(value.length);
}

logLength("짱구짱구");   // ✅ 5 — string에 length 있음
logLength([1, 2, 3]);    // ✅ 3 — array에 length 있음
logLength({ length: 10, label: "또치" }); // ✅ OK
logLength(42);           // ❌ 에러: number에 length 없음
```

## keyof로 키 제약 걸기

`keyof T`는 타입 `T`의 모든 키를 유니온으로 만든다.
`<K extends keyof T>`는 "K가 T의 키 중 하나여야 한다"는 제약이다.

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const 짱구 = { name: "짱구", age: 5, job: "없음" };

const name = getProperty(짱구, "name"); // name: string
const age  = getProperty(짱구, "age");  // age: number
getProperty(짱구, "height");            // ❌ 에러: "height"는 키가 아님
```

`T[K]`는 인덱스 접근 타입으로, `obj[key]`의 반환 타입을 정확히 추론해준다.
`key`에 맞는 타입이 자동으로 결정되는 게 핵심이다.

## 여러 제약 조합

`extends`에 인터섹션 타입(`&`)을 써서 여러 조건을 동시에 걸 수 있다.

```ts
interface Identifiable {
  id: number;
}

interface Nameable {
  name: string;
}

function greet<T extends Identifiable & Nameable>(item: T): string {
  return `[${item.id}] ${item.name}`;
}

greet({ id: 1, name: "또치", age: 7 }); // ✅ "[1] 또치"
greet({ id: 2 });                        // ❌ 에러: name 없음
```

## 기본 타입으로 제약

`extends`에는 원시 타입도 올 수 있다.

```ts
function double<T extends number>(value: T): number {
  return value * 2;
}

function toUpperCase<T extends string>(value: T): string {
  return value.toUpperCase();
}
```

## 비교: 제약 없음 vs 제약 있음

| 상황 | 제약 없음 `<T>` | 제약 있음 `<T extends ...>` |
|------|--------------|---------------------------|
| 허용 타입 | 모든 타입 | 조건을 만족하는 타입만 |
| 내부 접근 | 거의 불가 | 제약된 속성/메서드 사용 가능 |
| 타입 안전성 | 낮음 | 높음 |
| 사용 예 | `identity<T>` | `getProperty<T, K extends keyof T>` |

## 헷갈렸던 포인트

- **`extends`가 상속 의미가 아니다.** 제네릭 제약의 `extends`는 클래스 상속이 아니라 "해당 구조를 만족해야 한다"는 조건이다. `T extends { name: string }`은 "T가 name 속성을 가져야 한다"는 뜻이다.

- **`keyof T`는 타입이다, 값이 아니다.**
  ```ts
  const 짱구 = { name: "짱구", age: 5 };
  // keyof typeof 짱구 → "name" | "age"
  ```
  변수에서 `keyof`를 쓰려면 `typeof`를 먼저 적용해야 한다.

- **`T[K]`의 타입이 자동 결정된다.** `getProperty`에서 `key`가 `"name"`이면 반환값이 `string`, `"age"`면 `number`로 추론된다. `any`를 쓰지 않아도 된다.

- **제약을 걸면 해당 타입보다 "넓은" 타입은 전달 불가하다.** `<T extends string>`으로 제약을 걸면 `string | number`는 전달할 수 없다. 제약은 하한을 정하는 게 아니라 상한을 정한다.

## 요약

- `<T extends SomeType>`: T가 SomeType을 만족하는 타입만 허용
- `<K extends keyof T>`: K가 T의 키 중 하나여야 함
- `T[K]`: K 키에 해당하는 T의 값 타입 — `getProperty` 패턴에서 핵심
- 제약 없는 `T`에서는 속성에 접근 불가, 제약을 걸면 안전하게 접근 가능

참고: https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
