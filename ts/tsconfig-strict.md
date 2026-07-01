# tsconfig strict 옵션 정리 (strict)

TypeScript를 제대로 쓰려면 `tsconfig.json`의 `strict` 옵션을 켜야 한다.
`strict: true` 하나만 켜면 아래 여러 개의 세부 옵션이 한꺼번에 켜진다.
새 프로젝트라면 처음부터 켜고 시작하는 게 정신 건강에 이롭다.

## strict는 묶음 스위치다

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "strict": true
  }
}
```

`strict: true`는 다음 옵션들을 전부 켜는 상위 스위치다.

| 세부 옵션 | 하는 일 |
| --- | --- |
| `strictNullChecks` | `null` / `undefined`를 별도 타입으로 엄격히 구분 |
| `strictFunctionTypes` | 함수 파라미터 타입을 반공변(contravariant)으로 검사 |
| `strictBindCallApply` | `bind` / `call` / `apply` 인자 타입 검사 |
| `strictPropertyInitialization` | 클래스 필드 초기화 강제 |
| `noImplicitAny` | 암묵적 `any` 금지 |
| `noImplicitThis` | `this`가 암묵적 `any`면 에러 |
| `alwaysStrict` | 출력 파일마다 `"use strict"` 추가 |
| `useUnknownInCatchVariables` | `catch` 변수 타입을 `unknown`으로 |

개별로 끄고 싶으면 `strict: true`를 켠 뒤 원하는 것만 `false`로 덮어쓰면 된다.

## 핵심 옵션 하나씩

### strictNullChecks

가장 체감이 큰 옵션이다. 이게 꺼져 있으면 모든 타입에 `null`이 몰래 들어갈 수 있다.

```ts
// strictNullChecks: true
function getLength(text: string) {
  return text.length;
}

let value: string | null = null;
getLength(value); // ❌ 에러: null이 string 자리에 못 들어감

if (value !== null) {
  getLength(value); // ✅ 좁혀졌으니 통과
}
```

`null` 체크를 강제해서 런타임의 "Cannot read property of undefined" 류 에러를 대부분 컴파일 단계에서 잡아준다.

### noImplicitAny

타입을 안 적어서 `any`로 추론되면 바로 에러를 낸다.

```ts
// ❌ 파라미터 name이 암묵적 any
function greet(name) {
  return `안녕 ${name}`;
}

// ✅ 타입을 명시
function greet(name: string) {
  return `안녕 ${name}`;
}
```

`any`가 코드 곳곳에 조용히 퍼지는 걸 막아준다.

### strictPropertyInitialization

클래스 필드를 선언만 하고 초기화 안 하면 에러다. (`strictNullChecks`와 같이 켜져 있어야 동작한다.)

```ts
class Character {
  name: string; // ❌ 초기화 안 됨

  constructor() {
    // this.name 할당을 깜빡함
  }
}
```

```ts
class Character {
  name: string;

  constructor(name: string) {
    this.name = name; // ✅ 생성자에서 할당
  }
}
```

당장 값이 없으면 `name!: string`처럼 단언(assertion)을 붙이거나 `name?: string`으로 옵셔널 처리한다.

### useUnknownInCatchVariables

`catch (e)`의 `e` 타입을 `any`가 아니라 `unknown`으로 잡는다. 그래서 바로 프로퍼티에 접근 못 하고 좁혀야 한다.

```ts
try {
  throw new Error("뭔가 터짐");
} catch (e) {
  // e는 unknown
  if (e instanceof Error) {
    console.log(e.message); // ✅ 좁힌 뒤 접근
  }
}
```

## 자주 같이 쓰는 옵션 (strict 밖)

`strict`에 포함되진 않지만 같이 켜두면 좋은 것들이다.

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,        // 안 쓰는 지역 변수 에러
    "noUnusedParameters": true,    // 안 쓰는 파라미터 에러
    "noImplicitReturns": true,     // 일부 경로만 return하면 에러
    "noFallthroughCasesInSwitch": true, // switch break 누락 방지
    "noUncheckedIndexedAccess": true    // 인덱스 접근 결과에 undefined 포함
  }
}
```

특히 `noUncheckedIndexedAccess`는 배열/객체 인덱스 접근 결과를 `T | undefined`로 만들어줘서 `strictNullChecks`의 빈틈을 메워준다.

```ts
// noUncheckedIndexedAccess: true
const arr = ["짱구", "둘리"];
const first = arr[0]; // 타입: string | undefined
console.log(first.length); // ❌ undefined 가능성 때문에 에러
```

## 헷갈렸던 포인트

- **`strict: true`를 켜도 개별 옵션을 끌 수 있다**. 예를 들어 레거시 코드 마이그레이션 중이라면 `"strict": true` + `"strictNullChecks": false`처럼 임시로 일부만 완화한다. 나중에 하나씩 다시 켜면 된다.

- **`strictPropertyInitialization`은 `strictNullChecks` 없이는 동작 안 한다**. 둘은 세트로 묶여 있다.

- **`noImplicitAny`는 "명시적 any"는 안 막는다**. 직접 `: any`라고 적은 건 통과한다. 막는 건 타입을 안 적어서 추론된 `any`뿐이다.

- **`noUncheckedIndexedAccess`는 `strict`에 안 들어 있다**. 별도로 켜야 한다. 켜면 안전하지만 인덱스 접근마다 undefined 체크가 늘어나서 처음엔 귀찮게 느껴질 수 있다.

## 요약

- `strict: true` 하나면 8개 세부 옵션이 한 번에 켜진다
- `strictNullChecks`가 체감 1순위 — `null`/`undefined`를 강제로 다루게 함
- `noImplicitAny`로 암묵적 `any` 확산 차단
- 클래스는 `strictPropertyInitialization`으로 필드 초기화 강제
- `strict` 밖의 `noUncheckedIndexedAccess`, `noUnusedLocals` 등도 같이 켜면 더 촘촘해진다
- 새 프로젝트는 처음부터 `strict: true`로 시작하자

참고: https://www.typescriptlang.org/tsconfig#strict
