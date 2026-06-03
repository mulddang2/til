# interface vs type

TypeScript에서 타입을 정의할 때 `interface`와 `type` 중 무엇을 쓸지 늘 헷갈린다.
결론부터 말하면 **대부분의 상황에서 둘 다 된다** — 차이는 몇 가지 엣지 케이스에서 생긴다.

## 공통점: 거의 다 똑같이 쓸 수 있다

```ts
interface UserInterface {
  name: string;
  age: number;
}

type UserType = {
  name: string;
  age: number;
};

const 짱구: UserInterface = { name: "짱구", age: 5 };
const 둘리: UserType = { name: "둘리", age: 5 };
```

함수 파라미터, 반환 타입, 클래스 구현 등 대부분의 자리에서 교환해서 써도 동작한다.

## interface — 확장

### `extends`로 확장

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const 또치: Dog = { name: "또치", breed: "비글" };
```

### 선언 병합 (Declaration Merging)

`interface`는 **같은 이름으로 여러 번 선언하면 자동으로 합쳐진다**.

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}

// 실제 타입: { name: string; age: number }
const 짱구: User = { name: "짱구", age: 5 };
```

라이브러리 타입을 확장할 때 유용하다. 예를 들어 `Express`의 `Request` 타입에 커스텀 필드를 붙일 때 이 방식을 쓴다.

## type — 더 유연한 표현

### 유니온 타입

```ts
type Status = "loading" | "success" | "error";

type StringOrNumber = string | number;
```

`interface`로는 유니온을 직접 표현할 수 없다.

### 인터섹션 타입 (`&`)

```ts
type Animal = { name: string };
type Pet = { owner: string };

type PetAnimal = Animal & Pet;

const 짱구Pet: PetAnimal = { name: "짱구", owner: "짱구아빠" };
```

`interface`의 `extends`와 유사하지만, `type`의 `&`는 **기존 타입을 수정하지 않고 새 타입을 만든다**.

### 튜플, 원시 타입 별칭

```ts
type Point = [number, number];
type ID = string;
type Callback = (name: string) => void;
```

이런 형태는 `interface`로 못 쓴다.

### 조건부 타입 / 매핑 타입

```ts
type IsString<T> = T extends string ? true : false;

type ReadOnly<T> = {
  readonly [K in keyof T]: T[K];
};
```

고급 타입 유틸리티를 만들 땐 `type`만 가능하다.

## interface vs type 비교표

| 기능 | interface | type |
|------|-----------|------|
| 객체 타입 정의 | ✅ | ✅ |
| 클래스 `implements` | ✅ | ✅ |
| 확장 | `extends` | `&` |
| 선언 병합 | ✅ | ❌ |
| 유니온 타입 | ❌ | ✅ |
| 튜플 / 원시 타입 별칭 | ❌ | ✅ |
| 조건부 / 매핑 타입 | ❌ | ✅ |

## 언제 뭘 쓰나

**`interface` 우선** 권장 — React 공식 문서, TypeScript 공식 핸드북 모두 객체 형태 타입엔 `interface`를 기본으로 추천한다. 선언 병합 덕분에 외부에 열려 있어서 라이브러리 작성에도 적합하다.

**`type`을 써야 할 때**:
- 유니온/인터섹션이 필요할 때
- 튜플, 함수 타입, 원시 타입에 이름을 붙일 때
- 조건부 타입이나 매핑 타입을 만들 때

## 헷갈렸던 포인트

- **`interface extends` vs `type &`**: 둘 다 합치는 효과지만, `interface extends`는 구현 불가 시 에러를 더 명확하게 낸다. `type &`는 충돌하는 타입을 `never`로 만들어버리고 에러를 바로 내지 않을 수 있다.
  
  ```ts
  interface A { value: string }
  interface B extends A { value: number } // ❌ 에러: 즉시 잡힘

  type C = { value: string } & { value: number };
  // value: never — 에러 없이 통과, 쓸 때 터짐
  ```

- **선언 병합은 `type`에서 안 된다**. 같은 이름의 `type`을 두 번 선언하면 즉시 에러.

- **`implements`는 둘 다 된다**. 클래스가 `interface`든 `type`이든 `implements` 가능. 단, 유니온이 들어간 `type`은 `implements` 불가.

## 요약

- 객체 모양 타입 → `interface` 기본
- 유니온, 튜플, 고급 타입 → `type`
- 라이브러리/모듈 타입 확장 → `interface` (선언 병합)
- `extends` 충돌 에러는 `interface`가 더 빨리 잡아준다

참고: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
