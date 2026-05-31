# Next.js App Router

Next.js 13부터 도입된 **App Router**는 `app/` 디렉토리를 기반으로 라우팅을 정의한다.  
기존 `pages/` 방식(Pages Router)을 완전히 대체하며, React Server Components(RSC)가 기본이다.

## Pages Router vs App Router 한눈 비교

| 항목 | Pages Router (`pages/`) | App Router (`app/`) |
|---|---|---|
| 기본 컴포넌트 | 클라이언트 컴포넌트 | **서버 컴포넌트** |
| 데이터 페칭 | `getServerSideProps` / `getStaticProps` | `fetch()` + `async` 컴포넌트 |
| 레이아웃 | `_app.tsx` 전역만 | `layout.tsx` 중첩 가능 |
| 로딩 UI | 직접 구현 | `loading.tsx` 자동 Suspense |
| 에러 처리 | `_error.tsx` | `error.tsx` 자동 Error Boundary |

## 파일 기반 라우팅

`app/` 아래 폴더 구조가 곧 URL이다.

```
app/
├── layout.tsx          → /  (전체 공통 레이아웃)
├── page.tsx            → /
├── about/
│   └── page.tsx        → /about
└── blog/
    ├── page.tsx        → /blog
    └── [slug]/
        └── page.tsx    → /blog/:slug
```

폴더 이름에 `[param]`을 쓰면 동적 라우트가 된다.

```tsx
// app/blog/[slug]/page.tsx
export default function BlogPost({ params }: { params: { slug: string } }) {
  return <h1>글: {params.slug}</h1>;
}
```

## 특수 파일들

| 파일명 | 역할 |
|---|---|
| `page.tsx` | 라우트 UI. 없으면 해당 경로에 접근 불가 |
| `layout.tsx` | 자식 라우트에 공통으로 감싸지는 레이아웃 |
| `loading.tsx` | Suspense 경계. 페이지 로딩 중 보여줄 UI |
| `error.tsx` | Error Boundary. 렌더링 중 에러 발생 시 UI |
| `not-found.tsx` | 404 UI |

## layout.tsx — 중첩 레이아웃

```tsx
// app/layout.tsx (루트 레이아웃 — 필수)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  );
}
```

```tsx
// app/dashboard/layout.tsx (대시보드 전용 레이아웃)
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <nav>사이드바</nav>
      <main>{children}</main>
    </div>
  );
}
```

`/dashboard/settings`에 접근하면 RootLayout → DashboardLayout → page 순으로 중첩된다.

## 서버 컴포넌트에서 데이터 페칭

App Router에서는 컴포넌트 자체를 `async`로 만들어 데이터를 바로 받아올 수 있다.

```tsx
// app/todos/page.tsx — 서버에서만 실행, 브라우저에 JS 미포함
async function getTodos() {
  const res = await fetch("https://jsonplaceholder.typicode.com/todos?_limit=5", {
    cache: "no-store", // SSR: 매 요청마다 새로 가져오기
  });
  return res.json();
}

export default async function TodosPage() {
  const todos = await getTodos();
  return (
    <ul>
      {todos.map((todo: { id: number; title: string }) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

`cache` 옵션으로 캐싱 전략을 제어한다:

| `cache` 값 | 동작 | 기존 방식 |
|---|---|---|
| `force-cache` (기본) | 빌드 시 캐시 | `getStaticProps` |
| `no-store` | 매 요청마다 새로 fetch | `getServerSideProps` |
| `{ next: { revalidate: 60 } }` | 60초마다 재검증 | ISR |

## 클라이언트 컴포넌트

상태(`useState`), 이벤트 핸들러, 브라우저 API가 필요한 컴포넌트는 상단에 `'use client'`를 선언한다.

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>짱구 클릭: {count}</button>;
}
```

`'use client'`는 해당 파일부터 시작하는 **클라이언트 경계**를 표시한다.  
그 아래에 임포트된 컴포넌트들도 모두 클라이언트 번들에 포함된다.

## 서버 컴포넌트 안에 클라이언트 컴포넌트 넣기

```tsx
// app/page.tsx (서버)
import Counter from "@/components/Counter"; // 클라이언트 컴포넌트

export default async function Home() {
  const data = await fetchSomeData(); // 서버에서 실행
  return (
    <div>
      <h1>{data.title}</h1>
      <Counter /> {/* 클라이언트 컴포넌트 */}
    </div>
  );
}
```

서버 컴포넌트가 클라이언트 컴포넌트를 렌더링하는 건 OK.  
**반대로 클라이언트 컴포넌트가 서버 컴포넌트를 직접 임포트하는 건 안 된다.**  
`children`으로 props에서 받아 넘기는 패턴으로 우회한다.

## 헷갈렸던 포인트

- **`app/` 내 모든 컴포넌트는 기본 서버 컴포넌트**다. `useState` 쓰다 "Server Components cannot use hooks" 에러 보면 `'use client'` 빠진 것.
- **`layout.tsx`는 리렌더링 안 된다**. 자식 라우트 이동 시 layout은 유지되고 page만 바뀐다. 덕분에 공통 UI 깜빡임이 없다.
- **`loading.tsx`는 자동 Suspense**다. 따로 `<Suspense>` 감싸지 않아도 `async` page가 데이터 받는 동안 loading UI가 보인다.
- Pages Router에서 쓰던 `getServerSideProps`, `getStaticProps`는 App Router에서 쓸 수 없다.

## 요약

- App Router는 `app/` 폴더 구조 = URL, 서버 컴포넌트가 기본
- `layout.tsx` 중첩으로 레이아웃을 계층적으로 관리
- 데이터 페칭은 `async` 컴포넌트 + `fetch()` + `cache` 옵션으로 SSR/SSG/ISR 구분
- 브라우저 기능 필요 시 파일 상단에 `'use client'` 선언

참고: https://nextjs.org/docs/app/building-your-application/routing
