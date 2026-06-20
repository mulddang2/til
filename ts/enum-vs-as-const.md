# enum vs as const 객체 (Enum vs as const)

TypeScript에서 "정해진 값 몇 개 중 하나"를 표현할 때 `enum`을 쓸지, `as const` 객체를 쓸지 헷갈린다.
결론부터 말하면 요즘은 **`as const` 객체를 더 선호하는 분위기**다. 이유를 하나씩 보자.

## enum 기본

`enum`은 이름 있는 상수 집합을 만든다.

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

const move = Direction.Up; // 0
```

값을 안 주면 `0`부터 자동으로 숫자가 매겨진다. 이걸 **숫자 enum(numeric enum)**이라 한다.
문자열을 직접 지정하면 **문자열 enum(string enum)**이 된다.

```ts
enum Status {
  Loading = "loading",
  Success = "success",
  Error = "error",
}

const s = Status.Success; // "success"
```

## enum의 문제점

### 1. 컴파일하면 코드가 남는다

`enum`은 타입이 아니라 **실제 객체로 컴파일된다**. `interface`나 `type`은 컴파일 후 사라지는데, `enum`은 JS 코드를 남긴다.

```js
// enum Direction 컴파일 결과 (대략)
var Direction;
(function (Direction) {
  Direction[(Direction["Up"] = 0)] = "Up";
  Direction[(Direction["Down"] = 1)] = "Down";
})(Direction || (Direction = {}));
```

이렇게 양방향 매핑(reverse mapping)까지 만들어져서 번들 크기가 커진다.

### 2. 숫자 enum은 타입이 헐겁다

숫자 enum은 enum에 없는 숫자도 그냥 들어간다.

```ts
enum Direction {
  Up,
  Down,
}

const d: Direction = 99; // ❌ 에러 안 남! 통과해버린다
```

이건 명백히 버그인데 컴파일러가 안 잡아준다.

## as const 객체

같은 걸 `as const` 객체로 표현하면 이런 문제가 없다.

```ts
const Direction = {
  Up: "UP",
  Down: "DOWN",
  Left: "LEFT",
  Right: "RIGHT",
} as const;

type Direction = (typeof Direction)[keyof typeof Direction];
// "UP" | "DOWN" | "LEFT" | "RIGHT"

const move: Direction = "UP"; // ✅
const bad: Direction = "OOPS"; // ❌ 제대로 에러 남
```

`as const`를 붙이면 객체의 값들이 리터럴 타입(literal type)으로 좁혀진다.
`typeof` + `keyof`로 값들을 유니온 타입으로 뽑아낸다. 객체와 타입에 같은 이름을 써도 된다.

## 비교표

| 구분 | enum | as const 객체 |
| --- | --- | --- |
| 컴파일 후 JS 코드 | 남는다 | 일반 객체로 남음(가벼움) |
| 타입 안전성 | 숫자 enum은 헐겁다 | 안전하다 |
| 트리 셰이킹(tree-shaking) | 어렵다 | 잘 된다 |
| 값 순회 | 까다로움 | `Object.values()`로 간단 |
| 별도 문법 학습 | 필요 | 일반 JS 객체 그대로 |

## 값 순회도 더 자연스럽다

```ts
const Status = {
  Loading: "loading",
  Success: "success",
  Error: "error",
} as const;

// 모든 값 배열로
Object.values(Status); // ["loading", "success", "error"]
```

`enum`은 숫자 enum이면 키와 값이 섞여 나와서 순회할 때 한 번 더 거른다.

## 헷갈렸던 포인트

- **`const enum`은 어떤가?** `const enum`은 인라인되어서 코드가 거의 안 남는다. 하지만 `isolatedModules`(바벨, esbuild 등) 환경에서 동작이 깨질 수 있어서 추천하지 않는다.

- **enum을 무조건 쓰지 말라는 건 아니다.** 팀 컨벤션이 enum이거나, 백엔드와 enum 값을 공유하는 등 익숙한 패턴이면 string enum 정도는 써도 괜찮다. 다만 **숫자 enum은 피하는 게 안전하다.**

- **타입과 값 이름이 같아도 된다.** `const Direction`과 `type Direction`은 네임스페이스가 달라서 충돌하지 않는다. enum처럼 `Direction.Up`도 쓰고 `Direction` 타입도 쓸 수 있다.

## 요약

- `enum`은 컴파일 후 JS 코드가 남고, 숫자 enum은 타입이 헐겁다
- `as const` 객체 + `typeof`/`keyof` 조합이 더 가볍고 안전하다
- 값 순회는 `Object.values()`로 간단하게 된다
- 꼭 enum을 써야 한다면 **string enum만**, 숫자 enum은 피한다

참고: https://www.typescriptlang.org/docs/handbook/enums.html
