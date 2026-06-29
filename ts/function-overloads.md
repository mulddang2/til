# 함수 오버로드 (Function Overloads)

하나의 함수가 **인자 모양에 따라 다른 타입을 받고 다른 타입을 반환**해야 할 때가 있다.
이럴 때 같은 함수에 여러 개의 타입 시그니처를 붙이는 게 함수 오버로드(function overload)다.

## 왜 필요한가

유니온 타입만으로는 "입력 A면 반환 A, 입력 B면 반환 B" 같은 관계를 표현하기 어렵다.

```ts
// 유니온으로만 쓰면 반환 타입이 뭉뚱그려진다
function reverse(value: string | unknown[]): string | unknown[] {
  if (typeof value === "string") return value.split("").reverse().join("");
  return [...value].reverse();
}

const r = reverse("짱구"); // 타입: string | unknown[]  ← string인데 배열일 수도 있다고 나옴
```

문자열을 넣었으면 반환도 `string`이길 바라는데, 위에서는 `string | unknown[]`로 뭉개진다.
호출하는 쪽에서 다시 타입 단언을 해야 해서 불편하다.

## 오버로드 시그니처 + 구현 시그니처

오버로드는 **선언부(시그니처)** 여러 개와 **구현부** 하나로 이루어진다.

```ts
// 오버로드 시그니처 (선언만, 본문 없음)
function reverse(value: string): string;
function reverse<T>(value: T[]): T[];

// 구현 시그니처 (실제 본문, 호출하는 쪽에는 안 보임)
function reverse(value: string | unknown[]): string | unknown[] {
  if (typeof value === "string") return value.split("").reverse().join("");
  return [...value].reverse();
}

const a = reverse("둘리");        // 타입: string
const b = reverse([1, 2, 3]);     // 타입: number[]
```

이제 입력에 따라 반환 타입이 정확하게 좁혀진다.

### 중요한 규칙

- **구현 시그니처는 외부에 노출되지 않는다.** 호출하는 쪽은 오버로드 시그니처만 본다.
- 그래서 구현 시그니처는 모든 오버로드를 포괄할 수 있게 넓게 잡는다(보통 유니온).
- 오버로드 시그니처는 위에서 아래로 순서대로 매칭된다. **더 구체적인 시그니처를 위에** 둔다.

## 예시: 인자 개수가 다른 경우

```ts
function makeDate(timestamp: number): Date;
function makeDate(year: number, month: number, day: number): Date;
function makeDate(yearOrTimestamp: number, month?: number, day?: number): Date {
  if (month !== undefined && day !== undefined) {
    return new Date(yearOrTimestamp, month - 1, day);
  }
  return new Date(yearOrTimestamp);
}

const d1 = makeDate(1700000000000);   // OK
const d2 = makeDate(2026, 6, 29);     // OK
// const d3 = makeDate(2026, 6);      // ❌ 에러: 인자 2개짜리 시그니처가 없음
```

인자 2개로 호출하면 어떤 오버로드와도 안 맞아서 에러가 난다. 의도한 호출 형태만 허용된다.

## 오버로드 vs 유니온 / 제네릭

| 방식 | 입력-출력 관계 표현 | 추천 상황 |
| --- | --- | --- |
| 유니온 반환 타입 | ❌ (반환이 뭉개짐) | 입력과 출력이 무관할 때 |
| 함수 오버로드 | ✅ | 입력 형태마다 반환·인자 구조가 다를 때 |
| 제네릭 | ✅ (관계가 단순할 때) | 입력 타입을 그대로 흘려보낼 때 |

제네릭으로 깔끔하게 표현되면 제네릭을 먼저 고려한다.

```ts
// 제네릭이 더 단순한 경우 — 굳이 오버로드 안 써도 됨
function identity<T>(value: T): T {
  return value;
}
```

오버로드는 **인자 개수·조합 자체가 갈리거나**, 입력별로 반환 구조가 완전히 다를 때 빛난다.

## 헷갈렸던 포인트

- **구현 시그니처는 오버로드 목록에 안 들어간다.** 아래처럼 구현부만 넓게 써놓고 호출하면, 좁은 시그니처들만 적용되어 에러가 난다.

  ```ts
  function fn(x: string): string;
  function fn(x: number): number;
  function fn(x: string | number): string | number {
    return x;
  }
  // fn(true); // ❌ string도 number도 아니라서 에러 (구현부 union은 안 보임)
  ```

- **순서가 중요하다.** 더 구체적인 오버로드를 위에 둬야 한다. 넓은 시그니처가 위에 있으면 그게 먼저 매칭돼서 구체적인 게 무시된다.

- **오버로드보다 유니온이 나을 때도 많다.** 시그니처가 비슷한데 한두 개 타입만 다르면 오버로드 대신 유니온 파라미터가 더 읽기 쉽다. 공식 핸드북도 "가능하면 유니온을 먼저 고려하라"고 권한다.

- 객체 메서드나 `interface`에서도 오버로드를 선언할 수 있다.

  ```ts
  interface Greeter {
    greet(name: string): string;
    greet(names: string[]): string[];
  }
  ```

## 요약

- 함수 오버로드 = **여러 시그니처 + 하나의 구현부**
- 입력 형태에 따라 반환 타입·인자 구조가 달라질 때 쓴다
- 구현 시그니처는 외부에 안 보이고, 호출 측은 오버로드 시그니처만 본다
- 구체적인 시그니처를 위에, 넓은 건 아래에
- 단순히 타입만 흘려보내면 제네릭, 비슷한 시그니처면 유니온을 먼저 고려

참고: https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads
