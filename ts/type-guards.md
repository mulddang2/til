# 타입 가드 (Type Guards)

유니온 타입을 다루다 보면 "지금 이 값이 둘 중 어느 타입이지?"를 좁혀야 할 때가 온다.
이때 쓰는 게 **타입 가드(type guard)**다. 조건문 안에서 타입을 좁히면(narrowing),
TypeScript가 그 블록 안에서 타입을 더 구체적으로 추론해준다.

대표적으로 `typeof`, `instanceof`, `in` 세 가지가 있다.

## typeof — 원시 타입 좁히기

`string`, `number`, `boolean` 같은 **원시 타입(primitive)**을 구분할 때 쓴다.

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    // 이 블록 안에서 id는 string으로 좁혀진다
    console.log(id.toUpperCase());
  } else {
    // 여기선 number
    console.log(id.toFixed(2));
  }
}

printId("jjanggu");
printId(42);
```

`typeof`가 반환할 수 있는 값은 `"string"`, `"number"`, `"bigint"`, `"boolean"`,
`"symbol"`, `"undefined"`, `"object"`, `"function"` 이렇게 정해져 있다.
이 문자열로 비교하면 TypeScript가 알아서 타입을 좁혀준다.

## instanceof — 클래스 인스턴스 구분

`instanceof`는 어떤 값이 **특정 클래스의 인스턴스인지** 확인한다.

```ts
class Dog {
  bark() {
    console.log("멍멍");
  }
}

class Cat {
  meow() {
    console.log("야옹");
  }
}

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // animal은 Dog로 좁혀짐
  } else {
    animal.meow(); // Cat
  }
}

speak(new Dog());
```

`new`로 만든 인스턴스나 `Date`, `Array`, `Error` 같은 내장 클래스 판별에 잘 맞는다.
단, 클래스가 아니라 평범한 객체 리터럴엔 못 쓴다.

## in — 프로퍼티 존재 여부

`in` 연산자는 객체에 **특정 키가 있는지**로 타입을 좁힌다.
클래스가 아닌 객체 형태 유니온에서 특히 유용하다.

```ts
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim(); // Fish로 좁혀짐
  } else {
    animal.fly(); // Bird
  }
}
```

`swim` 프로퍼티가 있으면 `Fish`, 없으면 `Bird`로 추론한다.

## 비교 표

| 가드 | 주 용도 | 대상 |
| --- | --- | --- |
| `typeof` | 원시 타입 구분 | string / number / boolean 등 |
| `instanceof` | 클래스 인스턴스 구분 | `new`로 만든 객체, 내장 클래스 |
| `in` | 프로퍼티 존재로 구분 | 객체 리터럴, 인터페이스 유니온 |

## 헷갈렸던 포인트

- **`typeof null`은 `"object"`다.** JavaScript의 오래된 버그라 그렇다.
  그래서 `null` 체크는 `typeof`로 하면 안 되고 `=== null`이나 `== null`로 따로 해야 한다.

  ```ts
  function check(x: string | null) {
    if (typeof x === "object") {
      // x는 null로 좁혀진다 (string은 여기 안 들어옴)
    }
  }
  ```

- **`instanceof`는 클래스 전용이다.** `interface`나 `type`으로만 정의한 타입은
  런타임에 존재하지 않으므로 `instanceof`로 검사할 수 없다. 이럴 땐 `in`을 쓴다.

- **배열은 `typeof`로 `"object"`로 나온다.** 배열 판별은 `Array.isArray()`를 써야 한다.

  ```ts
  function flatten(x: number | number[]) {
    if (Array.isArray(x)) {
      return x.reduce((a, b) => a + b, 0);
    }
    return x;
  }
  ```

## 요약

- 타입 가드는 조건문으로 유니온 타입을 좁히는(narrowing) 기법
- `typeof` → 원시 타입 (단, `null`은 `"object"`라 주의)
- `instanceof` → 클래스 인스턴스 (interface/type엔 못 씀)
- `in` → 프로퍼티 존재 여부로 객체 구분
- 배열은 `Array.isArray()`로 따로 판별

참고: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
