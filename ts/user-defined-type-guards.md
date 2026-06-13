# 사용자 정의 타입 가드 (User-Defined Type Guards)

`typeof`, `instanceof`, `in` 같은 기본 타입 가드(type guard)만으로는
좁히기 어려운 경우가 있다. 예를 들어 `interface`로 정의한 두 객체를
구분하거나, 검사 로직이 복잡해서 함수로 빼고 싶을 때다.

이럴 때 **함수를 직접 만들어서 타입 가드로 쓸 수 있다**.
반환 타입에 `매개변수 is 타입` 형태(타입 술어, type predicate)를 적어주면 된다.

## 기본 형태

```ts
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

// 반환 타입이 boolean이 아니라 "animal is Fish"
function isFish(animal: Fish | Bird): animal is Fish {
  return (animal as Fish).swim !== undefined;
}

function move(animal: Fish | Bird) {
  if (isFish(animal)) {
    animal.swim(); // 이 블록에서 animal은 Fish로 좁혀진다
  } else {
    animal.fly(); // Bird
  }
}
```

`isFish`가 `true`를 반환하면, TypeScript는 그 블록 안에서
`animal`을 `Fish`로 좁혀준다. 반환 타입을 그냥 `boolean`으로 적으면
이 좁히기가 동작하지 않는다는 게 핵심이다.

## boolean과 뭐가 다른가

```ts
// ❌ boolean 반환 — 좁혀지지 않음
function isFishBad(animal: Fish | Bird): boolean {
  return (animal as Fish).swim !== undefined;
}

function moveBad(animal: Fish | Bird) {
  if (isFishBad(animal)) {
    animal.swim(); // ❌ 에러: animal은 여전히 Fish | Bird
  }
}
```

반환값은 둘 다 런타임에선 `true`/`false`로 똑같다.
차이는 **타입 시스템에 "이게 참이면 Fish다"라는 정보를 알려주느냐**다.
`is` 술어가 그 정보를 컴파일러에 넘겨준다.

## 실전 예시 — 좁히기 로직 재사용

검사 조건이 복잡하거나 여러 곳에서 쓰일 때 함수로 빼면 깔끔하다.

```ts
interface ApiSuccess {
  status: "ok";
  data: string;
}

interface ApiError {
  status: "error";
  message: string;
}

type ApiResponse = ApiSuccess | ApiError;

function isSuccess(res: ApiResponse): res is ApiSuccess {
  return res.status === "ok";
}

function handle(res: ApiResponse) {
  if (isSuccess(res)) {
    console.log(res.data); // ApiSuccess
  } else {
    console.log(res.message); // ApiError
  }
}
```

## null 걸러내기 패턴

`filter`로 `null`/`undefined`를 거를 때 특히 유용하다.

```ts
function isNotNull<T>(value: T | null): value is T {
  return value !== null;
}

const names: (string | null)[] = ["짱구", null, "둘리", null, "또치"];

// 타입 가드 없이 filter하면 결과 타입이 (string | null)[] 그대로다
const filtered = names.filter(isNotNull);
// filtered의 타입: string[] ✅
```

평범하게 `names.filter((n) => n !== null)`로 거르면
TypeScript는 결과를 여전히 `(string | null)[]`로 본다.
타입 가드 함수를 넘기면 `string[]`으로 정확히 좁혀준다.

## 비교 표

| 방식 | 반환 타입 | 좁히기 동작 |
| --- | --- | --- |
| 일반 함수 | `boolean` | ❌ 안 됨 |
| 타입 가드 함수 | `x is T` | ✅ 됨 |

## 헷갈렸던 포인트

- **`is` 술어는 개발자가 보증하는 거다.** 컴파일러가 함수 내부 로직이
  정말 맞는지 검증하진 않는다. 아래처럼 거짓말을 써도 통과한다.

  ```ts
  function isFishWrong(animal: Fish | Bird): animal is Fish {
    return true; // 항상 Fish라고 우긴다 — 컴파일러는 믿어버린다
  }
  ```

  그래서 술어 함수 내부 로직은 신중하게 작성해야 한다.

- **`as` 단언이 함수 안에 들어가는 건 괜찮다.** `(animal as Fish).swim`
  처럼 내부에서 단언을 쓰는 건 흔한 패턴이다. 검사 로직을 함수 안에
  가두고, 바깥에서는 깔끔하게 `is`로 좁히는 효과를 얻는다.

- **클래스 메서드에도 쓸 수 있다.** `this is T` 형태로 쓰면 메서드가
  `this`의 타입을 좁혀준다.

  ```ts
  class Box<T> {
    value?: T;
    hasValue(): this is { value: T } {
      return this.value !== undefined;
    }
  }
  ```

## 요약

- 반환 타입을 `매개변수 is 타입`으로 적으면 사용자 정의 타입 가드가 된다
- `boolean` 반환과 런타임 동작은 같지만, 좁히기는 `is`에서만 된다
- 복잡한 검사 로직을 함수로 빼서 재사용할 때 유용
- `filter`로 `null` 거를 때 결과 타입까지 정확히 좁혀준다
- 술어가 맞는지는 컴파일러가 검증 안 하므로 내부 로직은 신중하게

참고: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
