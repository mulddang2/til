# CORS (Cross-Origin Resource Sharing)

브라우저는 기본적으로 **다른 출처(origin)로의 요청을 차단**한다.
CORS는 서버가 "이 출처는 허용한다"고 명시적으로 알려줘서 이 제한을 풀어주는 메커니즘이다.

## 출처(Origin)란

출처는 **프로토콜 + 호스트 + 포트** 세 가지가 모두 같아야 같은 출처다.

| URL | 비교 기준 | 결과 |
|-----|---------|------|
| `https://짱구.com/a` → `https://짱구.com/b` | 경로만 다름 | ✅ 같은 출처 |
| `https://짱구.com` → `http://짱구.com` | 프로토콜 다름 | ❌ 다른 출처 |
| `https://짱구.com` → `https://둘리.com` | 호스트 다름 | ❌ 다른 출처 |
| `https://짱구.com` → `https://짱구.com:8080` | 포트 다름 | ❌ 다른 출처 |

## 동일 출처 정책 (Same-Origin Policy)

브라우저가 내장한 보안 정책으로, **다른 출처의 리소스를 읽는 것을 막는다**.
`<img>`, `<script src>`, `<link href>` 같은 임베딩은 허용하지만,
`fetch` / `XMLHttpRequest`로 읽어오는 응답은 차단한다.

```js
// 프론트: https://짱구.com 에서 실행 중
fetch("https://api.둘리.com/data")
  .then(res => res.json()); // ❌ CORS 에러 — 응답은 왔지만 브라우저가 차단
```

## 단순 요청 (Simple Request)

아래 조건을 **모두** 만족하면 preflight 없이 바로 요청한다.

- 메서드: `GET`, `HEAD`, `POST` 중 하나
- 헤더: 브라우저가 자동으로 붙이는 것 + `Content-Type` 정도
  - `Content-Type`이면 `text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded`만

서버가 응답에 `Access-Control-Allow-Origin`을 담아주면 브라우저가 응답을 허용한다.

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://짱구.com
```

## 프리플라이트 요청 (Preflight Request)

단순 요청 조건을 벗어나면 브라우저가 먼저 **OPTIONS 메서드로 사전 확인**을 보낸다.

```
OPTIONS /api/data HTTP/1.1
Origin: https://짱구.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization, Content-Type
```

서버가 허용 응답을 보내야 실제 요청이 간다.

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://짱구.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400
```

`Access-Control-Max-Age`로 preflight 결과를 캐시해두면 매 요청마다 OPTIONS를 보내지 않아도 된다.

## 주요 응답 헤더

| 헤더 | 역할 |
|------|------|
| `Access-Control-Allow-Origin` | 허용할 출처. `*`는 모두 허용 (credentials와 함께는 불가) |
| `Access-Control-Allow-Methods` | 허용할 HTTP 메서드 목록 |
| `Access-Control-Allow-Headers` | 허용할 요청 헤더 목록 |
| `Access-Control-Allow-Credentials` | 쿠키·인증 정보 포함 여부 |
| `Access-Control-Max-Age` | preflight 캐시 유지 시간 (초) |
| `Access-Control-Expose-Headers` | JS가 읽을 수 있는 응답 헤더 목록 |

## 인증 정보(Credentials) 포함 요청

쿠키나 `Authorization` 헤더를 같이 보내려면 **양쪽 모두 설정**해야 한다.

```js
// 클라이언트
fetch("https://api.둘리.com/me", {
  credentials: "include", // 쿠키 포함
});
```

```
# 서버 응답
Access-Control-Allow-Origin: https://짱구.com  ← * 사용 불가
Access-Control-Allow-Credentials: true
```

`credentials: "include"`인데 서버가 `Allow-Origin: *`를 보내면 브라우저가 차단한다.
반드시 특정 출처를 명시해야 한다.

## 개발 환경에서 우회하는 방법

프로덕션에서는 서버에서 해결해야 한다. 개발 중에는 프록시를 많이 쓴다.

```js
// vite.config.js
export default {
  server: {
    proxy: {
      "/api": {
        target: "https://api.둘리.com",
        changeOrigin: true,
      },
    },
  },
};
```

프록시를 쓰면 브라우저는 같은 출처(`localhost:5173`)로 요청하고,
개발 서버가 실제 API 서버로 중계해줘서 CORS를 우회할 수 있다.

## 헷갈렸던 포인트

- **CORS 에러는 서버가 응답을 안 보낸 게 아니다.** 응답은 왔지만 브라우저가 읽는 걸 막은 것. 서버 로그엔 요청이 찍힌다.
- **`Access-Control-Allow-Origin: *`와 `credentials: "include"`는 같이 쓸 수 없다.** 쿠키나 토큰을 담아 보낼 때는 정확한 출처를 지정해야 한다.
- **프리플라이트는 브라우저가 자동으로 보낸다.** 개발자가 직접 OPTIONS 요청을 만드는 게 아니다.
- **CORS는 브라우저의 제약이다.** `curl`이나 서버 간 통신에서는 아무 제약이 없다.
- **응답 헤더를 JS에서 읽으려면** `Access-Control-Expose-Headers`에 명시해야 한다. 기본 노출 헤더 외에는 숨겨진다.

## 요약

- 출처 = 프로토콜 + 호스트 + 포트, 셋 다 같아야 같은 출처
- 브라우저는 기본적으로 다른 출처 응답 차단 → 서버가 `Allow-Origin` 헤더로 풀어줌
- 단순 요청은 바로, 복잡한 요청은 OPTIONS preflight 먼저
- 쿠키·인증 포함이면 `Allow-Origin: *` 불가 → 출처 명시 + `Allow-Credentials: true`
- 개발 중 우회는 프록시 설정으로

참고: https://developer.mozilla.org/ko/docs/Web/HTTP/CORS
