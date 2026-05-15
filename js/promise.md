# Promise

비동기 작업의 **결과를 나중에 받겠다는 약속**을 객체로 표현한 것이다.
콜백 지옥을 탈출하기 위해 ES2015에서 도입됐다.

## 세 가지 상태

| 상태 | 의미 |
|---|---|
| `pending` | 대기 중 — 아직 결과 없음 |
| `fulfilled` | 성공 — `resolve(값)` 호출됨 |
| `rejected` | 실패 — `reject(에러)` 호출됨 |

한번 `fulfilled` 또는 `rejected`가 되면 **다시 바뀌지 않는다**.

## 기본 사용법

```js
const p = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("짱구 등장!");
  } else {
    reject(new Error("둘리가 방해함"));
  }
});

p.then((result) => {
  console.log(result); // "짱구 등장!"
}).catch((err) => {
  console.error(err.message);
}).finally(() => {
  console.log("항상 실행됨");
});
```

- `.then()` — fulfilled 됐을 때 실행
- `.catch()` — rejected 됐을 때 실행
- `.finally()` — 성공/실패 상관없이 마지막에 실행

## 체이닝 (Chaining)

`.then()`은 항상 새 Promise를 반환한다. 그래서 이어서 연결할 수 있다.

```js
fetch("/api/user")
  .then((res) => res.json())
  .then((data) => {
    console.log(data.name); // "짱구"
    return data.age;
  })
  .then((age) => {
    console.log(age); // 5
  })
  .catch((err) => {
    console.error("어디선가 실패:", err);
  });
```

체인 중 어느 단계에서든 에러가 나면 **가장 가까운 `.catch()`로 점프**한다.

## 여러 Promise를 한꺼번에

### `Promise.all` — 전부 성공해야 통과

```js
const p1 = Promise.resolve("짱구");
const p2 = Promise.resolve("둘리");
const p3 = Promise.resolve("또치");

Promise.all([p1, p2, p3]).then(([a, b, c]) => {
  console.log(a, b, c); // "짱구 둘리 또치"
});
```

하나라도 실패하면 전체가 `rejected`로 떨어진다.

### `Promise.allSettled` — 결과 가리지 않고 전부 수집

```js
Promise.allSettled([
  Promise.resolve("짱구"),
  Promise.reject(new Error("둘리 실패")),
]).then((results) => {
  results.forEach((r) => console.log(r.status, r.value ?? r.reason));
  // "fulfilled" "짱구"
  // "rejected"  Error: 둘리 실패
});
```

일부가 실패해도 나머지 결과를 모두 받고 싶을 때 쓴다.

### `Promise.race` — 가장 먼저 끝난 것만

```js
Promise.race([
  new Promise((res) => setTimeout(() => res("짱구"), 100)),
  new Promise((res) => setTimeout(() => res("둘리"), 200)),
]).then((winner) => {
  console.log(winner); // "짱구" (먼저 완료)
});
```

타임아웃 처리에 유용하다.

### `Promise.any` — 하나라도 성공하면 통과

```js
Promise.any([
  Promise.reject(new Error("실패1")),
  Promise.resolve("또치"),
  Promise.resolve("짱구"),
]).then((first) => {
  console.log(first); // "또치" (첫 번째 성공)
});
```

모두 실패하면 `AggregateError`가 던져진다.

## 헷갈렸던 포인트

- **`new Promise` 안의 코드는 동기 실행**된다. executor 함수(`(resolve, reject) => ...`)는 즉시 실행된다. 비동기가 되는 건 `.then()` 콜백이다.
- **`.catch()`는 `.then(null, handler)`의 축약**이다. 둘은 동일하게 동작하지만 `.catch()`가 훨씬 읽기 편하다.
- **체인에서 return을 빠뜨리면** 다음 `.then()`은 `undefined`를 받는다.
- **`Promise.all` vs `Promise.allSettled`**: 하나라도 실패하면 중단할 때는 `all`, 부분 실패를 허용하고 싶을 때는 `allSettled`.

## 요약

- Promise는 비동기 작업의 미래 값을 담는 객체
- `pending → fulfilled | rejected`, 상태는 한 번만 바뀐다
- `.then().catch().finally()` 체인으로 결과 처리
- 여러 개 동시 처리: `all` / `allSettled` / `race` / `any`
- `async/await`는 Promise 위에 만들어진 문법 설탕

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise
