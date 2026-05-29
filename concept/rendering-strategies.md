# 렌더링 전략 (CSR / SSR / SSG / ISR)

Next.js나 React 프레임워크를 쓰면 반드시 마주치는 네 가지 렌더링 전략. 무엇을 언제 고를지 헷갈리기 쉽다.

## 렌더링이란?

HTML을 **어디서, 언제 만드는가**의 문제다.

- 브라우저(클라이언트)에서 만들면 → CSR
- 서버에서 만들면 → SSR / SSG / ISR

---

## CSR (Client-Side Rendering)

브라우저가 JS를 받아서 직접 HTML을 그린다.

```
브라우저 요청 → 서버: 빈 HTML + JS 번들 반환
→ 브라우저: JS 실행 → API 호출 → DOM 그리기
```

```jsx
// React 단독 앱 (Vite, CRA)
function App() {
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    fetch("/api/todos").then(r => r.json()).then(setTodos);
  }, []);

  return <ul>{todos.map(t => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

**장점**
- 초기 로드 후 페이지 이동이 빠름 (SPA처럼 동작)
- 서버 부하가 적음
- 개발·배포 간단

**단점**
- 첫 화면(FCP)이 느림 — JS 다운로드 + 실행 + API 요청까지 기다려야 함
- SEO 불리 — 크롤러가 JS 실행 전 빈 HTML을 보는 경우 있음

---

## SSR (Server-Side Rendering)

**요청마다** 서버가 HTML을 만들어서 보낸다.

```
브라우저 요청 → 서버: 데이터 조회 → HTML 생성 → 완성된 HTML 반환
```

```jsx
// Next.js App Router (서버 컴포넌트)
// app/todos/page.tsx
async function TodosPage() {
  const todos = await fetch("https://api.example.com/todos").then(r => r.json());

  return <ul>{todos.map(t => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

**장점**
- 첫 화면이 빠름 — 완성된 HTML이 바로 옴
- SEO에 유리 — 크롤러가 완성된 HTML을 읽음
- 항상 최신 데이터

**단점**
- 요청마다 서버 연산 필요 → 서버 부하
- TTFB(Time To First Byte)가 길어질 수 있음
- 서버 인프라 필요

---

## SSG (Static Site Generation)

**빌드 타임**에 HTML을 미리 만들어 둔다. 요청 때는 만들어진 파일을 그냥 준다.

```
빌드 시: 데이터 조회 → HTML 생성 → 파일로 저장
브라우저 요청 → CDN/서버: 저장된 HTML 바로 반환
```

```jsx
// Next.js App Router — fetch 캐시를 force-cache로 (기본값)
async function BlogPost({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.id}`, {
    cache: "force-cache", // 빌드 시 캐시
  }).then(r => r.json());

  return <article>{post.content}</article>;
}
```

**장점**
- 응답 속도 최고 — CDN에서 파일 그냥 줌
- 서버 부하 거의 없음
- SEO 최적

**단점**
- 데이터가 바뀌어도 **재빌드 전까지 반영 안 됨**
- 빌드 시간이 길어질 수 있음 (페이지 수가 많을수록)
- 사용자별 맞춤 데이터 불가

---

## ISR (Incremental Static Regeneration)

SSG의 업그레이드. **일정 시간마다** 백그라운드에서 페이지를 재생성한다.

```
첫 요청 → 빌드된 HTML 반환 (stale이어도 일단 줌)
n초 경과 후 → 백그라운드에서 페이지 재생성 → 다음 요청부터 새 HTML
```

```jsx
// Next.js App Router — revalidate 설정
async function ProductPage({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { revalidate: 60 }, // 60초마다 재검증
  }).then(r => r.json());

  return <div>{product.name}: {product.price}원</div>;
}
```

**장점**
- SSG 속도 + 주기적 데이터 갱신
- 재빌드 없이도 새 데이터 반영 가능

**단점**
- n초 동안은 오래된 데이터가 나올 수 있음 (stale-while-revalidate)
- 실시간 정확도가 필요한 경우 부적합

---

## 전략 비교

| 전략 | HTML 생성 시점 | 데이터 최신성 | 속도 | SEO | 서버 필요 |
| ---- | -------------- | ------------- | ---- | --- | --------- |
| CSR  | 브라우저에서   | 실시간        | 첫 로드 느림 | 불리 | 불필요 |
| SSR  | 요청마다       | 실시간        | 보통  | 유리 | 필요 |
| SSG  | 빌드 시        | 빌드 시점     | 최고  | 최적 | 불필요(CDN) |
| ISR  | 빌드 + 주기적  | n초 이내      | 빠름  | 최적 | 필요 |

---

## 언제 뭘 쓰나

```
로그인 후 개인화 데이터 (대시보드, 마이페이지)
→ CSR (또는 SSR + 클라이언트 hydration)

실시간 데이터 (주식, 채팅, 재고)
→ SSR (또는 CSR + WebSocket)

자주 안 바뀌는 공개 페이지 (블로그, 문서, 마케팅)
→ SSG

상품 목록처럼 가끔 바뀌는 공개 페이지
→ ISR

SEO가 중요한데 데이터도 자주 바뀜
→ SSR
```

---

## 헷갈렸던 포인트

**1. Next.js App Router에서 기본값은 SSG**

`fetch()`를 쓰면 기본적으로 `force-cache`(SSG처럼 빌드 시 캐시)다. 실시간으로 받으려면 `cache: "no-store"` 또는 `revalidate: 0`을 명시해야 한다.

**2. Hydration이란**

SSR/SSG로 받은 HTML에 React가 이벤트 핸들러 등을 붙이는 과정. 화면은 빨리 보이지만 JS가 로드되기 전까진 클릭 등이 안 먹힐 수 있다(TTI, Time To Interactive).

**3. ISR은 stale-while-revalidate 전략**

요청이 들어오면 일단 오래된 캐시를 주고, 백그라운드에서 갱신한다. 짱구가 60초 타이머를 리셋하는 게 아니라, 요청이 들어올 때 타이머를 체크한다고 보면 됨.

**4. CSR도 SSR 안에서 같이 쓸 수 있다**

Next.js에서는 서버 컴포넌트(SSR/SSG)로 뼈대를 그리고, `"use client"` 클라이언트 컴포넌트로 인터랙티브한 부분만 CSR로 처리할 수 있다.

---

## 요약

- **CSR**: JS로 브라우저에서 직접 그림. 개인화 UI, SEO 불필요한 곳.
- **SSR**: 요청마다 서버에서 HTML 생성. 실시간 + SEO 둘 다 필요한 곳.
- **SSG**: 빌드 때 미리 생성. 안 바뀌는 공개 페이지, 속도 최우선.
- **ISR**: SSG + 주기적 갱신. 가끔 바뀌는 공개 페이지.

참고:
- https://nextjs.org/docs/app/building-your-application/rendering
- https://developer.mozilla.org/ko/docs/Web/Performance/Critical_rendering_path
