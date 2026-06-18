# 유니온 vs 인터섹션 타입 (Union vs Intersection)

타입을 합칠 때 `|`(유니온)과 `&`(인터섹션) 둘 다 쓰는데, 방향이 정반대다.
한 줄로 정리하면 **유니온은 "둘 중 하나", 인터섹션은 "둘 다"**이다.

## 유니온 (`|`) — OR

여러 타입 중 **하나**가 될 수 있다는 뜻이다.

```ts
type Id = string | number;

const a: Id = "abc"; // OK
const b: Id = 123; // OK
const c: Id = true; // ❌ string도 number도 아님
```

값이 둘 중 무엇이든 들어올 수 있으니, **공통으로 보장되는 멤버만** 바로 쓸 수 있다.

```ts
type Cat = { name: string; meow: () => void };
type Dog = { name: string; bark: () => void };

function speak(animal: Cat | Dog) {
  console.log(animal.name); // OK — 둘 다 name이 있다
  animal.meow(); // ❌ Dog일 수도 있어서 meow를 장담 못 함
}
```

`meow`를 쓰려면 타입을 좁혀야(narrowing) 한다.

```ts
function speak(animal: Cat | Dog) {
  if ("meow" in animal) {
    animal.meow(); // 여기서 animal은 Cat
  } else {
    animal.bark(); // 여기서 animal은 Dog
  }
}
```

## 인터섹션 (`&`) — AND

여러 타입을 **전부 만족**해야 한다. 즉 모든 멤버를 다 합친 타입이 된다.

```ts
type Nameable = { name: string };
type Ageable = { age: number };

type Person = Nameable & Ageable;

const 짱구: Person = { name: "짱구", age: 5 }; // 둘 다 있어야 함
```

`name`만 있거나 `age`만 있으면 에러다. 객체 타입을 조합해 더 큰 타입을 만들 때 쓴다.

## 헷갈리는 지점: "합친다"의 방향

이름만 보면 인터섹션(교집합)이 더 좁을 것 같은데, **객체 타입에선 인터섹션이 멤버가 더 많다.**
집합으로 생각하면 거꾸로라 헷갈린다. 값의 관점으로 보면 정리된다.

- 유니온 `Cat | Dog`: 들어올 수 있는 **값의 범위가 넓다** → 대신 쓸 수 있는 멤버는 좁다(교집합).
- 인터섹션 `Cat & Dog`: **멤버를 다 합친다** → 대신 그걸 다 만족하는 값은 드물다.

```ts
type CatDog = Cat & Dog;
// name, meow, bark를 전부 가진 객체만 CatDog
```

## 원시 타입끼리의 인터섹션은 never

객체가 아니라 `string & number`처럼 양립 불가능한 원시 타입을 `&`로 묶으면
"둘 다 만족하는 값"이 존재하지 않으므로 `never`가 된다.

```ts
type Impossible = string & number; // never
```

## 비교표

| | 유니온 `\|` | 인터섹션 `&` |
|---|---|---|
| 의미 | 둘 중 하나 (OR) | 둘 다 (AND) |
| 값의 범위 | 넓어진다 | 좁아진다 |
| 접근 가능한 멤버 | 공통 멤버만 | 모든 멤버 |
| 주 용도 | 상태/분기 모델링 | 타입 조합·확장 |
| 원시 타입 충돌 시 | 그대로 유니온 | `never` |

## 헷갈렸던 포인트

- **유니온에서 바로 못 쓰는 멤버**는 버그가 아니라 안전장치다. `Cat | Dog`에서 `meow`가
  막히는 건 런타임에 Dog가 올 수도 있기 때문이다. `in`, `typeof`, 판별 유니온 등으로 좁혀서 쓴다.

- **인터섹션 = "교집합"이라는 단어에 속지 말 것.** 객체 타입에선 멤버가 합쳐져서 오히려 커진다.
  "두 타입을 동시에 만족"으로 외우는 게 안전하다.

- **인터섹션으로 같은 키를 충돌시키면 `never`.** `{ x: string } & { x: number }`의 `x`는
  `string & number`라 `never`가 된다. 에러 없이 통과했다가 값을 넣을 때 터진다.

- **확장은 `&`, 분기는 `|`.** 기존 객체 타입에 필드를 더 붙이고 싶으면 인터섹션,
  "이거 아니면 저거" 같은 경우의 수를 표현하려면 유니온이다.

## 요약

- 유니온 `|` = 둘 중 하나, 값 범위가 넓고 공통 멤버만 바로 접근 가능
- 인터섹션 `&` = 둘 다, 멤버를 전부 합친 타입
- 객체 타입에선 인터섹션이 멤버가 더 많다 (이름과 반대라 헷갈림)
- 양립 불가능한 원시 타입을 `&`하면 `never`
- 분기/상태는 유니온, 타입 조합/확장은 인터섹션

참고: https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types
