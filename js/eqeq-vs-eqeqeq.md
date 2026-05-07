# `==` vs `===` 와 타입 강제 변환 (Type Coercion)

`===`는 값과 타입을 모두 비교한다. `==`는 타입이 다르면 먼저 변환(coercion)한 뒤 비교한다.

```js
1 == '1'   // true  (문자열 '1' → 숫자 1 으로 변환)
1 === '1'  // false (타입이 다르므로 바로 false)

null == undefined   // true
null === undefined  // false

0 == false  // true  (false → 0)
0 === false // false

'' == false  // true  ('' → 0, false → 0)
```

## 헷갈렸던 포인트

- `null == undefined`는 **true**지만 `null === undefined`는 false.
- `NaN == NaN`은 `==`도 `===`도 **false**. NaN 체크는 `Number.isNaN()` 사용.
- `==`의 변환 규칙은 직관적이지 않아서 팀 컨벤션상 `===`를 기본으로 쓴다.

## 요약

| 연산자 | 타입 변환 | 권장 |
|--------|-----------|------|
| `==`   | O         | X (특별한 이유 없으면 피함) |
| `===`  | X         | O |