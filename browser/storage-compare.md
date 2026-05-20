# localStorage / sessionStorage / cookie 비교

브라우저에서 데이터를 클라이언트 측에 저장하는 방법은 세 가지. 용도가 다르고 수명도 다르다.

## 한눈에 비교

| 항목 | localStorage | sessionStorage | cookie |
|------|-------------|----------------|--------|
| **수명** | 직접 삭제 전까지 유지 | 탭/창 닫으면 삭제 | `expires` / `max-age` 설정에 따라 |
| **용량** | ~5MB | ~5MB | ~4KB |
| **서버 전송** | X | X | 요청마다 자동 포함 |
| **접근 범위** | 같은 출처(origin) 전체 | 같은 탭 안에서만 | 도메인/경로 설정에 따라 |
| **JS 접근** | O | O | O (단, `HttpOnly`면 불가) |
| **API** | `Window.localStorage` | `Window.sessionStorage` | `document.cookie` |

## localStorage

영구 저장이 목적. 탭을 닫고 브라우저를 재시작해도 살아있다.

```js
// 저장
localStorage.setItem("user", "짱구");

// 읽기
const user = localStorage.getItem("user");
console.log(user); // "짱구"

// 삭제
localStorage.removeItem("user");

// 전체 삭제
localStorage.clear();
```

객체를 저장할 때는 직렬화가 필요하다.

```js
const profile = { name: "둘리", age: 5 };

localStorage.setItem("profile", JSON.stringify(profile));

const saved = JSON.parse(localStorage.getItem("profile"));
console.log(saved.name); // "둘리"
```

## sessionStorage

탭 단위 임시 저장. API는 localStorage와 완전히 동일하다.

```js
sessionStorage.setItem("token", "abc123");
sessionStorage.getItem("token"); // "abc123"
```

같은 URL이라도 **탭이 다르면 sessionStorage는 공유되지 않는다.**  
탭을 복제(Ctrl+D)하면 복제 시점의 데이터가 그대로 복사되지만, 이후에는 각자 독립적으로 관리된다.

## cookie

가장 오래된 저장소. 서버와 통신이 핵심 목적이다.

```js
// 기본 쓰기 (key=value 문자열)
document.cookie = "username=또치";

// 만료 시간 설정
document.cookie = "username=또치; max-age=3600"; // 1시간

// 읽기 — 모든 쿠키가 문자열 하나로 나온다
console.log(document.cookie); // "username=또치; theme=dark"

// 삭제 — max-age를 0이나 과거로 설정
document.cookie = "username=또치; max-age=0";
```

`document.cookie` API는 쓰기도 읽기도 불편하다. 실무에서는 [`js-cookie`](https://github.com/js-cookie/js-cookie) 같은 라이브러리를 쓰거나 유틸 함수를 직접 만든다.

### HttpOnly와 Secure

서버에서 응답 헤더로 설정하는 옵션들.

- `HttpOnly`: JS에서 접근 불가 → XSS로 탈취 못 하게 막음
- `Secure`: HTTPS 연결에서만 전송
- `SameSite`: CSRF 방어 (`Strict` / `Lax` / `None`)

```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax
```

인증 토큰처럼 민감한 정보는 `HttpOnly` 쿠키에 담는 게 안전하다.

## 용도 정리

| 상황 | 추천 저장소 |
|------|------------|
| 로그인 유지 (자동 로그인) | cookie (`HttpOnly`, `Secure`) |
| 접속 세션 중 임시 상태 (예: 장바구니 임시 저장) | sessionStorage |
| 테마, 언어 설정처럼 영구적인 UI 설정 | localStorage |
| 서버에 매 요청마다 보내야 하는 값 | cookie |

## 헷갈렸던 포인트

- **localStorage는 origin 단위**. `http://localhost:3000`과 `https://localhost:3000`은 다른 origin이라 데이터를 공유하지 않는다.
- **sessionStorage는 탭 단위**. 같은 브라우저 같은 사이트여도 탭이 다르면 별개. 새 창(`window.open`)도 별개 세션.
- **쿠키는 요청마다 자동 전송**. 크기가 크거나 불필요한 쿠키가 많으면 모든 요청에 실려가서 성능에 영향을 준다.
- **localStorage / sessionStorage는 서버 사이드 렌더링(SSR) 환경에서는 없다**. Next.js App Router 같은 서버 컴포넌트에서는 `window`가 없으니 접근하면 에러. `typeof window !== 'undefined'` 가드가 필요하다.

```js
// SSR 환경 가드 예시
const getToken = () => {
  if (typeof window === "undefined") return null;
  return localStorage.getItem("token");
};
```

## 요약

- **cookie**: 서버 전송이 목적, 만료 설정 가능, 용량 작음, `HttpOnly`로 JS 차단 가능
- **localStorage**: 영구 저장, 같은 origin 공유, 서버 전송 없음
- **sessionStorage**: 탭 닫으면 사라짐, 탭 단위 격리

참고: https://developer.mozilla.org/ko/docs/Web/API/Web_Storage_API
