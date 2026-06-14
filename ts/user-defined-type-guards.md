# 사용자 정의 타입 가드 (User-Defined Type Guards)

`typeof`, `instanceof`, `in` 같은 기본 타입 가드(type guard)로는 부족할 때가 있다.
좀 더 복잡한 조건으로 타입을 좁히고(narrowing) 싶을 때, **반환 타입에 `x is T`를 적은 함수**를
직접 만들 수 있다. 이걸 사용자 정의 타입 가드라고 한다.

## `x is T` — 타입 술어 (Type Predicate)

함수의 반환 타입 자리에 `매개변수 is 타입` 형태를 쓴다. 이걸 **타입 술어(type predicate)**라고 부른다.

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
    animal.swim(); // 여기서 animal은 Fish로 좁혀진다
  } else {
    animal.fly(); // Bird
  }
}
```

`isFish`가 `true`를 반환하면 TypeScript는 그 블록 안에서 `animal`을 `Fish`로 좁혀준다.
반대로 `else`엔 자동으로 `Bird`가 된다.

## 그냥 `boolean`을 반환하면 안 되나?

된다. 하지만 타입이 안 좁혀진다. 이게 핵심 차이다.

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

함수 안에서 아무리 검사를 잘 해도, 반환 타입이 그냥 `boolean`이면
호출하는 쪽은 "참/거짓"만 알 뿐 **어떤 타입인지는 모른다**.
`animal is Fish`라고 적어줘야 컴파일러가 좁혀준다.

## 실전 예시 — 배열 필터링

`Array.filter`에서 특히 유용하다. 술어를 안 쓰면 필터 후에도 타입이 안 좁혀진다.

```ts
type Animal = { name: string; legs?: number };

const animals: (Animal | null)[] = [
  { name: "둘리" },
  null,
  { name: "또치", legs: 2 },
];

// null을 걸러내는 사용자 정의 타입 가드
function isNotNull<T>(value: T | null): value is T {
  return value !== null;
}

const filtered = animals.filter(isNotNull);
// filtered: Animal[]  (null이 제거된 타입으로 좁혀짐)
```

`isNotNull` 대신 `(v) => v !== null`을 넣으면 결과 타입이 `(Animal | null)[]`로 남는다.
술어를 써야 `Animal[]`로 깔끔하게 좁혀진다.

## 비교 표

| 방식 | 반환 타입 | 호출부에서 타입 좁힘 |
| --- | --- | --- |
| 일반 함수 | `boolean` | ❌ 안 됨 |
| 사용자 정의 타입 가드 | `x is T` | ✅ 됨 |

## 헷갈렸던 포인트

- **술어가 거짓말을 해도 컴파일러는 믿는다.** `animal is Fish`라고 적어두면
  TypeScript는 함수 내부 로직이 실제로 맞는지 검사하지 않는다. 그냥 믿어버린다.
  그래서 검사 로직을 잘못 짜면 런타임에 터진다. 술어의 책임은 개발자에게 있다.

  ```ts
  function isFishWrong(animal: Fish | Bird): animal is Fish {
    return true; // 항상 Fish라고 우긴다 — 컴파일러는 그대로 믿음
  }
  ```

- **`in`이나 `typeof`로 충분하면 굳이 안 만들어도 된다.** 간단한 경우엘 기본 가드가 더 깔끔하다.
  여러 곳에서 재사용하거나, 검사 로직이 복잡할 때 함수로 빼는 게 좋다.

- **제네릭(generic)과 같이 쓰면 재사용성이 좋아진다.** 위 `isNotNull`처럼 `<T>`를 붙이면
  어떤 타입의 배열에도 쓸 수 있는 범용 가드가 된다.

- **`asserts` 버전도 있다.** `function assertIsFish(x): asserts x is Fish`처럼 쓰면
  반환값 없이, 통과하면 그 뒤 코드 전체에서 타입이 좁혀진다(어서션 함수, assertion function).
  조건이 안 맞으면 `throw` 해야 한다.

## 요약

- 반환 타입에 `매개변수 is 타입`을 적으면 사용자 정의 타입 가드
- 일반 `boolean` 반환은 호출부에서 타입을 안 좁혀준다 — 술어를 써야 좁혀짐
- `Array.filter`에서 `null` 제거 등에 특히 유용
- 컴파일러는 술어를 검증 없이 믿으니, 검사 로직은 개발자가 책임한다
- 제네릭과 같이 쓰면 범용 가드를 만들 수 있다

참고: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
