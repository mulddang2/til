# React Server Components (RSC)

React Server Components(RSC)는 **서버에서만 실행되고, 클라이언트 번들에 포함되지 않는** 컴포넌트다.
Next.js 13+ App Router 기준으로 `app/` 디렉토리 안에 있는 컴포넌트는 기본적으로 모두 서버 컴포넌트다.

## 서버 컴포넌트란

기존 React 컴포넌트는 서버에서 HTML을 만들어 보내더라도 결국 클라이언트에 JS로 다시 다운로드되고 **하이드레이션(hydration)**된다.
RSC는 다르다. 서버에서 렌더링 후 결과(직렬화된 트리)만 클라이언트에 전달하고, **해당 컴포넌트의 JS는 브라우저로 내려가지 않는다.**

### 핵심 차이

| 구분 | 서버 컴포넌트 | 클라이언트 컴포넌트 |
| --- | --- | --- |
| 실행 위치 | 서버 | 브라우저 (+ SSR 시 서버도) |
| 클라이언트 JS 번들 | 포함 안 됨 | 포함됨 |
| useState / useEffect | 사용 불가 | 사용 가능 |
| 이벤트 핸들러 | 사용 불가 | 사용 가능 |
| DB / 파일시스템 접근 | 직접 가능 | 불가 |
| `"use client"` 선언 | 없음 | 파일 상단에 필요 |

## 기본 사용 예시

```tsx
// app/users/page.tsx — 서버 컴포넌트 (기본값)
// "use client" 없으면 서버 컴포넌트

async function getUserList() {
  // DB나 fetch를 서버에서 직접 호출
  const res = await fetch("https://api.example.com/users");
  return res.json();
}

export default async function UsersPage() {
  const users = await getUserList(); // async/await 바로 가능

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

서버 컴포넌트는 `async` 함수로 만들 수 있어서 `useEffect` 없이 데이터를 바로 가져올 수 있다.

## 클라이언트 컴포넌트 분리

상호작용이 필요한 부분만 클라이언트 컴포넌트로 만든다.

```tsx
// components/LikeButton.tsx — 클라이언트 컴포넌트
"use client";

import { useState } from "react";

export default function LikeButton({ initialCount }: { initialCount: number }) {
  const [count, setCount] = useState(initialCount);

  return (
    <button onClick={() => setCount((c) => c + 1)}>
      ❤️ {count}
    </button>
  );
}
```

```tsx
// app/posts/[id]/page.tsx — 서버 컴포넌트가 클라이언트 컴포넌트를 포함
import LikeButton from "@/components/LikeButton";

export default async function PostPage({ params }: { params: { id: string } }) {
  const post = await getPost(params.id); // 서버에서 DB 조회

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <LikeButton initialCount={post.likeCount} />
    </article>
  );
}
```

서버 컴포넌트가 클라이언트 컴포넌트를 **children으로 사용하는 건 가능**하다.
반대로 클라이언트 컴포넌트가 서버 컴포넌트를 import해서 직접 렌더링하는 건 안 된다.

## 서버 컴포넌트가 클라이언트 컴포넌트의 children이 되는 패턴

```tsx
// app/layout.tsx — 서버 컴포넌트
import Sidebar from "@/components/Sidebar"; // 클라이언트 컴포넌트

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      {/* Sidebar는 클라이언트 컴포넌트지만 children으로 서버 컴포넌트를 받을 수 있음 */}
      <Sidebar>
        <UserInfo /> {/* 서버 컴포넌트 */}
      </Sidebar>
      <main>{children}</main>
    </div>
  );
}
```

`UserInfo`가 서버에서 렌더링된 결과를 `Sidebar`의 `children`으로 넘기는 방식이라 가능하다.

## 헷갈렸던 포인트

**1. "use client"는 파일 단위**

`"use client"`를 붙이면 해당 파일과 그 파일이 import하는 모든 모듈이 클라이언트 번들에 들어간다.
클라이언트 컴포넌트 안에서 import한 건 전부 클라이언트 사이드가 된다는 뜻이다.

**2. 서버 컴포넌트는 props로 직렬화 가능한 값만 받을 수 있다**

클라이언트 컴포넌트에서 서버 컴포넌트로 함수를 props로 넘기려고 하면 오류가 난다.
함수는 직렬화가 안 되기 때문이다.

```tsx
// ❌ 클라이언트 컴포넌트에서 서버 컴포넌트로 함수 넘기기
<ServerComp onClick={() => console.log("짱구")} />
```

**3. 서버 컴포넌트는 SSR과 다르다**

SSR은 모든 컴포넌트를 서버에서 HTML로 렌더링하지만, 클라이언트에 JS가 다 내려간다.
RSC는 서버 컴포넌트의 JS 자체를 클라이언트로 보내지 않아서 번들 크기가 줄어든다.

**4. Context는 클라이언트 컴포넌트에서만**

`createContext` / `useContext`는 클라이언트 컴포넌트에서만 쓴다. 서버 컴포넌트에는 Context가 없다.

## 요약

- 서버 컴포넌트는 서버에서만 실행, **클라이언트 번들에 JS 없음**
- `async/await`로 데이터 패칭 가능, DB·파일시스템 직접 접근 가능
- `useState`, `useEffect`, 이벤트 핸들러 사용 불가
- 상호작용이 필요한 부분만 `"use client"` 붙여 클라이언트 컴포넌트로 분리
- Next.js App Router에서 `app/` 안은 기본값이 서버 컴포넌트

참고: https://react.dev/reference/rsc/server-components
