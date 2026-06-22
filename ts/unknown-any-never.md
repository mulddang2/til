# unknown vs any vs never

TypeScript에는 비슷해 보이지만 성격이 완전히 다른 세 타입이 있다.
`any`, `unknown`, `never`. 이름만 보면 헷갈리는데 역할은 정반대에 가깝다.

한 줄로 요약하면 이렇다.

- `any` — **타입 체크를 끈다**. 아무거나 다 되고 아무 데나 다 들어간다.
- `unknown` — **모든 값을 담지만, 쓰기 전에 좁혀야 한다**. 안전한 `any`.
- `never` — **값이 절대 존재할 수 없다**. 끝까지 도달하지 않는 자리.

## any — 타입 검사 포기

`any`는 컴파일러한테 "이건 그냥 넘어가"라고 말하는 타입이다.
무슨 프로퍼티에 접근하든, 어떤 함수로 호출하든 에러를 안 낸다.

```ts
let value: any = "짱구";

value.foo.bar.baz;     // ❌ 런타임에 터지지만 컴파일은 통과
value();               // 호출해도 에러 없음
const n: number = value; // 아무 타입에나 그냥 들어감
```

편하지만 위험하다. `any`를 쓰는 순간 그 변수 주변의 타입 안전성은 사라진다.
런타임 에러를 컴파일 타임에 잡겠다는 TypeScript의 존재 이유를 스스로 꺼버리는 셈이다.

## unknown — 안전한 any

`unknown`도 모든 값을 받을 수 있다. 여기까진 `any`와 같다.
차이는 **꺼내 쓸 때** 생긴다. `unknown` 값은 타입을 좁히기 전엔 아무것도 못 한다.

```ts
let value: unknown = "짱구";

value.length;          // ❌ 에러: unknown에는 접근 불가
const s: string = value; // ❌ 에러: unknown을 string에 못 넣음

// 타입을 좁히면 그때부터 쓸 수 있다
if (typeof value === "string") {
  value.length;        // ✅ OK: 여기선 string으로 좁혀짐
}
```

그래서 외부에서 들어오는 정체불명의 값(예: `JSON.parse` 결과, API 응답)을
받을 땐 `any`보다 `unknown`이 안전하다. 강제로 타입 가드(type guard)를 거치게 만든다.

```ts
function parseConfig(raw: string): unknown {
  return JSON.parse(raw); // 무슨 모양일지 모르니 unknown
}

const config = parseConfig('{"port": 8080}');

if (typeof config === "object" && config !== null && "port" in config) {
  // 여기서 안전하게 다룬다
}
```

## never — 존재할 수 없는 값

`never`는 **절대 발생하지 않는 값**의 타입이다.
값을 만들 수 없고, 어떤 타입에도 (자기 자신 빼곤) 할당되지 않는다.

대표적으로 두 군데서 나온다.

```ts
// 1. 항상 예외를 던지는 함수 — 정상적으로 반환하지 않음
function throwError(message: string): never {
  throw new Error(message);
}

// 2. 끝나지 않는 무한 루프
function loop(): never {
  while (true) {}
}
```

또 유니온에서 어떤 경우도 남지 않으면 자동으로 `never`가 된다.

```ts
type A = string & number; // never — string이면서 number인 값은 없음
```

### never로 빠짐없이 처리했는지 검사 (Exhaustiveness Check)

`never`의 진짜 쓸모는 판별 유니온을 빠짐없이 처리했는지 확인할 때다.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      // 모든 케이스를 처리했다면 shape는 never가 된다
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

나중에 `Shape`에 `{ kind: "triangle" }`을 추가하면 `default`에서
`shape`가 더 이상 `never`가 아니게 되어 **컴파일 에러**가 난다.
처리 안 한 케이스를 컴파일러가 대신 잡아주는 것이다.

## 비교 표

| | any | unknown | never |
| --- | --- | --- | --- |
| 어떤 값이든 담기 | ✅ | ✅ | ❌ (값 없음) |
| 좁히지 않고 바로 사용 | ✅ | ❌ | — |
| 다른 타입에 할당 | ✅ (다 됨) | ❌ | ✅ (어디든 됨) |
| 이 타입에 할당받기 | ✅ (다 됨) | ✅ (다 됨) | ❌ (never만) |
| 타입 안전성 | 없음 | 있음 | 있음 |

## 헷갈렸던 포인트

- **`unknown`은 모든 타입의 상위(top), `never`는 모든 타입의 하위(bottom)다.**
  `unknown`은 모든 값을 받지만 어디에도 못 나가고, `never`는 아무 값도 못 받지만 어디에나 들어간다. 정반대다.

- **`any`는 `unknown`과 달리 양방향으로 다 통과한다.** `any`는 받기도 나가기도 다 되니까 사실상 타입 시스템에 구멍을 뚫는다. `unknown`은 들어오는 건 다 받아도 나가는 건 막는다.

- **`never`가 의도치 않게 튀어나올 때가 있다.** 빈 배열 `[]`의 추론, 절대 겹치지 않는 인터섹션(`string & number`) 등에서 `never`가 나오면 "여긴 값이 있을 수 없다"는 신호다.

- **`never`는 유니온에서 흡수된다.** `string | never`는 그냥 `string`이다. 반대로 `string & unknown`은 `string`, `string | unknown`은 `unknown`이 된다.

## 요약

- `any`는 타입 검사를 끄는 탈출구 — 가급적 피한다.
- 정체불명의 값은 `any` 대신 `unknown`으로 받고, 타입 가드로 좁혀서 쓴다.
- `never`는 "절대 도달 안 함"을 표현 — 예외 함수, exhaustiveness check에 쓴다.
- `unknown`은 top 타입, `never`는 bottom 타입. 둘은 정반대 위치에 있다.

참고: https://www.typescriptlang.org/docs/handbook/2/functions.html#never
