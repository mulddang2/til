# TanStack Query 기초 (feat. React Query)

서버 상태(server state)를 관리하는 라이브러리. 비동기 데이터를 가져오고, 캐싱하고, 동기화하는 작업을 직접 짜지 않아도 된다.

> React Query라는 이름으로 널리 알려졌지만, v4부터 TanStack Query로 이름이 바뀌었다. React 외에도 Vue, Solid 등 다양한 프레임워크를 지원한다.

## 왜 필요한가

`useState` + `useEffect`로 데이터를 직접 패칭하면 이런 코드가 반복된다.

```js
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setLoading(true);
  fetch("/api/user/1")
    .then((res) => res.json())
    .then(setData)
    .catch(setError)
    .finally(() => setLoading(false));
}, []);
```

이걸 모든 데이터 요청마다 반복하고, 거기에 캐싱·리패칭·중복 요청 방지까지 직접 구현하면 복잡도가 폭발한다. TanStack Query가 이 반복을 흡수해준다.

## 기본 셋업

```bash
npm install @tanstack/react-query
```

앱 최상단에 `QueryClientProvider`로 감싼다.

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <짱구네집 />
    </QueryClientProvider>
  );
}
```

## useQuery — 데이터 읽기

```jsx
import { useQuery } from "@tanstack/react-query";

function 짱구프로필() {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ["user", 1],
    queryFn: () => fetch("/api/user/1").then((res) => res.json()),
  });

  if (isPending) return <p>불러오는 중...</p>;
  if (isError) return <p>에러: {error.message}</p>;

  return <h1>{data.name}</h1>;
}
```

| 반환값 | 의미 |
| --- | --- |
| `data` | 성공 시 서버 응답 데이터 |
| `isPending` | 아직 데이터 없음 (처음 로딩 중) |
| `isFetching` | 백그라운드 리패칭 포함해서 요청 중 |
| `isError` | 요청 실패 |
| `error` | 에러 객체 |

`isPending`과 `isFetching`은 다르다. 캐시된 데이터가 있으면서 리패칭 중일 때는 `isPending = false`, `isFetching = true`.

## queryKey — 캐시 키

`queryKey`는 배열 형태로 캐시를 식별한다. 같은 키면 같은 캐시를 본다.

```js
// 짱구(id=1)의 데이터와 둘리(id=2)의 데이터는 각각 별도 캐시
useQuery({ queryKey: ["user", 1], queryFn: ... });
useQuery({ queryKey: ["user", 2], queryFn: ... });

// 아이디가 바뀌면 자동으로 새 요청
useQuery({ queryKey: ["user", userId], queryFn: ... });
```

키가 바뀌면 `queryFn`을 다시 실행한다. 의존성 배열처럼 생각하면 편하다.

## useMutation — 데이터 변경

읽기가 아닌 쓰기(POST, PUT, DELETE) 에는 `useMutation`을 쓴다.

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function 이름바꾸기() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newName) =>
      fetch("/api/user/1", {
        method: "PUT",
        body: JSON.stringify({ name: newName }),
      }).then((res) => res.json()),
    onSuccess: () => {
      // 캐시 무효화 → useQuery가 자동으로 리패칭
      queryClient.invalidateQueries({ queryKey: ["user", 1] });
    },
  });

  return (
    <button onClick={() => mutation.mutate("둘리")}>
      이름 변경
    </button>
  );
}
```

`mutate(변수)`를 호출하면 `mutationFn`에 그 값이 전달된다.

## 캐시 옵션

```js
useQuery({
  queryKey: ["user", 1],
  queryFn: ...,
  staleTime: 1000 * 60 * 5,  // 5분 동안 fresh 상태 유지 (리패칭 안 함)
  gcTime: 1000 * 60 * 10,    // 10분 동안 캐시 보관 후 garbage collect
});
```

| 옵션 | 기본값 | 설명 |
| --- | --- | --- |
| `staleTime` | `0` | 이 시간 동안은 fresh → 리패칭 안 함 |
| `gcTime` | `5분` | 마운트 해제 후 캐시 유지 시간 |

기본 `staleTime = 0`이라 컴포넌트가 마운트될 때마다 백그라운드 리패칭이 일어난다. 자주 바뀌지 않는 데이터엔 `staleTime`을 늘려서 불필요한 요청을 줄이는 게 좋다.

## 자동 리패칭 트리거

기본적으로 다음 상황에 자동으로 리패칭한다.

- 컴포넌트가 다시 마운트될 때 (`refetchOnMount`)
- 브라우저 탭이 다시 포커스될 때 (`refetchOnWindowFocus`)
- 네트워크가 다시 연결될 때 (`refetchOnReconnect`)

테스트할 때 탭 이동했다고 갑자기 요청이 가는 게 의외로 당황스럽다. 이게 기본 동작이다.

## 헷갈렸던 포인트

- **`isPending` vs `isLoading`**: v5부터 `isLoading`이 `isPending && isFetching`으로 의미가 바뀌었다. 처음 로딩 상태는 `isPending`으로 체크하는 게 안전하다.
- **`queryKey`는 직렬화 가능해야 한다**: 함수나 클래스 인스턴스 같은 건 키에 넣으면 안 된다. 문자열, 숫자, 배열, 순수 객체로 구성해야 한다.
- **`invalidateQueries`는 refetch를 예약한다**: 호출 즉시 요청하는 게 아니라, 해당 쿼리가 마운트되어 있으면 바로, 아니면 다음 마운트 시 패칭한다.
- **서버 상태 ≠ 클라이언트 상태**: 모달 열림 여부 같은 UI 상태는 TanStack Query에 넣으면 안 된다. 그건 Zustand나 useState로 관리한다.

## 요약

- **`useQuery`**: 데이터 읽기 + 캐싱 + 자동 리패칭
- **`useMutation`**: 쓰기 작업 + 성공/실패 콜백
- **`queryKey`**: 캐시 식별자 — 키가 바뀌면 새 요청
- **`staleTime`**: 리패칭 빈도 조절 — 0이면 항상 stale 취급
- **`invalidateQueries`**: 특정 캐시 무효화 → 자동 리패칭 유도

참고: https://tanstack.com/query/latest/docs/framework/react/overview
