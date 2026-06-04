# 제네릭 기초 (Generics)

제네릭(generic)은 타입을 변수처럼 다루는 기능이다.
함수, 클래스, 인터페이스를 작성할 때 **구체적인 타입을 나중에 결정**할 수 있게 해준다.
코드를 재사용하면서도 타입 안전성을 유지하는 게 핵심이다.

## 왜 필요한가

제네릭 없이 여러 타입을 받는 함수를 만들면 `any`를 써야 한다.

```ts
function identity(value: any): any {
  return value;
}

const result = identity("짱구"); // result 타입: any — 타입 정보 사라짐
```

`any`를 쓰면 반환값의 타입 정보가 날아간다. 자동완성도 안 되고, 타입 에러도 못 잡는다.

## 제네릭으로 해결

`<T>`라는 **타입 파라미터**를 붙이면 타입 정보를 보존할 수 있다.

```ts
function identity<T>(value: T): T {
  return value;
}

const a = identity<string>("짱구"); // a: string
const b = identity<number>(5);      // b: number
const c = identity("둘리");         // c: string — 타입 추론으로 생략 가능
```

`T`는 관례적인 이름일 뿐이다. `Value`, `Item`, `Data` 같은 이름을 써도 된다.
실제로 함수를 호출할 때 `T`가 구체적인 타입으로 채워진다.

## 제네릭 배열

```ts
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

const first = getFirst([1, 2, 3]);       // first: number
const name = getFirst(["짱구", "둘리"]); // name: string
```

## 여러 타입 파라미터

타입 파라미터를 여러 개 쓸 수 있다.

```ts
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}

const p = pair("짱구", 5); // p: [string, number]
```

## 제네릭 인터페이스 / 타입

```ts
interface Box<T> {
  value: T;
  label: string;
}

const numBox: Box<number> = { value: 42, label: "나이" };
const strBox: Box<string> = { value: "짱구", label: "이름" };
```

```ts
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

const ok: Result<number> = { success: true, data: 100 };
const fail: Result<number> = { success: false, error: "에러 발생" };
```

API 응답 타입처럼 **성공 시 데이터 타입이 달라지는 구조**에 자주 쓴다.

## 제네릭 클래스

```ts
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }
}

const numStack = new Stack<number>();
numStack.push(1);
numStack.push(2);
const top = numStack.pop(); // top: number | undefined
```

## `any` vs 제네릭 비교표

| 비교 항목 | `any` | 제네릭 `<T>` |
|-----------|-------|-------------|
| 타입 정보 보존 | ❌ 사라짐 | ✅ 유지됨 |
| 자동완성 | ❌ 동작 안 함 | ✅ 동작 |
| 타입 에러 감지 | ❌ 못 잡음 | ✅ 잡아줌 |
| 재사용성 | ✅ 가능 | ✅ 가능 |

## 헷갈렸던 포인트

- **`<T>`는 호출 시점에 결정된다.** 작성 시점엔 그냥 "나중에 들어올 타입"이라는 자리표시자다.

- **타입 추론 덕분에 대부분 명시 안 해도 된다.** `identity<string>("짱구")` 대신 `identity("짱구")`만 써도 TypeScript가 `T = string`으로 추론한다.

- **`any`와 다르다.** `any`는 타입 검사를 포기하는 것이고, 제네릭은 타입을 유연하게 만드는 것이다. 제네릭은 `T`로 받은 값을 `T` 타입 그대로 돌려줘서 타입 안전성이 유지된다.

- **`T[]`와 `Array<T>`는 같다.** 취향 차이일 뿐이다.
  ```ts
  function len<T>(arr: T[]): number { return arr.length; }
  function len2<T>(arr: Array<T>): number { return arr.length; }
  ```

## 요약

- 제네릭은 타입을 파라미터처럼 받아 재사용성과 타입 안전성을 동시에 챙긴다.
- `function fn<T>(value: T): T` 형태가 가장 기본.
- 인터페이스, 타입 별칭, 클래스에도 쓸 수 있다.
- `any` 대신 제네릭을 쓰면 타입 정보가 보존된다.

참고: https://www.typescriptlang.org/docs/handbook/2/generics.html
