# async / await

Promise를 더 읽기 좋게 쓰기 위한 **문법 설탕(syntactic sugar)**이다.
`async/await`를 쓴다고 내부적으로 Promise가 사라지는 게 아니다 — 여전히 Promise 위에서 동작한다.

## 기본 문법

```js
async function fetchUser() {
  const res = await fetch("/api/user");
  const data = await res.json();
  return data;
}
```

- `async` 키워드가 붙은 함수는 **항상 Promise를 반환**한다.
- `await`는 Promise가 settled 될 때까지 **해당 함수 내부 실행을 일시 정지**한다.
- `await`는 반드시 `async` 함수 안에서만 쓸 수 있다 (최상위 `await`는 ES2022 모듈 한정).

### 반환값은 Promise

```js
async function greet() {
  return "짱구 등장!";
}

greet().then(console.log); // "짱구 등장!"
```

`return "짱구 등장!"`이지만 실제론 `Promise.resolve("짱구 등장!")`을 반환한 것과 같다.

## 에러 처리 — try / catch

Promise의 `.catch()`에 해당한다.

```js
async function getUser(id) {
  try {
    const res = await fetch(`/api/user/${id}`);
    if (!res.ok) throw new Error("서버 에러");
    return await res.json();
  } catch (err) {
    console.error("실패:", err.message);
  }
}
```

`await` 중에 Promise가 rejected 되면 `catch` 블록으로 떨어진다.
`finally`도 똑같이 쓸 수 있다.

## 순차 실행 vs 병렬 실행

### 순차 실행 (각 요청이 이전 결과에 의존할 때)

```js
async function sequential() {
  const user = await fetchUser("짱구");     // 완료 후
  const posts = await fetchPosts(user.id); // 여기 시작
  return posts;
}
```

두 번째 요청이 첫 번째 결과에 의존할 때 적합하다.
의존 관계가 없는데 이렇게 쓰면 **불필요하게 느려진다**.

### 병렬 실행 (독립적인 요청 여러 개)

```js
async function parallel() {
  const [짱구Data, 둘리Data] = await Promise.all([
    fetchUser("짱구"),
    fetchUser("둘리"),
  ]);
  return { 짱구Data, 둘리Data };
}
```

`Promise.all`로 묶으면 두 요청이 **동시에 출발**해서 더 빠르다.

### 흔한 실수 — 루프 안에서 순차 실행이 돼버리는 경우

```js
// 느린 버전 — 하나 끝나야 다음 시작
const users = ["짱구", "둘리", "또치"];
for (const name of users) {
  const data = await fetchUser(name); // 직렬
}

// 빠른 버전 — 동시에 출발
const results = await Promise.all(users.map((name) => fetchUser(name)));
```

## async/await vs Promise 체인 비교

```js
// Promise 체인
function getPostTitle() {
  return fetchUser("짱구")
    .then((user) => fetchPost(user.id))
    .then((post) => post.title)
    .catch((err) => console.error(err));
}

// async/await
async function getPostTitle() {
  try {
    const user = await fetchUser("짱구");
    const post = await fetchPost(user.id);
    return post.title;
  } catch (err) {
    console.error(err);
  }
}
```

로직이 복잡해질수록 `async/await`가 훨씬 읽기 편하다.
`.then()` 체인은 콜백이 중첩되다 보면 흐름 파악이 힘들어진다.

## 헷갈렸던 포인트

- **`await` 없이 `async` 함수 호출하면 Promise가 그냥 반환**된다. 결과를 쓰려면 `await` 하거나 `.then()` 해야 한다.
- **`async` 함수 안에서 `throw`하면 반환 Promise가 rejected**된다. 별도 `reject()` 호출 필요 없다.
- **`forEach`에 `async` 콜백을 넘기면 `await`가 제대로 동작하지 않는다.** `forEach`는 반환된 Promise를 기다리지 않는다. `for...of` 또는 `Promise.all + map` 쓸 것.
- **`await`는 Promise가 아닌 값도 받는다.** `await 42`는 그냥 `42`를 반환한다 — 무의미하지만 에러는 아니다.

## 요약

| 항목 | 설명 |
|------|------|
| `async` 함수 | 항상 Promise 반환, 내부에서 `await` 사용 가능 |
| `await` | Promise가 settled 될 때까지 함수 내부 일시 정지 |
| 에러 처리 | `try / catch / finally` |
| 병렬 처리 | `Promise.all([...])` 과 함께 사용 |
| `forEach` 주의 | `async` 콜백 + `forEach`는 `await`가 무시됨 |

- `async/await`는 Promise를 대체하는 게 아니라 더 편하게 쓰는 방법
- 독립적인 비동기 작업은 `Promise.all`로 묶어 병렬 처리
- 에러는 `try/catch`로, 공통 처리는 `.catch()` 체인도 여전히 유효

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function
