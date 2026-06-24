# keyof & 인덱스 접근 타입 (Indexed Access)

타입을 직접 손으로 다 적는 대신, **이미 있는 타입에서 정보를 뽑아 쓰는** 문법이 있다.
`keyof`는 객체 타입의 키를 유니온으로 뽑고, 인덱스 접근(indexed access)은 특정 속성의 타입을 뽑는다.
이 둘을 조합하면 "값과 타입을 따로 관리하지 않아도 되는" 코드가 된다.

## `keyof` — 키를 유니온으로 뽑는다

`keyof T`는 `T`의 모든 속성 이름을 문자열 리터럴 유니온으로 만든다.

```ts
interface User {
  name: string;
  age: number;
  isAdmin: boolean;
}

type UserKey = keyof User;
// "name" | "age" | "isAdmin"

const key: UserKey = "name"; // ✅
// const bad: UserKey = "email"; // ❌ User에 없는 키
```

키가 늘어나면 `UserKey`도 자동으로 같이 늘어난다. 손으로 유니온을 관리할 필요가 없다.

## 인덱스 접근 타입 — 속성의 타입을 뽑는다

`T[K]`는 `T`에서 `K` 속성의 **타입**을 꺼낸다. 값이 아니라 타입을 인덱싱한다고 보면 된다.

```ts
interface User {
  name: string;
  age: number;
}

type NameType = User["name"]; // string
type AgeType = User["age"];   // number

// 유니온으로도 뽑을 수 있다
type ValueType = User["name" | "age"]; // string | number

// keyof와 조합하면 모든 값 타입의 유니온
type AllValues = User[keyof User]; // string | number
```

배열 타입에서 원소 타입을 뽑을 때도 자주 쓴다.

```ts
const animals = ["짱구", "둘리", "또치"];
type Animal = (typeof animals)[number]; // string

type Tuple = [number, string, boolean];
type Second = Tuple[1]; // string
```

## 둘을 합치면: 안전한 get 함수

`keyof` + 인덱스 접근의 대표 예시가 제네릭 `getProperty`다.

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const 짱구 = { name: "짱구", age: 5 };

const n = getProperty(짱구, "name"); // string으로 추론
const a = getProperty(짱구, "age");  // number로 추론
// getProperty(짱구, "email"); // ❌ "email"은 keyof에 없음
```

`key`를 `keyof T`로 제약하니 잘못된 키는 컴파일 단계에서 막힌다.
반환 타입도 `T[K]`라 키마다 정확한 타입이 나온다. `string`이나 `any`로 뭉개지 않는다.

## keyof vs 인덱스 접근 비교

| | 문법 | 뽑는 것 | 결과 예시 |
| --- | --- | --- | --- |
| keyof | `keyof T` | 키들의 유니온 | `"name" \| "age"` |
| 인덱스 접근 | `T[K]` | 그 키의 값 타입 | `string` |
| 조합 | `T[keyof T]` | 모든 값 타입의 유니온 | `string \| number` |

## 헷갈렸던 포인트

- **`keyof`는 타입에, `Object.keys`는 값에 쓴다.** `keyof`는 컴파일 타임에 키 이름을
  타입으로 뽑고, `Object.keys`는 런타임에 키 배열을 만든다. 둘은 완전히 다른 세계다.
  심지어 `Object.keys`의 반환 타입은 `string[]`이라 `keyof`만큼 안 좁혀진다.

- **`obj[key]`의 `key`는 변수면 안 된다.** 인덱스 접근 타입 `T[K]`의 `K`는 **타입**이어야 한다.
  ```ts
  const k = "name";
  type T1 = User[typeof k]; // ✅ typeof로 타입을 꺼내면 됨
  // type T2 = User[k];     // ❌ k는 값이라 안 됨
  ```

- **숫자 인덱스는 `[number]`로 접근한다.** 배열/튜플에서 원소 타입을 뽑을 땐
  `Arr[number]`처럼 쓴다. `Arr[0]`은 튜플에서 첫 번째 원소 타입만 콕 집는다.

- **`keyof any`는 `string | number | symbol`이다.** 객체 키로 쓸 수 있는 모든 타입을
  의미한다. `Record` 같은 유틸리티가 내부에서 이걸 제약 조건으로 쓴다.

## 요약

- `keyof T` → 객체 타입의 키들을 문자열 리터럴 유니온으로 뽑는다.
- `T[K]` → 인덱스 접근, 특정 키의 값 타입을 뽑는다.
- `T[keyof T]` → 모든 값 타입의 유니온.
- `<T, K extends keyof T>(obj, key): T[K]` 패턴으로 타입 안전한 속성 접근을 만든다.
- `keyof`는 타입, `Object.keys`는 런타임 값 — 헷갈리지 말 것.

참고: https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
