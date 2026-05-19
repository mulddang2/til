# fetch와 axios

브라우저에서 HTTP 요청을 보내는 두 가지 대표 수단이다.  
`fetch`는 브라우저 내장, `axios`는 외부 라이브러리 — 이 차이에서 나머지 특징이 갈린다.

## fetch — 브라우저 내장 API

```js
const res = await fetch("https://api.example.com/users");
const data = await res.json();
console.log(data);
```

두 가지를 기억해야 한다.

1. **응답을 두 번 처리**해야 한다. `fetch`는 네트워크 응답 자체(`Response`)를 반환하고, 실제 본문은 `.json()`, `.text()` 같은 메서드로 한 번 더 꺼내야 한다.
2. **4xx / 5xx 에러를 throw하지 않는다**. `ok` 프로퍼티(`res.ok`)가 `false`여도 에러 없이 resolve된다.

```js
const res = await fetch("https://api.example.com/users/999");

if (!res.ok) {
  throw new Error(`HTTP ${res.status}`);
}

const data = await res.json();
```

`res.ok`를 직접 확인하지 않으면 404가 와도 조용히 넘어간다.

## POST 요청

```js
await fetch("https://api.example.com/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "짱구", age: 5 }),
});
```

`body`에는 반드시 **문자열**로 직렬화해서 넣어야 한다. 객체를 그냥 넣으면 `"[object Object]"` 문자열이 전송된다.

## axios — 외부 라이브러리

```js
import axios from "axios";

const { data } = await axios.get("https://api.example.com/users");
console.log(data);
```

`fetch`와 달라지는 점이 몇 가지 있다.

- **응답이 한 번에 파싱**된다. `res.json()` 없이 `.data`로 바로 접근.
- **4xx / 5xx에서 자동으로 throw**한다. 별도 `ok` 체크 불필요.
- `Content-Type: application/json` 헤더를 자동 설정한다.
- 객체를 넘기면 직렬화도 알아서 해준다.

```js
await axios.post("https://api.example.com/users", {
  name: "둘리",
  age: 5,
});
```

`fetch`와 비교하면 코드가 훨씬 짧아진다.

## POST 에러 처리 비교

```js
// fetch
try {
  const res = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: "또치" }),
  });

  if (!res.ok) throw new Error(`${res.status}`);
  const data = await res.json();
} catch (e) {
  console.error(e);
}

// axios
try {
  const { data } = await axios.post("/api/users", { name: "또치" });
} catch (e) {
  console.error(e.response?.status); // 에러 응답 정보 포함
}
```

axios의 에러 객체는 `e.response`, `e.request`, `e.message` 를 담고 있어서 디버깅이 편하다.

## fetch vs axios 비교

| 항목 | fetch | axios |
|---|---|---|
| 설치 | 불필요 (내장) | `npm i axios` |
| 응답 파싱 | `.json()` 별도 호출 | 자동 |
| 4xx / 5xx 에러 | throw 안 함 (직접 확인) | 자동 throw |
| 요청 body | `JSON.stringify` 직접 | 자동 직렬화 |
| 인터셉터 | 없음 | 있음 |
| 요청 취소 | `AbortController` | `CancelToken` / `AbortController` |
| Node.js (18 이전) | 미지원 | 지원 |

## 인터셉터 — axios의 핵심 장점

모든 요청/응답에 공통 처리를 끼워넣을 수 있다.

```js
// 모든 요청에 Authorization 헤더 자동 삽입
axios.interceptors.request.use((config) => {
  config.headers.Authorization = `Bearer ${getToken()}`;
  return config;
});

// 401이면 로그아웃 처리
axios.interceptors.response.use(
  (res) => res,
  (err) => {
    if (err.response?.status === 401) logout();
    return Promise.reject(err);
  }
);
```

`fetch`로 같은 걸 구현하려면 래퍼 함수를 직접 만들어야 한다.

## 헷갈렸던 포인트

- **fetch는 네트워크 에러만 reject**한다. 서버가 400, 500을 돌려줘도 Promise는 resolve되기 때문에 `res.ok` 체크를 빠뜨리면 조용히 잘못된 데이터를 쓰게 된다.
- **axios는 `data`를 한 번 감싼 구조**다. 응답 전체는 `res`이고, 서버 응답 본문은 `res.data`. 구조분해로 바로 `const { data } = await axios.get(...)` 쓰는 게 편하다.
- 번들 크기가 신경 쓰이는 환경에서는 `fetch`가 더 적합하다. axios는 약 15KB(gzip) 추가된다.

## 요약

- `fetch`: 내장이라 추가 설치 없음 — 단, 에러 처리와 파싱을 직접 해야 한다
- `axios`: 보일러플레이트 줄어들고 인터셉터로 공통 처리 가능 — 의존성 추가됨
- 작은 프로젝트는 `fetch`, 토큰 처리·공통 에러 핸들링이 필요한 프로젝트는 `axios`

참고:
- https://developer.mozilla.org/ko/docs/Web/API/Fetch_API/Using_Fetch
- https://axios-http.com/kr/docs/intro
