# 판별 유니온 (Discriminated Union)

여러 형태의 객체를 하나의 유니온(union)으로 묶을 때, 각 타입에 **공통된 리터럴 필드**를 두면
그 필드 값만 보고 타입을 좁힐(narrowing) 수 있다. 이걸 판별 유니온(discriminated union)이라고 한다.
태그 유니온(tagged union)이라고도 부른다.

## 어떤 문제를 푸나

도형 넓이를 구하는 코드를 생각해보자. 원이면 반지름, 사각형이면 가로·세로가 필요하다.
판별 필드 없이 그냥 합치면 어떤 도형인지 구분할 방법이 애매하다.

```ts
// ❌ 판별 필드가 없는 유니온
type Circle = { radius: number };
type Square = { side: number };
type Shape = Circle | Square;

function area(shape: Shape) {
  // shape.radius? shape.side? 뭘 봐야 할지 타입만으로는 모른다
  return "radius" in shape ? Math.PI * shape.radius ** 2 : shape.side ** 2;
}
```

`in`으로 어찌어찌 되긴 하지만, 종류가 늘어나면 금방 지저분해진다.

## 판별 필드(discriminant)를 추가한다

각 타입에 `kind` 같은 **리터럴 타입(literal type)** 필드를 하나씩 박는다.
이 공통 필드가 판별자(discriminant) 역할을 한다.

```ts
interface Circle {
  kind: "circle"; // 리터럴 타입
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

type Shape = Circle | Square | Rectangle;
```

이제 `switch`로 `kind`만 보면 TypeScript가 알아서 타입을 좁혀준다.

```ts
function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // shape는 Circle
    case "square":
      return shape.side ** 2; // shape는 Square
    case "rectangle":
      return shape.width * shape.height; // shape는 Rectangle
  }
}
```

`case "circle"` 블록 안에서는 `shape`가 `Circle`로 좁혀져서 `radius`에 안전하게 접근된다.
없는 필드를 건드리면 컴파일 에러가 난다.

## never로 빠뜨린 케이스 잡기

판별 유니온의 진짜 강점은 **새 타입이 추가됐을 때 처리 안 한 곳을 컴파일러가 잡아주는 것**이다.
`default`에서 `never`에 할당해보면 된다. 이걸 망라 검사(exhaustiveness check)라고 한다.

```ts
function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // 모든 케이스를 처리했다면 shape는 never가 된다
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

만약 나중에 `Triangle`을 `Shape`에 추가하고 `case`를 안 만들면,
`default`에서 `shape`가 `never`가 아니라 `Triangle`이라 **할당 에러**가 난다.
빠뜨린 분기를 컴파일 단계에서 바로 알 수 있다.

## API 상태 모델링에도 좋다

로딩/성공/에러 같은 상태를 다룰 때 특히 잘 맞는다.

```ts
type FetchState =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string };

function render(state: FetchState) {
  switch (state.status) {
    case "loading":
      return "불러오는 중...";
    case "success":
      return state.data.join(", "); // data는 여기서만 접근 가능
    case "error":
      return state.message;
  }
}
```

`data`는 `success`일 때만, `message`는 `error`일 때만 존재한다.
잘못된 상태에서 엉뚱한 필드를 읽는 실수를 타입이 막아준다.

## 헷갈렸던 포인트

- **판별 필드는 리터럴 타입이어야 한다.** `kind: string`처럼 넓은 타입이면 좁혀지지 않는다.
  `kind: "circle"`처럼 구체적인 문자열 리터럴(또는 숫자, boolean 리터럴)을 써야 판별이 된다.

- **모든 멤버가 같은 이름의 필드를 가져야 한다.** 판별자 이름은 `kind`든 `type`이든 `status`든
  자유지만, 유니온의 모든 멤버에 동일한 이름으로 존재해야 한다.

- **`type` 예약어와 헷갈리지 말 것.** 판별 필드 이름으로 `type`을 자주 쓰는데,
  이건 그냥 객체 속성 이름이라 TypeScript의 `type` 키워드와는 무관하다.

- **`if`로도 되지만 `switch`가 깔끔하다.** 케이스가 둘이면 `if (shape.kind === "circle")`도
  충분하다. 셋 이상이면 `switch` + `never` 조합이 망라 검사까지 챙겨줘서 유리하다.

## 요약

- 유니온의 각 타입에 공통 리터럴 필드(판별자)를 두는 패턴이 판별 유니온
- 판별 필드 값으로 `switch`하면 타입이 자동으로 좁혀진다
- 판별 필드는 반드시 리터럴 타입 (`string`처럼 넓으면 안 됨)
- `default`에서 `never`에 할당하면 빠뜨린 케이스를 컴파일러가 잡아준다 (망라 검사)
- API 로딩/성공/에러 상태 모델링에 특히 잘 어울린다

참고: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
