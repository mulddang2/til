# 판별 유니온 (Discriminated Union)

여러 객체 타입을 유니온으로 묶었을 때, **공통 리터럴 필드(태그)로 어떤 타입인지 구분**하는 패턴이다.
태그된 유니온(tagged union)이라고도 부른다. TypeScript에서 상태나 이벤트를 다룰 때 거의 필수로 쓰인다.

## 어떤 문제를 푸나

도형 넓이를 구하는 함수를 생각해보자. 그냥 옵셔널 필드로 묶으면 타입이 애매해진다.

```ts
// ❌ 애매한 설계 — 무슨 도형인지 타입만 봐선 모른다
interface BadShape {
  kind?: string;
  radius?: number; // 원일 때만
  width?: number;  // 사각형일 때만
  height?: number; // 사각형일 때만
}
```

`radius`와 `width`가 동시에 존재할 수도 있는 타입이 돼버린다. 컴파일러가 막아주질 못한다.

## 판별 유니온으로 바꾸기

각 타입에 **공통 이름의 리터럴 필드(여기선 `kind`)**를 넣고, 유니온으로 묶는다.

```ts
interface Circle {
  kind: "circle"; // 판별자 (discriminant)
  radius: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;
```

`kind`가 리터럴 타입(`"circle"` 등)이라는 게 핵심이다. `string`이 아니라 정확한 값으로 고정돼 있다.

## switch로 좁히기

`kind`를 검사하면 그 블록 안에서 타입이 자동으로 좁혀진다(narrowing).

```ts
function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      // shape는 Circle로 좁혀짐 → radius 접근 가능
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
  }
}
```

`case "circle"` 안에서는 `shape.width`를 쓰려고 하면 컴파일 에러가 난다. 원에는 없는 필드니까.
태그 하나로 컴파일러가 알아서 갈라준다.

## never로 빠짐없이 검사 (Exhaustiveness Check)

`default`에 `never`를 두면, 나중에 새 도형을 추가했는데 처리를 빼먹었을 때 컴파일 단계에서 잡힌다.

```ts
function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      // 모든 case를 처리했다면 shape는 never가 된다
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

여기서 `Shape`에 `Square`를 새로 추가하고 `case`를 안 만들면, `shape`가 `never`로 안 좁혀져서
`_exhaustive`에 빨간 줄이 뜬다. **빠진 처리를 컴파일러가 알려주는** 셈이다.

## 비교 표

| 방식 | 잘못된 조합 방지 | 자동 타입 좁힘 | 누락 검사 |
| --- | --- | --- | --- |
| 옵셔널 필드 한 덩어리 | ❌ | ❌ | ❌ |
| 판별 유니온 | ✅ | ✅ | ✅ (never로) |

## 실전 예시 — API 상태

리덕스 액션이나 비동기 요청 상태에도 그대로 쓴다.

```ts
type FetchState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string };

function render(state: FetchState) {
  switch (state.status) {
    case "success":
      return state.data.join(", "); // data는 여기서만 접근 가능
    case "error":
      return `에러: ${state.message}`;
    default:
      return "...";
  }
}
```

`loading` 상태에서 `data`에 접근하는 실수를 컴파일러가 막아준다.

## 헷갈렸던 포인트

- **판별자는 반드시 리터럴 타입이어야 한다.** `kind: string`으로 두면 좁혀지지 않는다.
  `kind: "circle"`처럼 정확한 값이어야 컴파일러가 갈라준다.

- **판별자 이름은 자유다.** `kind`, `type`, `tag`, `status` 등 뭐든 된다. 단, 유니온의 모든 멤버가
  **같은 이름의 필드**를 공유해야 한다.

- **`never` 트릭은 런타임이 아니라 컴파일 타임 안전장치다.** 새 case를 빠뜨렸을 때
  빌드 단계에서 알려주는 게 목적이다. 깜빡 누락을 막는 데 아주 강력하다.

## 요약

- 공통 리터럴 필드(판별자)로 유니온 멤버를 구분하는 패턴
- `switch`/`if`로 판별자를 검사하면 타입이 자동으로 좁혀진다
- 잘못된 필드 조합을 타입 레벨에서 원천 차단
- `default`에 `never`를 두면 case 누락을 컴파일 타임에 잡을 수 있다
- API 상태, 리덕스 액션, 이벤트 처리 등에서 자주 쓰인다

참고: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
