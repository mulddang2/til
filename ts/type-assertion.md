# 타입 단언 (Type Assertion) & non-null assertion (`!`)

타입 단언(type assertion)은 컴파일러한테 "이 값의 타입은 내가 더 잘 안다"고 알려주는 문법이다.
`as`나 non-null assertion(`!`)으로 추론된 타입을 **개발자가 직접 덮어쓴다**.

주의할 점이 하나 있다. 단언은 **런타임에 아무 일도 안 한다**.
값을 변환하지도, 검사하지도 않는다. 그냥 컴파일러의 입을 막을 뿐이다.

## `as` — 타입을 직접 지정

가장 기본적인 형태다. `값 as 타입`으로 쓴다.

```ts
const value: unknown = "짱구";

// unknown이라 바로 못 쓰지만, string이라고 단언하면 통과
const len = (value as string).length;
```

DOM을 다룰 때 자주 쓴다. `getElementById`는 `HTMLElement | null`을 반환하는데,
실제로는 `input` 요소인 걸 아는 경우다.

```ts
const input = document.getElementById("name") as HTMLInputElement;
input.value = "둘리"; // HTMLElement에는 value가 없으니 단언이 필요
```

### `<타입>값` 꺾쇠 문법

```ts
const len = (<string>value).length;
```

같은 의미지만 JSX와 충돌하기 때문에 `.tsx`에서는 못 쓴다.
요즘은 그냥 `as`로 통일하는 분위기다.

## non-null assertion (`!`) — null/undefined 아님을 보장

값 뒤에 `!`를 붙이면 "이건 절대 `null`이나 `undefined`가 아니다"라고 단언한다.

```ts
function getLength(text?: string) {
  // text는 string | undefined
  return text!.length; // ! 로 undefined 아님을 단언
}
```

DOM에서도 자주 보인다.

```ts
const el = document.querySelector(".box")!; // Element | null → Element
el.classList.add("active");
```

`!`는 `as NonNullable<T>`의 짧은 버전이라고 보면 된다.
다만 **정말로 null이면 런타임에 그냥 터진다**. 컴파일러가 안 막아줄 뿐이다.

## 이중 단언 (Double Assertion)

타입이 너무 멀리 떨어져 있으면 `as`가 한 번에 안 먹는다.
이럴 때 `as unknown as 타입`으로 우회한다.

```ts
const num = 42;

// const str = num as string; // ❌ number와 string은 안 겹침
const str = num as unknown as string; // ⚠️ 강제로 통과 (위험)
```

컴파일러가 막는 걸 억지로 뚫는 거라, 정말 확신할 때만 써야 한다.

## `as const` 와는 다르다

이름이 비슷해서 헷갈리는데 `as const`는 단언이라기보다 **리터럴 고정**이다.
값을 좁혀서 읽기 전용으로 만든다. (별도 노트 참고)

```ts
const colors = ["red", "green"] as const;
// readonly ["red", "green"] 로 좁혀짐
```

## 단언 vs 타입 가드

| | 타입 단언 (`as` / `!`) | 타입 가드 (`typeof`, `in` 등) |
| --- | --- | --- |
| 런타임 검사 | ❌ 없음 | ✅ 실제로 검사함 |
| 틀렸을 때 | 런타임 에러 (조용히 통과) | 안전하게 분기 |
| 안전성 | 개발자 책임 | 컴파일러가 보장 |

가능하면 단언보다 타입 가드가 안전하다.
단언은 "컴파일러가 모르는 정보를 내가 알 때"만 쓰는 탈출구다.

## 헷갈렸던 포인트

- **단언은 값을 바꾸지 않는다.** `"123" as number`처럼 써도 실제 변환은 안 일어난다.
  (애초에 이건 타입이 안 겹쳐서 에러가 난다.) 숫자로 바꾸려면 `Number("123")`을 써야 한다.

- **`!`는 그냥 미룬 폭탄이다.** null 가능성을 코드에서 없앤 게 아니라 컴파일러 경고만 끈 거라,
  실제로 null이 들어오면 `Cannot read properties of null` 에러로 터진다.
  가능하면 옵셔널 체이닝(`?.`)이나 명시적 null 체크로 처리하는 게 낫다.

- **`as`로 좁히기만 되는 게 아니라 넓히기도 막힌다.** 서로 전혀 안 겹치는 타입끼리는
  단언이 거부된다. 그래서 `as unknown as T` 같은 이중 단언이 등장한다 — 이게 보인다면 설계를 의심해보자.

- **`as`는 만능 도구가 아니다.** 단언이 많아질수록 타입 시스템을 끄고 있다는 신호다.
  `any` 남발과 비슷한 냄새가 난다.

## 요약

- `as`는 추론된 타입을 개발자가 덮어쓰는 단언 — 런타임 변환은 없다.
- `!`는 null/undefined 아님을 단언하는 짧은 문법 — 틀리면 런타임에 터진다.
- 안 겹치는 타입은 `as unknown as T`로 우회하지만, 위험 신호다.
- 단언보다 타입 가드가 안전하다. 단언은 컴파일러가 모르는 걸 내가 확신할 때만.

참고: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions
