# 이벤트 루프 (Event Loop)

자바스크립트는 **싱글 스레드** 언어다. 한 번에 하나의 작업만 처리할 수 있다.  
그런데 어떻게 타이머, 네트워크 요청 같은 비동기 작업을 처리하는 걸까?  
그 답이 **이벤트 루프**다.

## 핵심 구성 요소

### 콜 스택 (Call Stack)

현재 실행 중인 함수가 쌓이는 곳. 함수를 호출하면 프레임이 쌓이고, 반환하면 빠진다.  
스택이 비어 있어야 이벤트 루프가 다음 작업을 가져올 수 있다.

### Web APIs (브라우저) / Node.js APIs

`setTimeout`, `fetch`, DOM 이벤트 핸들러 등 **비동기 작업을 브라우저(또는 Node.js)가 대신 처리**한다.  
작업이 완료되면 콜백을 큐에 넣는다.

### 태스크 큐 (Task Queue / Macro-task Queue)

`setTimeout`, `setInterval`, I/O 이벤트 콜백이 줄 서는 곳.  
콜 스택이 비면 이벤트 루프가 여기서 하나씩 꺼내 실행한다.

### 마이크로태스크 큐 (Microtask Queue)

`Promise.then`, `queueMicrotask`, `MutationObserver` 콜백이 줄 서는 곳.  
**태스크 큐보다 우선순위가 높다.** 콜 스택이 비면 마이크로태스크 큐를 먼저 전부 비운 뒤에야 태스크 큐로 넘어간다.

## 실행 순서

```
콜 스택 실행
  → 콜 스택이 비면
    → 마이크로태스크 큐 전부 처리
      → 태스크 큐에서 하나 꺼내 실행
        → 반복
```

## 코드 예시

```js
console.log("1");

setTimeout(() => {
  console.log("2 — setTimeout");
}, 0);

Promise.resolve().then(() => {
  console.log("3 — Promise.then");
});

console.log("4");
```

출력 순서:

```
1
4
3 — Promise.then
2 — setTimeout
```

순서를 따라가 보면:
1. `console.log("1")` → 콜 스택 실행
2. `setTimeout(...)` → Web API에게 넘기고 바로 반환. 타이머 완료 후 콜백이 **태스크 큐**에 들어감
3. `Promise.resolve().then(...)` → then 콜백이 **마이크로태스크 큐**에 들어감
4. `console.log("4")` → 콜 스택 실행
5. 콜 스택이 빔 → **마이크로태스크 큐** 먼저 처리 → `"3 — Promise.then"` 출력
6. 마이크로태스크 큐가 빔 → **태스크 큐**에서 setTimeout 콜백 꺼냄 → `"2 — setTimeout"` 출력

## 마이크로태스크 vs 태스크 비교

| 구분           | 종류                                      | 처리 시점                       |
| -------------- | ----------------------------------------- | ------------------------------- |
| 마이크로태스크 | `Promise.then`, `queueMicrotask`, `await` | 콜 스택이 빌 때마다 **전부** 처리 |
| 태스크         | `setTimeout`, `setInterval`, I/O          | 마이크로태스크 큐가 빈 뒤 **하나씩** 처리 |

## async/await도 마이크로태스크

```js
async function run() {
  console.log("A");
  await Promise.resolve();
  console.log("B");  // 마이크로태스크
}

run();
console.log("C");
```

출력:

```
A
C
B
```

`await` 뒤의 코드는 `Promise.then` 콜백으로 마이크로태스크 큐에 들어간다.  
그래서 `C`가 먼저 출력되고 나서 `B`가 나온다.

## 헷갈렸던 포인트

- **`setTimeout(fn, 0)`은 "즉시"가 아니다.** 0ms라도 태스크 큐로 가기 때문에, 마이크로태스크(Promise) 뒤에 실행된다.
- **마이크로태스크가 끝없이 쌓이면 태스크 큐가 영원히 실행 안 된다.** `Promise.then` 안에서 계속 새 Promise를 만들면 블로킹처럼 된다.
- **이벤트 루프는 브라우저/런타임이 제공한다.** 자바스크립트 엔진(V8) 자체에 포함된 게 아니라 실행 환경이 붙여주는 것.
- **렌더링도 태스크 사이에 끼어든다.** 브라우저는 보통 마이크로태스크 처리 → 렌더링 → 다음 태스크 순으로 동작한다. 마이크로태스크가 오래 걸리면 화면이 굳는 것처럼 보인다.

## 요약

- 자바스크립트는 싱글 스레드 → **이벤트 루프**가 비동기 콜백 실행을 관리
- 마이크로태스크 큐(`Promise.then`, `await`)는 태스크 큐(`setTimeout`) **보다 먼저** 실행
- 콜 스택이 비어야 큐의 작업을 가져올 수 있다
- `setTimeout(fn, 0)`이 Promise.then보다 늦게 실행되는 이유가 이 구조 때문

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Event_loop
